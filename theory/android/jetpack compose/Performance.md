Compose-производительность = минимум рекомпозиций + лёгкая composition/layout/draw. Связано с [[Recomposition]], [[Stability]].

### Чек-лист оптимизаций

1. **Поднимайте состояние правильно (state hoisting)**: состояние ближе к потребителю = меньше область рекомпозиции.
2. **remember / rememberSaveable**: не создавать объекты заново (Paint, Path, formatter'ы, лямбды).
3. **Ключи списков**: `key(item.id) { Item(...) }` в LazyColumn/Column — стабильность идентичности элементов, особенно при вставках/удалениях.
4. **Lazy-списки**: `LazyColumn` с contentType для разнородных списков (лучший reuse), avoid nesting scrollable в scrollable, `items(count = ..., key = { ... })`.
5. **Modifier order**: порядок модификаторов влияет на layout/рисование и стоимость (background до padding = лишняя работа при наложении; clip до graphicsLayer и т.п.).
6. **Чтение состояния на правильной фазе**: для анимаций — `Modifier.graphicsLayer`/`drawWithContent` (draw phase) вместо пересоздания layout-параметров.
7. **Профилирование**: Layout Inspector (recomposition counts/highlight), Macrobenchmark + Baseline Profiles (особенно важно для Compose: композиция первых кадров ускоряется AOT).

### Чего избегать
- Создание View внутри composable (interop — помнить о costs двойного дерева).
- Тяжёлые вычисления в composition — в `remember` + `produceState`/`LaunchedEffect` с вычислением вне composition.
- Неглубокие, но широкие recompositions: гигантский экран-функция → разбивка на мелкие skippable-компоненты.

### Профилирование: что смотреть
- **Layout Inspector**: «Show Recomposition Counts» — какие composable перезапускаются и сколько раз. Подсветка: чем горячее цвет, тем больше рекомпозиций.
- **Baseline Profiles**: компиляция критичных composable AOT — измеряем через Macrobenchmark (`BaselineProfileGenerator`).
- **JankStats** / Macrobenchmark frame-timing: frozen/slow frames в Compose.

### Compose vs View performance-разговор
- Compose строит/сравнивает дерево на каждой рекомпозиции — без скейтинга это дороже View-invalidate вручную оптимизированного списка; со скейтингом и стабильными типами — сопоставимо или лучше за счёт меньшего overdraw и более умного layout.
- LazyLayout ~ RecyclerView с compose-переиспользованием: важны `key` и `contentType`.

### Вопросы уровня Senior
- «Как бы вы искали jank в Compose-экране?» — Layout Inspector recomposition counts → compiler stability report → Macrobenchmark trace (recomposition vs measure vs draw) → фикс (stability/derivedStateOf/keys).
- «Почему список на 10k элементов тормозит?» — скорее всего не ленивость (Column вместо LazyColumn), нестабильные item-типы, ключи по индексу при вставках, дорогой onBind-эквивалент в composition.
- «Как ускорить cold start Compose-приложения?» — Baseline Profiles (compose-runtime классы компилируются AOT), отложенная инициализация не-критичных графов, App Startup discipline.
