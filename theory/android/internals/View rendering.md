### Два потока: UI и Render
- **UI thread (main)**: строит/изменяет иерархию View, вычисляет лэйаут, записывает display list.
- **RenderThread**: выполняет GPU-команды (начиная с Android 5/6 большинство графических операций вынесено туда, главный поток не блокируется синхронизацией с GPU).

### Что происходит при invalidate() — три фазы
1. **Measure** — `onMeasure()`: дерево View измеряется сверху вниз (AT_MOST/EXACTLY/UNSPECIFIED). Дорого на глубокой иерархии.
2. **Layout** — `onLayout()`: каждому View присваиваются координаты (left/top/right/bottom).
3. **Draw** — `onDraw()`: View рисует себя на Canvas (display list / recording). Перерисовка происходит не сразу, а по сигналу VSYNC (Choreographer).

### Choreographer и VSYNC
- Дисплей обновляется с фиксированной частотой (60/90/120 Гц). Каждый VSYNC-пульс Choreographer запускает callback'и: input → animation → traversal (measure/layout/draw).
- Кадр должен быть готов к следующему VSYNC. Уложились — 60 fps; не уложились — «skipped frame» (jank). Для 120 Гц бюджет ~8 мс на кадр.

### Перерисовка: как уменьшить
- **invalidate()** перерисует всю View. **invalidate(Rect)** / **invalidate(l,t,r,b)** — только область.
- Сложный `onDraw` — кэшировать в Bitmap, уменьшать overdraw (полупрозрачные слои поверх всего экрана), не создавать объекты в onDraw (allocations в горячем пути → GC).
- `clipRect`/`quickReject` для отсечения невидимых областей.
- Overdraw: 4x+ окраска в «Overdraw»-режиме отладки — убирать лишние фоны.

### Hierarchy Viewer / Layout Inspector
Глубина дерева влияет на measure/layout линейно. Flat-иерархия (ConstraintLayout, Compose без глубокой вложенности Column/Row) решает.

### Терминология собеса
- **Jank** — пропущенные кадры, заметные подёргивания.
- **Frozen frames** — кадры > 700 мс (считаются серьёзными в Play Vitals).
- **Slow frames / UI freezes**: мониторинг через FrameMetricsAggregator, JankStats (Jetpack), Macrobenchmark.

### Почему это важно для Compose
Compose: рекомпозиция ~ аналог invalidate; модификаторы влияют на measure/layout аналогично. Понимание VSYNC помогает объяснить, почему тяжёлая работа в рекомпозиции — это jank, и зачем `derivedStateOf`, `remember`, стабильные типы (см. [[!Jetpack Compose]]).

### Стандартные вопросы
- «Почему нельзя делать работу в onDraw?» — вызывается каждый кадр; allocations → GC-паузы → dropped frames; тяжёлая логика → пропуск VSYNC.
- «Как ускорить список?» — ViewHolder/Compose Lazy, уменьшить глубину иерархии, убрать overdraw, AsyncLayoutInflater, prefetching (RecyclerView).
