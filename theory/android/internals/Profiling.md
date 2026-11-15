Инструменты, которые ожидают увидеть в арсенале Senior'а для диагностики производительности.

### Android Studio Profiler
- **CPU**: метод-трейсинг (Sampled/Instrumented — instrumented точнее, но искажает; sampled — меньше overhead), потоки, coroutine tracing (с coroutines-debug).
- **Memory**: heap dump (классы, instances, references), allocation tracking (где создаются объекты), timeline памяти против событий.
- **Network**: трафик по запросам (с Network Inspector).
- **Energy/Power**: расход батареи по компонентам.

### Perfetto / Systrace
Системный трейс: sched (планировщик CPU), freq, gfx (SurfaceFlinger), binder, аммуниция против «почему фрейм пропущен». Читаем дорожки: UI-thread → RenderThread → SurfaceFlinger → VSYNC. Показывает, кто виноват: наша view-логика, GPU, или система.

### Macrobenchmark / Microbenchmark (Jetpack)
- **Microbenchmark**: измерение конкретной функции (nanosecond-precision), влияние компиляции (baseline profiles, DEX layout).
- **Macrobenchmark**: cold/warm/hot startup, scrolling-метрики, frame timing на реальном устройстве; выгружает трассировки для Perfetto. Результаты — в CI как перформанс-тесты.

### Baseline Profiles (важно для собеса 2024+)
AOT-компиляция критичных путей заранее (при установке/обновлении), минуя интерпретатор и JIT. Даёт ускорение cold start и jank-метрик (особенно Compose, где первые композиции дорогие). Генерация: макробенчмарки (`BaselineProfileGenerator`) собирают профиль, поставляется в APK/AAB, Android компилирует перечисленные классы/методы AOT.

### Play Vitals / мониторинг в проде
- **Android Vitals** (Play Console): ANR-rate, crash-rate, slow/frozen frames, excessive wakeups, network/battery. Влияет на ранжирование.
- **Firebase Performance / App Metrics**: кастомные трейсы (cold start, экраны), network-timings, percentile-based (p50/p90/p99) — среднее скрывает проблемы хвоста.

### Эталонный сценарий ответа «как искали проблему производительности»
1. Симптом из Vitals: p99 cold start 4.5 сек на low-end.
2. Macrobenchmark + Perfetto: Application.onCreate — 1.8 сек (инициализация 12 SDK в т.ч. аналитика), Activity onCreate ещё 1.2 (main-thread DB-query).
3. Фиксы: Jetpack App Startup с инициализацией по требованию, аналитика — асинхронно, DB — Room + Dispatcher.IO, Baseline Profile для навигации.
4. Результат: p99 -> 2.1 сек, Vitals-граница соблюдена (cold start должен быть < 5 сек; критично для бизнес-метрик конверсии).
