**Stability** — определяет, может ли Compose пропустить (skip) composable при рекомпозиции, сравнив аргументы.

### Три категории типов
1. **Stable**: Compose гарантирует, что при равенстве (`equals`) значение не изменится. Скейтинг работает.
2. **Unstable**: значение может измениться мимо `equals` (мутабельные поля) — сравнение бессмысленно, функция перезапускается всегда.
3. **Immutable**: подмножество stable — глубокая иммутабельность (все поля val + immutable типы).

### Что считается stable автоматически
- Примитивы, String.
- Классы, у которых **все публичные свойства и методы** stable, а класс помечен `@Stable`/`@Immutable` или является data class / value class (по правилам компилятора Compose: иммутабельные поля, equals/hashCode корректные).
- Классы с `@StableState` / MutableState-полями (состояние внутри отслеживается отдельно).
- Compose runtime типы (Modifier, State...).

### Классика проблемы: List из data class
```kotlin
data class News(val id: String, val title: String) // stable

// НО: List<News> — интерфейс. Реализация ArrayList — @Stable? НЕТ.
// Compose не знает, что ваш список не мутабелен → List<News> = unstable параметр
```
**Решения:**
1. Kotlinx Immutable Collections: `ImmutableList<News>` (`kotlinx.collections.immutable`) — stable из коробки.
2. Обернуть в stable-обёртку: `data class NewsList(val items: List<News>)` + `@Immutable` (обещание разработчика).
3. `@Stable`/`@Immutable` на свои типы — контракт «не буду менять мимо equals».

### Compose compiler reports
Диагностика: `./gradlew assembleRelease -PcomposeCompilerReports=true` → CSV/JSON: какие функции skippable, какие параметры unstable. Первый инструмент при оптимизации.

### Дополнительные правила
- `var` в data class делает его unstable? Нет, само по себе — нет (var с stable типом и корректным equals ок, но рекомендуют val). Но var с custom setter мутирующим что-то вне — проблема. На практике: val-only для моделей.
- Enum-классы stable.
- Функциональные типы (лямбды): stable, если не захватывают нестабильное. Каждый вызов composable создаёт новый экземпляр лямбды — но равенство лямбд — ссылочное, поэтому `remember` важен:
```kotlin
// Плохо: новая лямбда каждую рекомпозицию → дочерний может перезапускаться
Child(onClick = { viewModel.refresh() })

// Хорошо: стабильная ссылка
val onClick = remember { { viewModel.refresh() } }
Child(onClick = onClick)
// Или key: Child(onClick = remember(viewModel) { { viewModel.refresh() } })
```

### derivedStateOf — связь со stability
Когда производное состояние вычисляется из быстро меняющегося (scroll, drag), оборачиваем в `remember { derivedStateOf { ... } }` — рекомпозиция только при изменении результата, а не при каждом входном импульсе:
```kotlin
val showButton by remember { derivedStateOf { listState.firstVisibleItemIndex > 0 } }
```

### Стандартные вопросы
- «Что такое stability и зачем?» — для скейтинга: экономим рекомпозиции.
- «Почему data class всё ещё пересобирается?» — параметр `List<X>` — нестабильный по типу интерфейса.
- «Чем @Stable отличается от @Immutable?» — @Immutable гарантирует глубокую иммутабельность (все поля immutable), @Stable — только корректное equals + отслеживаемые изменения (например, через MutableState внутри).
