# Android MVP — ZapDNS

## Что входит в первую версию (MVP)

### Обязательно (релиз 0.1)

- [x] Базовый VpnService с подключением/отключением
- [x] Интеграция **hufrea/byedpi** как нативной библиотеки (через NDK)
- [x] Запуск byedpi как локального SOCKS5
- [x] Перенаправление трафика через TUN → byedpi (на базе кода из ByeDPIAndroid / ByeByeDPI)
- [x] Простая настройка **своего DoH** (один URL в настройках)
- [x] 3–4 готовых профиля:
  - YouTube + Discord
  - Игры (минимальный overhead)
  - Максимальная скорость (минимальный bypass)
  - Максимальная приватность (DoH + агрессивный DPI)
- [x] Переключение профилей в UI
- [x] Автозапуск при загрузке устройства
- [x] Foreground Service + уведомление
- [x] Quick Settings Tile (плитка в шторке)
- [x] Material You дизайн (Jetpack Compose)

### Желательно в MVP (релиз 0.2)

- [ ] Переключатель «Включить полный Xray туннель»
- [ ] Базовая статистика (кол-во обработанных пакетов, топ доменов)
- [ ] Логи DPI-стратегий (какие домены были обработаны)
- [ ] Сохранение выбранного профиля между перезапусками
- [ ] Простой split-tunneling по приложениям (выбор 2–3 приложений)

### Не входит в MVP (позже)

- Полноценный per-app + per-domain split-tunneling
- Красивые графики и статистика
- Автоматическое обновление стратегий
- Поддержка нескольких DoH-серверов с failover
- IPv6
- Экспорт/импорт профилей
- Темная тема + кастомизация (уже будет из Material You)

## Технические требования к MVP

- Минимальная версия Android: **API 24** (Android 7.0) — для широкой совместимости
- Целевая: API 34+ (Android 14)
- NDK: r26+
- Kotlin 2.0+
- Compose BOM 2025.x

## Структура проекта (MVP)

```
app/
├── src/main/
│   ├── java/io/github/evgeniynec/zapdns/
│   │   ├── ui/                  # Compose screens
│   │   ├── domain/              # UseCases, Models
│   │   ├── data/                # Repositories, DataStore, Room
│   │   ├── vpn/                 # ZapDNSVpnService
│   │   ├── native/              # JNI + ByeDPI wrapper
│   │   ├── di/                  # Hilt или Koin
│   │   └── MainActivity.kt
│   ├── jni/                     # C/C++ (byedpi integration)
│   ├── res/
│   └── AndroidManifest.xml
├── build.gradle.kts
└── CMakeLists.txt
```

## План разработки MVP (первые 4–6 недель)

**Неделя 1–2**: Базовый скелет + VpnService + интеграция byedpi
- Форкнуть/изучить dovecoteescapee/ByeDPIAndroid или romanvht/ByeByeDPI
- Собрать byedpi под Android (NDK)
- Запустить простой VPN + byedpi без кастомного DNS

**Неделя 3**: DoH интеграция
- Реализовать DNS перехват + DoH клиент
- Тестирование на своём VPS

**Неделя 4**: UI + Профили + Автозапуск
- Jetpack Compose экраны
- DataStore + Room
- BootReceiver + TileService

**Неделя 5**: Полировка + Xray fallback (базовый)
- Переключатель полного туннеля (пока заглушка или простой Xray)
- Тестирование на реальных блокировках (YouTube, Discord и т.д.)

**Неделя 6**: Релиз 0.1 (debug) + сбор фидбека

## Критерии готовности MVP

- Приложение успешно подключается и обходит DPI на YouTube/Discord без полного VPN
- Свой DoH работает (проверяется через https://dnsleaktest.com или аналог)
- Автозапуск работает после перезагрузки
- Нет крашей на Android 10–14 (тестировать на нескольких устройствах)
- Код открыт и документирован

---

Готов начать разработку MVP сразу после утверждения архитектуры и стека.
