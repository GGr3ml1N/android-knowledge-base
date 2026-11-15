**ANR (Application Not Responding)** — диалог «Приложение не отвечает» появляется, когда главный поток не обрабатывает события слишком долго.

### Лимиты времени
| Сценарий | Лимит |
| :--- | :--- |
| BroadcastReceiver.onReceive (foreground) | ~10 сек |
| BroadcastReceiver.onReceive (background, ужесточено) | ~10-60 сек с версиями, но фактически минимум |
| Input event (клик/тач) | 5 сек |
| Service.onCreate/onStartCommand и т.д. | ~20 сек (ForegroundServiceDidNotStartInTimeException для FGS) |

Ключевой: **5 секунд без реакции UI-потока на input-событие**.

### Причины
1. **Блокировка главного потока**: сетевой вызов, чтение файла, тяжёлый цикл, `runBlocking` (классика: `runBlocking(Dispatchers.IO)`).
2. **Deadlock**: главный поток ждёт результат от фонового потока, а фоновый — от главного.
3. **Binder-транзакции**: чрезмерная межпроцессная коммуникация на главном потоке.
4. **Тяжёлая отрисовка**: render thread не успевает (skipped frames → при длительном jank выглядит как ANR).
5. Долгий `onCreate()`/`onResume()` Activity, долгий `onReceive()`.

### Диагностика
- **ANR traces**: `/data/anr/traces.txt` (системный) — дамп всех потоков в момент ANR: смотрим главный поток (чем заблокирован — `BLOCKED` на мониторе? в каком методе?).
- **StrictMode**: ловит диск/сеть на главном потоке в debug (`detectDiskReads/NetworkAccess`, `penaltyDeath`).
- **Android Studio Profiler / Perfetto**: CPU-трейсы, длительность фреймов.
- **ApplicationExitInfo** (API 30+): `getHistoricalProcessExitReasons` — причины выхода включая ANR.
- **Firebase Performance / Crashlytics ANR-отчёты**: stack главного потока в момент диалога.

### Как отвечать «как боролись с ANR»
Алгоритм: (1) нашли в трейсах, чем занят главный поток → (2) причина (например, SharedPreferences.apply/commit синхронный IO при большом файле, или JSON-парсинг) → (3) решение: перенос на Dispatchers.Default/IO, ленивая инициализация, батчинг операций → (4) превентивно: StrictMode в debug, CI-метрики cold start, мониторинг frozen frames в проде.

### Prevention-чеклист
- Нет блокирующих операций на Main (сеть, БД, файлы > крошечных).
- Холодный старт: Application.onCreate лёгкий, ContentProvider'ы-инициализаторы (Jetpack App Startup вместо авто-инициализации библиотек).
- BroadcastReceiver — триггер WorkManager, а не работа внутри.
- Тяжёлые вычисления — синхронизация с render thread'ом (Choreographer, постепенная загрузка списков).
