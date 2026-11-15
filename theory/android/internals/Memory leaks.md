**Утечка памяти** — объект больше не нужен, но остаётся reachable через цепочку сильных ссылок, GC не может его освободить. В Android последствия: рост памяти → срабатывание GC-джиттера (jank) → LMK убивает наш процесс раньше других.

### Классические утечки в Android
1. **Статические ссылки на Context/View**
```kotlin
// ПЛОХО: companion держит Activity после destroy
companion object { lateinit var currentActivity: Activity }
```
2. **Нестатические внутренние классы (Inner classes)**
```kotlin
// ПЛОХО: Handler анонимный класс держит внешний this (Activity)
// Если сообщение с задержкой переживает destroy — Activity утечёт
Handler(Looper.getMainLooper()).postDelayed({ ... }, 5000)
```
Решение: static nested class + WeakReference, или корутины с lifecycleScope (автоотмена).
3. **Listeners/callbacks без отписки** — BroadcastReceiver не зарегистрированный в onStop, Flow.collect без lifecycle (в Compose — collectAsStateWithLifecycle).
4. **Singleton'ы, кэши без eviction** — объект с неограниченным кэшем картинок/запросов (паттерн: LruCache).
5. **ViewModel держит View** — ViewModel переживает View; ссылка на View/контекст Activity во VM = утечка.
6. **WebView** — известный источник утечек (Context + нативные ссылки); выносить в отдельный процесс или тщательно уничтожать.
7. **Thread без управления** — поток живёт дольше Activity и держит её ссылку (вместо этого — coroutineScope или Executor с lifecycle).
8. **RxJava Disposables** — подписка не добавлена в CompositeDisposable, живёт после destroy (см. [[Basics]] Dagger/Rx раздел).

### Слабые ссылки — инструмент, а не решение
`WeakReference<Activity>` позволяет GC собрать Activity, но код с WeakReference хрупок (в любой момент get() вернёт null). Предпочтительнее архитектурные решения: lifecycle-aware коллекторы, отписка, coroutine scopes.

### Инструменты обнаружения
- **LeakCanary** (square): автоматически в debug: смотрит destroyed Activity/Fragment, делает heap dump, строит цепочку ссылок (leak trace) — по ней видно, КТО держит объект.
- **Android Studio Profiler / Memory Analyzer (MAT)**: heap dump, dominator tree, path to GC roots.
- **StrictMode.detectLeakedClosableObjects / ActivityLeaks** (базовые проверки).

### Пример рассуждения на собесе
«Пользователь жалуется, что приложение съедает память при навигации туда-обратно». План: LeakCanary в debug воспроизводит сценарий → leak trace показывает: `StaticNewsCache.images` держит `Bitmap`, который держит... → фикс: кэш с LRU-лимитом + очистка в onTrimMemory; репорт подтверждает отсутствие утечки.

### onTrimMemory / onLowMemory
Система сообщает приложению об уровне памяти (`onTrimMemory(TRIM_MEMORY_RUNNING_LOW / UI_HIDDEN / ...)`) — место для освобождения кэшей. Это хороший тон и вопрос: «как приложение ведёт себя при нехватке памяти?»
