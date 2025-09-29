# Smart Health Systems — Wear OS Companion

## 📌 Описание проекта
Приложение **Smart Health Systems** предназначено для умных часов на базе Wear OS. Оно собирает данные встроенных сенсоров (пульс, давление, температура окружающей среды), отображает их на экране часов и отправляет на удалённый сервер для дальнейшего анализа. Проект реализован на **Kotlin** с использованием **Jetpack Compose for Wear OS** и ориентирован на работу со сторонними медицинскими или фитнес‑платформами.

Основной сценарий работы:
1. При запуске приложение проверяет доступность ключевых сенсоров и регистрирует `SensorListener` для каждого найденного устройства.
2. При получении новых данных слушатель:
   * обновляет отображаемое значение на UI;
   * извлекает идентификатор пользователя из `SharedPreferences`;
   * отправляет показания на REST‑endpoint `https://aiigh.space/api/event/create/` в формате JSON.
3. На главном экране выводятся текущие значения пульса и давления, а также Android ID устройства (может использоваться как уникальный идентификатор, если пользовательский ID не задан).

## 🗂️ Структура репозитория
```
smart-health-systems-watch-wearos/
├── app/                       # Основной Android-модуль
│   ├── build.gradle           # Зависимости и настройки модуля
│   └── src/main/
│       ├── AndroidManifest.xml    # Разрешения, декларация активности и standalone-режима
│       ├── java/com/example/smart_health_systems_sw/
│       │   └── presentation/
│       │       ├── MainActivity.kt    # Точка входа, регистрация сенсоров, Compose UI
│       │       └── theme/Theme.kt     # Базовая тема Material для Wear OS
│       └── res/                   # Ресурсы приложения (строки, иконки и т.п.)
├── build.gradle                # Общие gradle-плагины
├── gradle/libs.versions.toml   # Версии зависимостей (Compose, Coroutines, Wear)
├── gradle.properties           # Дополнительные свойства сборки
├── settings.gradle             # Подключение модулей к сборке
└── gradlew / gradlew.bat, gradle/wrapper/   # Скрипты и конфигурация Gradle Wrapper
```

## ⚙️ Ключевые компоненты кода
### `MainActivity`
- Наследуется от `ComponentActivity` и выступает единственной экранной активностью.
- Создаёт `SensorManager`, запрашивает датчики `TYPE_HEART_RATE`, `TYPE_PRESSURE` и `TYPE_AMBIENT_TEMPERATURE` и регистрирует слушателей при запуске.
- Через `setContent` строит Compose-интерфейс: выводит BPM, давление и Android ID. (Поле для ввода пользовательского ID подготовлено, но закомментировано.)

### `SensorListener`
- Реализует `SensorEventListener` для выбранного сенсора.
- Сохраняет последнее измерение в `mutableStateOf`, чтобы Compose автоматически перерисовывал UI.
- Отправляет данные на сервер в корутине (`Dispatchers.IO`).
- Получает идентификатор пользователя из `SharedPreferences` (`MyPrefs.user_input`).

### Сеть и корутины
- Отправка данных выполняется вручную через `HttpURLConnection` с JSON-телом вида:
  ```json
  {
    "device_id": "<Android_ID>",
    "type": "heart-rate",
    "data": 78
  }
  ```
- Вся работа с сетью вынесена в suspend-функцию `sendDataToServer`, которая вызывается из корутины `SensorListener`.

### UI и тема
- Базовая тема определена в `SmarthealthsystemsswTheme`, откуда можно начать настройку цветовой схемы и типографики.
- Используется `MaterialTheme` для Wear OS, `TimeText` можно подключить при необходимости.

## 📦 Зависимости
Основные библиотеки подключаются через `gradle/libs.versions.toml`:
- **AndroidX Wear Compose** (`compose-material`, `compose-foundation`) — UI-компоненты для часов.
- **AndroidX Activity Compose** — интеграция Compose с Activity.
- **Kotlin Coroutines** — асинхронная отправка данных на сервер.
- **Play Services Wearable** — интеграция с экосистемой Wear (опционально, при необходимости).
- **SplashScreen API** — нативный Splash на старте приложения.

## 🚀 Запуск и сборка
1. Установите **Android Studio Hedgehog (или новее)** и **JDK 17+** (требование AGP 8.4.0).
2. Клонируйте репозиторий:
   ```bash
   git clone https://gitflic.ru/project/juxel/smart-health-systems-watch-wearos.git
   cd smart-health-systems-watch-wearos
   ```
3. Откройте проект в Android Studio, дождитесь синхронизации Gradle.
4. Выберите Wear OS устройство (реальное или эмулятор). Для тестирования сенсоров предпочтительно использовать реальные часы, так как эмулятор не предоставляет показания пульса/давления.
5. Запустите конфигурацию **Run → Run 'app'**.

### Права и разрешения
В `AndroidManifest.xml` уже указаны:
- `android.permission.BODY_SENSORS` — доступ к биометрическим данным.
- `android.permission.ACTIVITY_RECOGNITION`, `android.permission.FOREGROUND_SERVICE`, `android.permission.INTERNET`, `android.permission.READ_PHONE_STATE` — вспомогательные разрешения для телеметрии и сетевых запросов.
На устройствах с Wear OS 3+ доступ к датчикам пульса требуется подтверждать вручную через системный диалог.

## 🧪 Тестирование и отладка
- Для просмотра логов используйте `adb logcat` и фильтруйте по тегу `smart-health-systems`.
- Чтобы проверить сетевые запросы, можно добавить промежуточный прокси (например, mitmproxy) или временно заменить `urlString` в `sendDataToServer` на тестовый endpoint.
- Для отключения реальной отправки данных закомментируйте вызов `sendDataToServer` внутри `SensorListener.onSensorChanged`.

## 🔧 Дополнительная настройка
- **Пользовательский идентификатор**: раскомментируйте `UserInputField()` в `MainActivity` и добавьте UI на экране, чтобы вводить идентификатор пользователя прямо с часов. Значение будет храниться в `SharedPreferences` и подставляться в каждую отправку.
- **Новые датчики**: добавьте дополнительные `Sensor`/`SensorListener` по аналогии с существующими тремя блоками.
- **Смена адреса сервера**: измените `urlString` в `sendDataToServer`. Рекомендуется вынести адрес в конфигурацию или BuildConfig для удобства.

## 📄 Лицензия и контакты
Лицензия не указана. При необходимости добавьте файл `LICENSE` и раздел с контактами команды.

---
Если вы только начинаете работу с проектом, начните с изучения `MainActivity.kt` и запусков на реальном устройстве — это даст наглядное понимание того, какие данные собираются и как они попадают на сервер.
