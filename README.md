# ZapDNS

**Гибридный DPI-bypass + собственный DNS для Android (и позже Windows/Linux)**

Лёгкая, быстрая и контролируемая альтернатива тяжёлому VPN для обхода DPI и цензуры.  
Сочетает локальный DPI-байпас (на базе [hufrea/byedpi](https://github.com/hufrea/byedpi)) с использованием **своего DNS-сервера на VPS** (DoH/DoT) и опциональным подключением к Xray-туннелю только когда нужно.

> **Преимущества перед обычным VPN:**
> - Выше скорость (большая часть трафика идёт напрямую)
> - Меньше расход батареи и мобильного трафика
> - Больше контроля (профили, split-tunneling, свои DNS)
> - Не меняет IP без необходимости
> - Полная приватность — DNS на своём VPS, без логов третьих сторон

## 🎯 Цели проекта

Создать открытое, удобное приложение, которое:

1. Запускает **локальный DPI-bypass** (как zapret / ByeDPI) без необходимости полного туннеля.
2. Интегрируется со **своим DNS-сервером на VPS** (DoH).
3. Позволяет быстро переключаться между лёгким режимом и полным туннелем (Xray / sing-box).
4. Имеет удобные профили под задачи (YouTube + Discord, Игры, Максимальная скорость, Максимальная приватность).

## 📱 Фаза 1: Android (MVP)

### Основные функции MVP

| Функция              | Приоритет | Описание |
|----------------------|-----------|----------|
| **DPI Bypass**       | Высокий   | Запуск локального byedpi (SOCKS5) через VpnService |
| **Свой DNS (DoH)**   | Высокий   | Настройка и использование своего DoH-сервера с VPS |
| **Профили**          | Средний   | Пресеты: YouTube+Discord, Игры, Быстрый, Приватный |
| **Стратегии DPI**    | Средний   | Выбор стратегий (split, disorder, fake, и т.д.) |
| **Автозапуск**       | Высокий   | Запуск при загрузке устройства (Boot Receiver + VPN) |
| **Интеграция Xray**  | Средний   | Кнопка «Включить полный туннель» (fallback) |
| **Статистика**       | Низкий    | Какие домены/приложения идут через bypass |
| **Split-tunneling**  | Средний   | Per-app или per-domain routing (позже) |

**Ключевой принцип MVP**: Не изобретать DPI-движок заново. Использовать проверенный **hufrea/byedpi** как нативную библиотеку (как сделано в [dovecoteescapee/ByeDPIAndroid](https://github.com/dovecoteescapee/ByeDPIAndroid) и [romanvht/ByeByeDPI](https://github.com/romanvht/ByeByeDPI)).

## 🖥️ Фаза 2: Windows / ПК

- GUI-обёртка над zapret (или свой движок на базе того же byedpi)
- Управление службой (Windows Service)
- Интеграция с тем же DNS-сервером на VPS
- Split-tunneling по приложениям и доменам
- Единая кодовая база / общие стратегии с Android

## 🛠️ Технологии (Android)

- **Язык**: Kotlin
- **UI**: Jetpack Compose + Material You
- **VPN**: Android VpnService (основа)
- **Native**: CMake + NDK (сборка byedpi)
- **DPI Engine**: hufrea/byedpi (C) — как .so библиотека
- **Сетевой туннель**: hev-socks5-tunnel (опционально, как в ByeDPIAndroid)
- **DoH клиент**: OkHttp + dns-over-https или своя лёгкая реализация
- **Хранение**: DataStore (настройки) + Room (профили, статистика)
- **Архитектура**: MVVM + Clean Architecture (рекомендуется)

**Почему именно этот стек?**  
Самый рабочий и поддерживаемый путь в 2026 году для DPI-bypass приложений на Android.

## 📚 Основные репозитории-источники (изучить / форк)

1. [dovecoteescapee/ByeDPIAndroid](https://github.com/dovecoteescapee/ByeDPIAndroid) — лучшая архитектура VPN + byedpi
2. [hufrea/byedpi](https://github.com/hufrea/byedpi) — сам DPI-движок (C)
3. [romanvht/ByeByeDPI](https://github.com/romanvht/ByeByeDPI) — активный русский форк с хорошим UX
4. [bol-van/zapret](https://github.com/bol-van/zapret) — для понимания стратегий DPI (Linux/Windows)
5. [Flowseal/zapret-discord-youtube](https://github.com/Flowseal/zapret-discord-youtube) — готовые проверенные стратегии

## 🚀 Быстрый старт (пока в разработке)

```bash
# Клонирование (когда появится)
git clone https://github.com/EvgeniyNec/ZapDNS.git
cd ZapDNS

# Сборка Android (требуется Android Studio + NDK)
./gradlew assembleDebug
```

## 📖 Документация

- [ARCHITECTURE.md](ARCHITECTURE.md) — подробная архитектура приложения
- [ANDROID_MVP.md](ANDROID_MVP.md) — что входит в первую версию
- [TECH_STACK.md](TECH_STACK.md) — технологии, зависимости, почему именно они
- [INTEGRATION_VPS.md](INTEGRATION_VPS.md) — как подключить свой DNS и Xray с VPS
- [ROADMAP.md](ROADMAP.md) — поэтапный план разработки

## 🤝 Участие

Проект открытый. Идеи, Issues, Pull Requests приветствуются!

Особенно нужны:
- Тестировщики стратегий DPI на разных провайдерах (РФ)
- Помощь с интеграцией DoH
- Портирование нативной части
- UI/UX (Compose)

## 📄 Лицензия

GPL-3.0 (как у большинства проектов в этой нише)

---

**ZapDNS** — контроль над своим трафиком. Без лишнего VPN.  
Сделано с ❤️ к свободному интернету и open source.

Связь: [GitHub Issues](https://github.com/EvgeniyNec/ZapDNS/issues) | Telegram (позже)
