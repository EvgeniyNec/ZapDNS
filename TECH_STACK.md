# Tech Stack — ZapDNS

## Android (основная платформа)

| Область              | Технология                          | Версия     | Почему именно она |
|----------------------|-------------------------------------|------------|-------------------|
| **Язык**             | Kotlin                              | 2.0+       | Современный стандарт Android |
| **UI**               | Jetpack Compose + Material You      | BOM 2025.x | Декларативный, красивый, быстрый |
| **Архитектура**      | MVVM + Clean Architecture + UDF     | -          | Масштачируемость, тестируемость |
| **DI**               | Hilt (или Koin)                     | -          | Стандарт де-факто |
| **Навигация**        | Compose Navigation + Type Safe      | -          | Рекомендуется Google |
| **VPN**              | Android VpnService                  | API 24+    | Единственный способ перехватывать трафик |
| **Native код**       | CMake + NDK                         | r26+       | Сборка C-библиотек |
| **DPI Engine**       | hufrea/byedpi (C)                   | latest     | Лучший open-source DPI bypass на 2026 |
| **SOCKS туннель**    | hev-socks5-tunnel (опционально)     | -          | Используется в успешных проектах |
| **Хранение настроек**| DataStore (Preferences + Proto)     | -          | Асинхронный, типобезопасный |
| **База данных**      | Room                                | -          | Профили, статистика, история |
| **Сеть / DoH**       | OkHttp + собственный DoH клиент     | 4.12+      | Гибкость, контроль |
| **Асинхронность**    | Kotlin Coroutines + Flow            | -          | Стандарт |
| **Тестирование**     | JUnit5 + MockK + Turbine            | -          | Современный стек |

## Почему не Flutter?

- Flutter даёт кроссплатформенность, но:
  - Хуже интеграция с низкоуровневым VPN API и NDK
  - Больше overhead
  - Сложнее отлаживать нативные краши
- Для DPI-bypass критически важна **нативная производительность** и точный контроль над сетью.
- Позже можно рассмотреть **Kotlin Multiplatform** для общих модулей (стратегии DPI, DNS) между Android и Desktop.

## Почему не Rust / Go для нативной части?

- byedpi уже написан на **C** и отлично работает.
- Переписывание на Rust/Go — огромные трудозатраты с минимальным выигрышем на старте.
- Можно добавить Rust позже (через `cargo-ndk`) для новых фич (например, умный split-tunneling).

## Windows / Desktop (Фаза 2)

| Область         | Технология                     | Комментарий |
|-----------------|--------------------------------|-------------|
| GUI             | Compose for Desktop / Jetpack Compose + Skiko | Единый UI стек с Android |
| Служба          | Windows Service (Rust или C#)  | Или .NET |
| DPI Engine      | zapret (bol-van) или byedpi port | zapret уже имеет winws |
| Split-tunneling | WinDivert / WFP                | Низкоуровневый перехват |
| DNS             | Тот же DoH клиент              | Общий код |

**Альтернатива**: Electron + Tauri (если хотим быстро) — но теряем нативность.

## Рекомендуемые библиотеки (Android)

```kotlin
// build.gradle.kts (app)
dependencies {
    // Compose
    implementation(platform("androidx.compose:compose-bom:2025.05.00"))
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.material3:material3")
    implementation("androidx.compose.material:material-icons-extended")

    // Navigation
    implementation("androidx.navigation:navigation-compose:2.8.0")

    // DI
    implementation("com.google.dagger:hilt-android:2.51.1")
    ksp("com.google.dagger:hilt-compiler:2.51.1")

    // Storage
    implementation("androidx.datastore:datastore-preferences:1.1.1")
    implementation("androidx.room:room-ktx:2.6.1")
    ksp("androidx.room:room-compiler:2.6.1")

    // Network
    implementation("com.squareup.okhttp3:okhttp:4.12.0")
    implementation("com.squareup.okhttp3:okhttp-dnsoverhttps:4.12.0") // если подойдёт

    // Coroutines
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.9.0")

    // Lifecycle
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.8.0")
    implementation("androidx.lifecycle:lifecycle-runtime-compose:2.8.0")
}
```

## NDK / CMake (byedpi)

```cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.22.1)

project("zapdns")

add_library(byedpi SHARED IMPORTED)
set_target_properties(byedpi PROPERTIES
    IMPORTED_LOCATION ${CMAKE_SOURCE_DIR}/jniLibs/${ANDROID_ABI}/libbyedpi.so
)

add_library(${CMAKE_PROJECT_NAME} SHARED
    native-lib.cpp
)

target_link_libraries(${CMAKE_PROJECT_NAME}
    byedpi
    log
)
```

## Итог: Почему этот стек?

- Максимальная совместимость с существующими успешными проектами (ByeDPIAndroid, ByeByeDPI)
- Высокая производительность (C для DPI + Kotlin для UI)
- Легко поддерживать и расширять
- Открытый код + активное сообщество
- Возможность постепенного перехода на KMP / Desktop позже

---

**ZapDNS** выбирает **проверенные инструменты**, а не модные хайп-технологии.
