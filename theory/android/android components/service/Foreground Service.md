Привет, коллега. Foreground Service (FSS) — это один из самых "капризных" компонентов в современном Android. Если в эпоху Android 7 и ниже это был просто способ сказать системе "не убивай меня", то начиная с Android 8 (Oreo) и особенно в Android 13/14, это строгий контракт с ОС и пользователем.

Как Senior-разработчик, ты должен понимать FSS не просто как класс `Service`, а как **инструмент управления жизненным циклом и ресурсами устройства**.

Давай разберем теорию, эволюцию ограничений и правильную архитектуру.

---

### 1. Суть 
**Foreground Service** — это сервис, который выполняет задачу, заметную для пользователя. 
* **Главное правило:** Он **обязан** показать `Notification` (уведомление) в течение **5 секунд** после вызова `startForeground()`.
* **Приоритет:** Он имеет приоритет выше, чем фоновые процессы, и система старается его не убивать.
* **Исключения:** Он не защищен от "убийства" пользователем (свайп из недавних) или системой при экстремальной нехватке памяти (Low Memory Killer), но в 99% случаев он живет, пока работает задача.

---

### 2. Эволюция разрешений (Самое важное для Android 13/14)

Если ты пишешь под современные SDK, ты должен знать, что "просто так" запустить FSS больше нельзя. ОС требует явного указания **типа** задачи.

#### Android 9 (API 28)
Появилось базовое разрешение:
```xml
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
```

#### Android 13 (API 33) — Гранулярные разрешения
Google разделил `FOREGROUND_SERVICE` на подкатегории. Теперь ты должен просить разрешение под конкретный тип задачи:
```xml
<!-- Примеры -->
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_LOCATION" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_CAMERA" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_MEDIA_PLAYBACK" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_DATA_SYNC" />
```
*Если ты используешь `location`, но не попросил `FOREGROUND_SERVICE_LOCATION`, приложение крашнется с `SecurityException`.*

#### Android 14 (API 34) — Две новые боли
1. **Пользовательский контроль:** Пользователь теперь может отозвать разрешение на FSS для конкретного типа (например, запретить трекинг локации) в настройках, не удаляя приложение. Ты должен обрабатывать `SecurityException` при старте.
2. **Запрет на запуск из фона:** FSS больше нельзя запустить, если приложение находится в фоне (кроме исключений, например, при получении FCM high-priority пуша или при взаимодействии с системными UI).

---

### 3. ForegroundServiceType (Обязательно с Android 14)

Начиная с Android 10 (и строго обязательно с 14), ты должен указать `foregroundServiceType` в манифесте и при вызове `startForeground()`. Это говорит системе, какие ресурсы ты используешь, чтобы она могла применять специфичные правила (например, не давать доступ к микрофону, если тип `dataSync`).

**В Манифесте:**
```xml
<service
    android:name=".tracking.LocationTrackingService"
    android:foregroundServiceType="location"
    android:exported="false" />
```

**В коде (при старте):**
```kotlin
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
    startForeground(
        NOTIFICATION_ID,
        notification,
        ServiceInfo.FOREGROUND_SERVICE_TYPE_LOCATION
    )
} else {
    startForeground(NOTIFICATION_ID, notification)

Привет, коллега. Foreground Service (FSS) — это один из самых "капризных" компонентов в современном Android. Если в эпоху Android 7 и ниже это был просто способ сказать системе "не убивай меня", то начиная с Android 8 (Oreo) и особенно в Android 13/14, это строгий контракт с ОС и пользователем.

Как Senior-разработчик, ты должен понимать FSS не просто как класс `Service`, а как **инструмент управления жизненным циклом и ресурсами устройства**.

Давай разберем теорию, эволюцию ограничений и правильную архитектуру.

---

### 1. Суть и Отличия от Background Service

**Background Service** (обычный) работает, пока приложение в фоне, но система может убить его в любой момент для экономии батареи (Doze mode, App Standby).
**Foreground Service** — это сервис, который выполняет задачу, заметную для пользователя. 
* **Главное правило:** Он **обязан** показать `Notification` (уведомление) в течение **5 секунд** после вызова `startForeground()`.
* **Приоритет:** Он имеет приоритет выше, чем фоновые процессы, и система старается его не убивать.
* **Исключения:** Он не защищен от "убийства" пользователем (свайп из недавних) или системой при экстремальной нехватке памяти (Low Memory Killer), но в 99% случаев он живет, пока работает задача.

---

### 2. Эволюция разрешений (Самое важное для Android 13/14)

Если ты пишешь под современные SDK, ты должен знать, что "просто так" запустить FSS больше нельзя. ОС требует явного указания **типа** задачи.

#### Android 9 (API 28)
Появилось базовое разрешение:
```xml
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
```

#### Android 13 (API 33) — Гранулярные разрешения
Google разделил `FOREGROUND_SERVICE` на подкатегории. Теперь ты должен просить разрешение под конкретный тип задачи:
```xml
<!-- Примеры -->
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_LOCATION" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_CAMERA" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_MEDIA_PLAYBACK" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_DATA_SYNC" />
```
*Если ты используешь `location`, но не попросил `FOREGROUND_SERVICE_LOCATION`, приложение крашнется с `SecurityException`.*

#### Android 14 (API 34) — Две новые боли
1. **Пользовательский контроль:** Пользователь теперь может отозвать разрешение на FSS для конкретного типа (например, запретить трекинг локации) в настройках, не удаляя приложение. Ты должен обрабатывать `SecurityException` при старте.
2. **Запрет на запуск из фона:** FSS больше нельзя запустить, если приложение находится в фоне (кроме исключений, например, при получении FCM high-priority пуша или при взаимодействии с системными UI).

---

### 3. ForegroundServiceType (Обязательно с Android 14)

Начиная с Android 10 (и строго обязательно с 14), ты должен указать `foregroundServiceType` в манифесте и при вызове `startForeground()`. Это говорит системе, какие ресурсы ты используешь, чтобы она могла применять специфичные правила (например, не давать доступ к микрофону, если тип `dataSync`).

**В Манифесте:**
```xml
<service
    android:name=".tracking.LocationTrackingService"
    android:foregroundServiceType="location"
    android:exported="false" />
```

**В коде (при старте):**
```kotlin
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
    startForeground(
        NOTIFICATION_ID,
        notification,
        ServiceInfo.FOREGROUND_SERVICE_TYPE_LOCATION
    )
} else {
    startForeground(NOTIFICATION_ID, notification)
}
```

---

### 4. Архитектура и Чистый Код (Senior Approach)

Главная ошибка джунов — писать бизнес-логику внутри `Service`. 
**Service — это просто обертка над жизненным циклом Android.** Вся логика должна быть вынесена в UseCase или Manager.

#### Плохо (Нарушает SRP и KISS):
```kotlin
class TrackingService : Service() {
    override fun onStartCommand(...) {
        // Смесь логики трекинга, работы с БД и UI (Notification)
        locationManager.requestLocationUpdates(...) 
        db.saveLocation(...)
        showNotification(...)
    }
}
```

#### Хорошо (ООП, DRY, Делегирование):
```kotlin
class TrackingService : Service() {

    // Инжектим менеджер, который не знает про Android Service
    @Inject lateinit var locationTracker: LocationTrackerManager
    
    private val serviceScope = CoroutineScope(SupervisorJob() + Dispatchers.Main.immediate)

    override fun onCreate() {
        super.onCreate()
        // Показываем уведомление СРАЗУ (правило 5 секунд)
        startForeground(NOTIFICATION_ID, createNotification())
        
        // Запускаем логику в корутинах
        serviceScope.launch {
            locationTracker.startTracking().collect { location ->
                // Обновляем уведомление или сохраняем в БД
                updateNotification(location) 
            }
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        // Корректно останавливаем логику
        serviceScope.cancel()
        locationTracker.stopTracking()
    }

    override fun onBind(intent: Intent?): IBinder? = null
}
```

---

### 5. Уведомления (Notifications)

Уведомление для FSS должно быть **не удаляемым** (user cannot swipe it away), пока сервис работает.
* Используй `NotificationCompat.Builder`.
* Обязательно укажи `setOngoing(true)`.
* В Android 13+ для показа уведомлений нужно runtime-разрешение `POST_NOTIFICATIONS`. Если пользователь его не дал, FSS **не запустится** (краш или молчаливый фейл в зависимости от версии SDK).

---

### 6. Когда ИСПОЛЬЗОВАТЬ, а когда НЕТ

Как Senior, ты должен уметь отговаривать продактов от использования FSS, если задача не подходит.

**Использовать FSS, если:**
1. Пользователь явно ожидает непрерывной работы (музыка, навигация, запись аудио, фитнес-трекер).
2. Задача критична и не может быть прервана (загрузка большого файла, который пользователь явно нажал "Скачать").

**НЕ использовать FSS (Использовать альтернативы), если:**
1. **Синхронизация данных в фоне:** Используй **WorkManager** (с `Constraints` и `Expedited Work` для Android 12+).
2. **Периодические задачи:** Используй **WorkManager** или **AlarmManager** (для точных будильников).
3. **Короткие фоновые задачи (до 10 минут):** Используй **WorkManager** с `setExpedited()`.
4. **Медиа-плеер:** Не пиши свой FSS с нуля. Используй **Media3 / ExoPlayer** и `MediaSessionService`. Они уже реализуют правильный FSS с интеграцией в системный UI (Bluetooth, наушники, экран блокировки).

---

### 7. Чек-лист для ревью кода FSS

Если джун/мидл приносит PR с Foreground Service, проверь по пунктам:
1. [ ] Есть ли `startForeground()` в `onCreate()` или в самом начале `onStartCommand()`? (Если в `onStartCommand`, убедись, что нет путей, где он не вызовется).
2. [ ] Указан ли `foregroundServiceType` в манифесте и в `startForeground()`?
3. [ ] Запрошены ли специфичные разрешения (API 33+) в манифесте?
4. [ ] Проверяется ли наличие runtime-разрешения `POST_NOTIFICATIONS` перед стартом?
5. [ ] Обрабатывается ли `SecurityException` при вызове `startForeground()` (на случай, если пользователь отозвал разрешение в настройках на Android 14)?
6. [ ] Вынесена ли бизнес-логика из `Service` в отдельный класс?
7. [ ] Корректно ли останавливается сервис и освобождаются ресурсы (камеры, GPS, корутины) в `onDestroy()`?

**Резюме:** Foreground Service в современном Android — это "тяжелая артиллерия". Используй её только когда другие средства (WorkManager, FCM) не справляются, и всегда помни про строгие требования ОС к уведомлениям и типам задач.

Если нужно углубиться в конкретную тему (например, как правильно связать FSS с MediaSession или как обойти ограничения запуска из фона на Android 14) — дай знать!
#in_progress 