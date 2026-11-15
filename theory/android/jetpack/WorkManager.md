**WorkManager** — библиотека для **гарантированного выполнения отложенной фоновой работы**, переживающей убийство приложения и перезагрузку устройства. Для работы «обязательно должна случиться, не обязательно сейчас».

### Когда WorkManager, а когда нет
| Задача | Инструмент |
| :--- | :--- |
| Обновить UI/сделать запрос, пока пользователь на экране | корутины + lifecycle |
| Долгая работа, видимая пользователю | Foreground Service |
| **Отложенная, гарантированная работа** (синхронизация, логи, отложенная отправка) | **WorkManager** |
| Точное время (будильник) | AlarmManager + exact alarms |

### Базовый API
```kotlin
val constraints = Constraints.Builder()
    .setRequiredNetworkType(NetworkType.CONNECTED)
    .setRequiresCharging(true)
    .setRequiresBatteryNotLow(true)
    .build()

val work = OneTimeWorkRequestBuilder<SyncWorker>()
    .setConstraints(constraints)
    .setBackoffCriteria(BackoffPolicy.EXPONENTIAL, 30, TimeUnit.SECONDS)
    .build()

WorkManager.getInstance(context).enqueueUniqueWork(
    "sync", ExistingWorkPolicy.REPLACE, work
)
```

```kotlin
class SyncWorker @AssistedInject constructor(
    @Assisted context: Context,
    @Assisted params: WorkerParameters,
    private val repository: SyncRepository
) : CoroutineWorker(context, params) {

    override suspend fun doWork(): Result = try {
        repository.sync()
        Result.success()
    } catch (e: Exception) {
        if (runAttemptCount < 5) Result.retry() else Result.failure()
    }
}
```

### Ключевые концепции
- **OneTimeWorkRequest** — одноразовая задача. **PeriodicWorkRequest** — периодическая (минимальный интервал **15 минут**, нет точного тайминга — батарея/Doze).
- **Constraints**: сеть, зарядка, idle, battery not low, storage. Задача выполнится, когда все условия выполнятся.
- **BackoffPolicy**: при `Result.retry()` повтор с экспоненциальным/линейным бэкоффом.
- **Retry vs Failure**: retry — transient (нет сети); failure — конец (не будет повторов).
- **Coroutines**: `CoroutineWorker` — `doWork()` suspend, отмена кооперативная.
- **Unique work + ExistingWorkPolicy**: KEEP (не трогать существующую), REPLACE (заменить), APPEND (в очередь после), APPEND_OR_REPLACE.

### Цепочки и зависимости
```kotlin
WorkManager.getInstance(context)
    .beginWith(uploadWork)
    .then(compressWork)
    .then(notifyWork)
    .enqueue()
```
Input/Output Data передаётся между звеньями (до 10 КБ).

### Наблюдение статуса
```kotlin
WorkManager.getInstance(context).getWorkInfoByIdLiveData(work.id)
    .observe(owner) { info -> /* ENQUEUED, RUNNING, SUCCEEDED, FAILED, BLOCKED, CANCELLED */ }
```
Состояние переживает перезапуск процесса (хранится в БД Room внутри WorkManager).

### Под капотом (вопрос Senior-уровня)
WorkManager использует: JobScheduler (API 23+), AlarmManager + BroadcastReceiver (ниже 23), в зависимости от версии ОС и наличия Google Play Services (GCMNetworkManager / Firebase JobDispatcher исторически). Расписание агрессивно, но учитывает Doze/App Standby — периодические задачи могут откладываться.

### Стандартные вопросы
- «Почему нельзя точно по времени через WorkManager?» — batching с другими задачами/соблюдение Doze, это trade-off батареи.
- «Что с задачей при убийстве процесса?» — ничего: планировщик ОС перезапустит worker позже, состояние сохранено в его БД.
- «WorkManager vs Foreground Service» — видимость/срочность: FGS работает немедленно с уведомлением; WM — гарантированно, но когда системе удобно. Начиная с WM 2.8 есть `setForeground()` для долгих worker'ов (экспедиции).
