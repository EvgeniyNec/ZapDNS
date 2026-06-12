# Архитектура ZapDNS

## Общая концепция

ZapDNS — это **гибридное** приложение, которое не пытается засунуть весь трафик в VPN-туннель, а применяет DPI-bypass **только там, где нужно**, и использует **своим DNS** для максимальной скорости и приватности.

### Два режима работы (MVP + будущие)

1. **Лёгкий режим (по умолчанию)** — рекомендуется
   - VpnService захватывает трафик
   - TCP/UDP пакеты обрабатываются локальным **byedpi** (DPI bypass)
   - DNS-запросы идут на **свой DoH-сервер** на VPS
   - Большая часть трафика идёт напрямую (без изменения IP)

2. **Полный туннель (опционально)**
   - Переключатель «Включить Xray»
   - Весь трафик (или выбранные приложения) идёт через Xray/VLESS+Reality или Hysteria2 на VPS
   - Используется когда лёгкого bypass недостаточно

## Android Архитектура (высокий уровень)

```
ZapDNS Android App
├── UI Layer (Jetpack Compose)
│   ├── MainActivity + Navigation
│   ├── Screens: Home, Profiles, Settings, Stats, Logs
│   └── Material You + Dynamic Color
│
├── Domain Layer (Use Cases + Models)
│   ├── BypassManager
│   ├── DnsManager (DoH)
│   ├── ProfileManager
│   ├── XrayManager (опционально)
│   └── StatsRepository
│
├── Data Layer
│   ├── Local: DataStore (настройки) + Room (профили, история)
│   ├── Native: byedpi (JNI) + hev-socks5-tunnel
│   └── Remote: DoH клиент → VPS
│
├── VPN Core (VpnService)
│   ├── ZapDNSVpnService (extends VpnService)
│   ├── Tun2Socks / Packet redirection
│   ├── Integration with byedpi SOCKS5 proxy
│   └── DNS handling (DoH resolver)
│
├── Native Layer (C / C++)
│   ├── libbyedpi.so (собранный hufrea/byedpi)
│   ├── JNI bindings (Kotlin ↔ C)
│   └── (опционально) libhev-socks5-tunnel.so
│
└── Background
    ├── BootReceiver (автозапуск)
    ├── TileService (Quick Settings Tile)
    └── Foreground Service (persistent notification)
```

## Ключевые компоненты

### 1. VpnService (основа всего)

```kotlin
class ZapDNSVpnService : VpnService() {
    private lateinit var byedpi: ByeDPI
    private lateinit var dnsResolver: DoHResolver

    override fun onCreate() {
        // 1. Создаём TUN интерфейс
        val builder = Builder()
            .setSession("ZapDNS")
            .addAddress("10.0.0.1", 24)
            .addRoute("0.0.0.0", 0)           // весь IPv4
            .addDnsServer("...")              // или перехватываем DNS
            // ...

        vpnInterface = builder.establish()

        // 2. Запускаем локальный byedpi SOCKS5
        byedpi = ByeDPI().apply { start(config) }

        // 3. Настраиваем перенаправление трафика (tun2socks или свой)
        // 4. Настраиваем DoH resolver
    }

    override fun onDestroy() {
        byedpi.stop()
        super.onDestroy()
    }
}
```

**Важно**: В ByeDPIAndroid и ByeByeDPI уже реализован этот паттерн.  
Мы **не пишем с нуля** — форкаем/изучаем и расширяем.

### 2. DPI Bypass (byedpi)

- Библиотека: `hufrea/byedpi` (C)
- Запускается как **локальный SOCKS5 прокси** (обычно `127.0.0.1:1080`)
- Применяет стратегии:
  - `split` / `disorder` / `fake` / `oob` и т.д.
  - Можно задавать разные стратегии под разные домены (в будущем)
- В MVP: одна глобальная стратегия + пресеты в профилях

**Интеграция**:
- Собираем `byedpi` через NDK/CMake в `.so`
- Kotlin общается через JNI (или Process + local socket)

### 3. DNS Module (свой DoH)

**Проблема**: Android VpnService позволяет задать только IP DNS-серверов.  
Для **DoH** нужно приложение-уровень.

Решения (по приоритету):

1. **Лучшее для гибридного подхода**:
   - Перехватывать UDP 53 в TUN
   - Перенаправлять DNS-запросы на встроенный DoH-клиент
   - Использовать библиотеку `dns-over-https` или `OkHttp` + своя обёртка

2. **Простое (MVP)**:
   - Использовать системный Private DNS (если пользователь включит DoH на устройстве)
   - Но тогда сложно делать per-profile

3. **Продвинутое**:
   - Полностью свой DNS-resolver внутри VPN-сервиса (рекомендуется в долгосрочной перспективе)

**Рекомендация для MVP**: Начать с варианта 1 — перехват DNS + DoH к своему VPS.

### 4. Профили и Стратегии

```kotlin
data class Profile(
    val id: String,
    val name: String,
    val dpiStrategy: DpiStrategy,      // split, disorder, fake...
    val useCustomDns: Boolean,
    val customDnsUrl: String?,         // https://dns.tvoj-vps.ru/dns-query
    val enableXrayFallback: Boolean,
    val selectedApps: List<String>,    // package names (split-tunneling)
    val isDefault: Boolean
)
```

### 5. Xray Integration (опционально)

- Кнопка в UI: «Включить полный туннель»
- При включении:
  - Останавливается лёгкий byedpi режим
  - Запускается Xray / sing-box / v2rayNG-core в фоне
  - Перенастраивается VpnService на routing через локальный SOCKS/HTTP прокси Xray
- Можно использовать `hev-socks5-tunnel` или `libv2ray` / `Xray-core` Android библиотеки

## Диаграмма потока данных (упрощённо)

```
Приложение / Браузер
        ↓
   Android VpnService (TUN)
        ↓
   [Packet Capture / Modification]
        ↓
   ┌─────────────────────┐
   │  Если DNS-запрос    │ → DoH Resolver → VPS DoH (твой сервер)
   └─────────────────────┘
        ↓
   ┌─────────────────────┐
   │  TCP/UDP данные     │ → byedpi (DPI bypass стратегии)
   └─────────────────────┘
        ↓
   Реальный интернет (прямо или через Xray при необходимости)
```

## Безопасность и Приватность

- Весь код open source
- DNS только на своём VPS (никаких Google/Cloudflare DoH)
- Логи только локально (опционально отправлять на свой сервер)
- Нет обязательной регистрации / аккаунтов
- Возможность полностью отключить сбор статистики

## Производительность

- byedpi очень лёгкий (написан на C)
- VpnService на Android имеет overhead, но для DPI-bypass это стандарт
- Цель: < 5-10% overhead по сравнению с прямым подключением

## Следующие шаги (после MVP)

- Полноценный split-tunneling (per-app + per-domain)
- Статистика и логи в реальном времени
- Авто-определение заблокированных доменов
- Портирование на Windows (GUI + служба)
- Общий модуль стратегий DPI (Android + ПК)
- Поддержка IPv6
- WireGuard fallback (как дополнительный режим)

---

**ZapDNS** — не просто ещё один байпассер.  
Это **твой** инструмент с полным контролем над трафиком и DNS.
