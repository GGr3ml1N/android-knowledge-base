Компоненты `LazyVerticalGrid` и `LazyHorizontalGrid` обеспечивают отображение элементов в виде сетки. Эти компоненты похожи на списки `LazyColumn/LazyRow`, только для описания содержимого применяют интерфейс `LazyGridScope`

##### LazyHorizontalGrid
LazyHorizontalGrid отображает элементы в горизонтально прокручиваемом контейнере, распределенном по нескольким строкам, и имеет следующие параметры:

```kotlin
@Composable
fun LazyHorizontalGrid(
    rows: GridCells,
    modifier: Modifier = Modifier,
    state: LazyGridState = rememberLazyGridState(),
    contentPadding: PaddingValues = PaddingValues(0.dp),
    reverseLayout: Boolean = false,
    horizontalArrangement: Arrangement.Horizontal = if (!reverseLayout) Arrangement.Start else Arrangement.End,
    verticalArrangement: Arrangement.Vertical = Arrangement.Top,
    flingBehavior: FlingBehavior = ScrollableDefaults.flingBehavior(),
    userScrollEnabled: Boolean = true,
    content: LazyGridScope.() -> Unit
)
```

- `rows`: представляет тип GridCells, который описывает количество и размер строк
- `modifier`: применяемые к контейнеру модификаторы
- `state`: объект состояния LazyGridState, применяемый для управления состоянием контейнера
- `contentPadding`: отступы вокруг содержимого
- `reverseLayout`: при значении true располагает элементы в обратном порядке
- `verticalArrangement`: настройки расположения элементов по горизонтали
- `horizontalArrangement`: выравнивание элементов по вертикали
- `flingBehavior`: описывает поведение при таком типе прокрутки, когда пользователь быстро перетаскивает что-то и поднимает палец. Представляет объект FlingBehavior
- `userScrollEnabled`: указывает, доступна ли прокрутка жестами либо через специальные инструменты управления доступом
- `content`: устанавливает содержимое контейнера с помощью функции типа LazyGridScope.() -> Unit.

Для установки содержимого интерфейс LazyGridScope предоставляет несколько функций:
- `LazyListScope.item()`: для добавления одного элемента
- `LazyListScope.items()`: для добавления нескольких элементов
- `LazyListScope.itemsIndexed()`: для добавления нескольких элементов с использованием индексов