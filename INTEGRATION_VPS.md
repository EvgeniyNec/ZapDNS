# Интеграция с VPS (DNS + Xray)

## Твоя текущая инфраструктура (идеально подходит)

У тебя уже есть:
- VPS на AEZA.net (Debian 12)
- HiddifyManager + Xray / sing-box (VLESS+Reality, Hysteria2, Shadowsocks)
- Собственный DNS (можно поднять DoH/DoT)

ZapDNS будет **идеально** с этим работать.

## 1. DNS на VPS (DoH / DoT) — приоритет №1

### Варианты подъёма DoH-сервера

**Рекомендуемый (простой и быстрый)**:

```bash
# На VPS (Debian/Ubuntu)
# Вариант A: AdGuard Home (самый простой UI)
docker run -d \
  --name adguardhome \
  -p 53:53/tcp -p 53:53/udp \
  -p 853:853/tcp \
  -p 443:443/tcp -p 443:443/udp \
  -v /opt/adguardhome/work:/opt/adguardhome/work \
  -v /opt/adguardhome/conf:/opt/adguardhome/conf \
  adguard/adguardhome
```

Затем включи **DoH** в настройках AdGuard Home:
- `https://dns.tvoj-domen.ru/dns-query`

**Вариант B: Caddy + CoreDNS** (лёгкий, без Docker)

**Вариант C: dns-over-https (от Google, но свой)** — https://github.com/m13253/dns-over-https

### Настройка в ZapDNS (MVP)

В настройках приложения:
```
DoH URL: https://dns.tvoj-vps.ru/dns-query
```

Приложение должно:
1. Перехватывать DNS-запросы из TUN
2. Пересылать их по DoH на твой сервер
3. Возвращать ответы в приложение

**Проверка**:
- https://dnsleaktest.com
- https://www.dnsleaktest.com/
- Должно показывать только твой VPS / твой ASN

## 2. Xray / sing-box как fallback

### Сценарий использования

- **Обычный день**: ZapDNS в лёгком режиме (byedpi + твой DoH)
- **Нужно максимум**: Нажал кнопку → переключился на полный Xray-туннель

### Как реализовать переключение

В `ZapDNSVpnService`:

```kotlin
when (mode) {
    Mode.LIGHT -> {
        // byedpi + DoH
        startByeDPI()
        configureDoH()
    }
    Mode.FULL_TUNNEL -> {
        // Останавливаем лёгкий режим
        stopByeDPI()
        // Запускаем Xray core
        startXray(configFromVPS)
        // Маршрутизируем весь трафик через локальный прокси Xray
    }
}
```

### Варианты запуска Xray на Android

1. **Лучший**: Использовать готовые библиотеки
   - `com.github.MatsuriDayo:neko` или `Xray-core` Android обёртки
   - `hev-socks5-tunnel` для простого SOCKS5 → TUN

2. **Простой**: Запускать Xray как отдельный процесс (`.so` + `exec`)

3. **Самый стабильный**: Использовать код из **v2rayNG** или **NekoBox** (открытый)

### Конфигурация Xray на VPS (твоя)

У тебя уже есть HiddifyManager — отлично.

В ZapDNS можно будет импортировать:
- VLESS + Reality конфиг
- Hysteria2
- Или sing-box JSON

## 3. Единая точка управления (будущее)

Идеально сделать небольшой **веб-интерфейс** или API на VPS, чтобы:
- ZapDNS запрашивал актуальные стратегии DPI
- Получал обновлённые Xray-конфиги
- Отправлял анонимную статистику (опционально)

Но это **не MVP**.

## 4. Безопасность интеграции

- DoH должен быть на **своём домене** с нормальным TLS (Let's Encrypt)
- Xray Reality — лучший выбор (не отличим от обычного HTTPS)
- В приложении — возможность **проверить fingerprint** Reality
- Никогда не хардкодить чувствительные данные в APK

## Рекомендуемая схема (2026)

```
Android (ZapDNS)
    ├── Лёгкий режим
    │   ├── byedpi (DPI bypass)
    │   └── DoH → AdGuard Home / CoreDNS на VPS
    │
    └── Полный режим
        └── Xray / sing-box → VPS (VLESS+Reality / Hysteria2)
```

## Что нужно сделать на VPS прямо сейчас (подготовка)

1. Поднять **AdGuard Home** или **CoreDNS + Caddy** с DoH
2. Настроить firewall (открыть 443/tcp для DoH)
3. Протестировать DoH:
   ```bash
   curl -v "https://dns.tvoj-domen.ru/dns-query?name=example.com&type=A"
   ```
4. Подготовить Xray/sing-box конфиг для импорта в приложение (позже)

---

**ZapDNS + твой VPS = максимально контролируемый и быстрый обход DPI.**
