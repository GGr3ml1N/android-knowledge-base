##### LazyVerticalStaggeredGrid
Компонент LazyVerticalStaggeredGrid представляет вертикальную сетку. Как и другие lazy-контейнеры, этот компонент компонует и размещает только элементы, которые видимые в данный момент на экране. Он имеет следующие параметры:

```kotlin
@Composable
fun LazyVerticalStaggeredGrid(
    columns: StaggeredGridCells,
    modifier: Modifier = Modifier,
    state: LazyStaggeredGridState = rememberLazyStaggeredGridState(),
    contentPadding: PaddingValues = PaddingValues(0.dp),
    reverseLayout: Boolean = false,
    verticalItemSpacing: Dp = 0.dp,
    horizontalArrangement: Arrangement.Horizontal = Arrangement.spacedBy(0.dp),
    flingBehavior: FlingBehavior = ScrollableDefaults.flingBehavior(),
    userScrollEnabled: Boolean = true,
    content: LazyStaggeredGridScope.() -> Unit
)
```

- `columns`: представляет размер и количество столбцов в виде объекта `StaggeredGridCells`
- `modifier`: применяемые к компоненту модификаторы
- `state`: состояние в виде объекта `LazyStaggeredGridState`. Для его создания применяется функция `rememberLazyStaggeredGridState()`
- `contentPadding`: отступы вокруг контента
- `reverseLayout`: указывает, будут ли элементы грида располагаться в обратном порядке
- `horizontalArrangement`: определяет расположение элементов по горизонтали
- `verticalItemSpacing`: вертикальные отступы между элементами
- `flingBehavior`: описывает поведение при таком типе прокрутки, когда пользователь быстро перетаскивает что-то и поднимает палец.
- `userScrollEnabled`: указывает, будет ли доступна прокрутка
- `content`: представляет функцию типа `LazyStaggeredGridScope.() -> Unit`, которая задает вложенные компоненты. Для добавления элементов в грид можно использовать функции `LazyStaggeredGridScope.items()` (для списка объектов) или `LazyStaggeredGridScope.item()` (для одного элемента)

```kotlin
LazyVerticalStaggeredGrid(
    columns = StaggeredGridCells.Fixed(3),  // 3 столбца
    modifier = Modifier.fillMaxSize(),
    contentPadding = PaddingValues(8.dp),
    horizontalArrangement = Arrangement.spacedBy(8.dp),
    verticalItemSpacing = 8.dp
) {
    items(50) {_ ->
        Box(
	        modifier = Modifier
	            .fillMaxWidth()
                .height(Random.nextInt(50, 200).dp)
                .background(
	                Color(
	                    Random.nextInt(255),
	                    Random.nextInt(255),
	                    Random.nextInt(255),
	                    255
                    )
	            )
        )
    }
}
```
![[staggered_grid_vertical.png]]

##### LazyHorizontalStaggeredGrid
Компонент `LazyHorizontalStaggeredGrid` представляет горизонтальную сетку. Как и другие lazy-контейнеры, этот компонент компонует и размещает только элементы, которые видимые в данный момент на экране. Он имеет следующие параметры:

```
@Composable
fun LazyHorizontalStaggeredGrid(
    rows: StaggeredGridCells,
    modifier: Modifier = Modifier,
    state: LazyStaggeredGridState = rememberLazyStaggeredGridState(),
    contentPadding: PaddingValues = PaddingValues(0.dp),
    reverseLayout: Boolean = false,
    verticalArrangement: Arrangement.Vertical = Arrangement.spacedBy(0.dp),
    horizontalItemSpacing: Dp = 0.dp,
    flingBehavior: FlingBehavior = ScrollableDefaults.flingBehavior(),
    userScrollEnabled: Boolean = true,
    content: LazyStaggeredGridScope.() -> Unit
)
```

- `rows`: представляет размер и количество строк в виде объекта `StaggeredGridCells`
- `modifier`: применяемые к компоненту модификаторы
- `state`: состояние в виде объекта `LazyStaggeredGridState`. Для его создания применяется функция `rememberLazyStaggeredGridState()`
- `contentPadding`: отступы вокруг контента
- `reverseLayout`: указывает, будут ли элементы грида располагаться в обратном порядке
- `verticalArrangement`: определяет расположение элементов по вертикали
- `horizontalItemSpacing`: горизонтальные отступы между элементами
- `flingBehavior`: описывает поведение при таком типе прокрутки, когда пользователь быстро перетаскивает что-то и поднимает палец.
- `userScrollEnabled`: указывает, будет ли доступна прокрутка
- `content`: представляет функцию типа `LazyStaggeredGridScope.() -> Unit`, которая задает вложенные компоненты. Для добавления элементов в грид можно использовать функции `LazyStaggeredGridScope.items()` (для списка объектов) или `LazyStaggeredGridScope.item()` (для одного элемента)

```kotlin
LazyHorizontalStaggeredGrid(
    rows = StaggeredGridCells.Fixed(3),
    modifier = Modifier.fillMaxSize(),
    contentPadding = PaddingValues(8.dp),
    horizontalItemSpacing = 8.dp,
    verticalArrangement = Arrangement.spacedBy(8.dp)
) {
	items(50) {_ ->
        Box(
	        modifier = Modifier
                .fillMaxHeight()
                .width(Random.nextInt(50, 200).dp)
                .background(Color(
                            Random.nextInt(255),
                            Random.nextInt(255),
                            Random.nextInt(255),
                            255)
                )
        )
    }
}
```
![[staggered_grid_horizontal.png]]
