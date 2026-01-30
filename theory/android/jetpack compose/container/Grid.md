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

- `rows`: представляет тип `GridCells`, который описывает количество и размер строк
- `modifier`: применяемые к контейнеру модификаторы
- `state`: объект состояния `LazyGridState`, применяемый для управления состоянием контейнера
- `contentPadding`: отступы вокруг содержимого
- `reverseLayout`: при значении `true` располагает элементы в обратном порядке
- `verticalArrangement`: настройки расположения элементов по горизонтали
- `horizontalArrangement`: выравнивание элементов по вертикали
- `flingBehavior`: описывает поведение при таком типе прокрутки, когда пользователь быстро перетаскивает что-то и поднимает палец. Представляет объект `FlingBehavior`
- `userScrollEnabled`: указывает, доступна ли прокрутка жестами либо через специальные инструменты управления доступом
- `content`: устанавливает содержимое контейнера с помощью функции типа `LazyGridScope.() -> Unit`.

Для установки содержимого интерфейс `LazyGridScope` предоставляет несколько функций:
- `LazyListScope.item()`: для добавления одного элемента
- `LazyListScope.items()`: для добавления нескольких элементов
- `LazyListScope.itemsIndexed()`: для добавления нескольких элементов с использованием индексов

Для определения количества и размера ячеек применяется интерфейс `GridCells`. В частности, он предоставляет следующие методы:
- `GridCells.Adaptive`: определяет сетку с максимально возможным количеством строк или столбцов при условии, что каждая ячейка имеет как минимум пространство `minSize` и все дополнительное пространство распределено равномерно.
- `GridCells.Исправлено`: определяет сетку с фиксированным количеством строк или столбцов.
- `GridCells.FixedSize`: определяет сетку с максимально возможным количеством строк или столбцов при условии, что каждая ячейка занимает ровное пространство.

Например, определим простейший грид из текстовых элементов:

```kotlin
val itemsList = (0..12).toList()
LazyHorizontalGrid(
    rows = GridCells.Fixed(3),
    modifier = Modifier.fillMaxSize()
) {
    items(itemsList) {item ->
        Text(
	        text = "Item $item", 
	        fontSize = 28.sp,
	        modifier= Modifier
		        .padding(8.dp)
	    )
    }
}
```
![[lazy_horizontal_grid.png]]

##### LazyVerticalGrid
`LazyVerticalGrid` отображает элементы в вертикально прокручиваемом контейнере, распределенном по нескольким столбцам, и имеет следующие параметры:

```kotlin
@Composable
fun LazyVerticalGrid(
    columns: GridCells,
    modifier: Modifier = Modifier,
    state: LazyGridState = rememberLazyGridState(),
    contentPadding: PaddingValues = PaddingValues(0.dp),
    reverseLayout: Boolean = false,
    verticalArrangement: Arrangement.Vertical = if (!reverseLayout) Arrangement.Top else Arrangement.Bottom,
    horizontalArrangement: Arrangement.Horizontal = Arrangement.Start,
    flingBehavior: FlingBehavior = ScrollableDefaults.flingBehavior(),
    userScrollEnabled: Boolean = true,
    content: LazyGridScope.() -> Unit
)
```

- `columns`: представляет объект `GridCells`, который описывает количество и размер столбцов
- `modifier`: применяемые к контейнеру модификаторы
- `state`: объект состояния `LazyGridState`, применяемый для управления состоянием контейнера
- `contentPadding`: отступы вокруг содержимого
- `reverseLayout`: при значении `true` располагает элементы в обратном порядке
- `verticalArrangement`: настройки расположения элементов по вертикали
- `horizontalArrangement`: выравнивание элементов по горизонтали
- `flingBehavior`: описывает поведение при таком типе прокрутки, когда пользователь быстро перетаскивает что-то и поднимает палец. Представляет объект `FlingBehavior`
- `userScrollEnabled`: указывает, доступна ли прокрутка жестами либо через специальные инструменты управления доступом
- `content`: устанавливает содержимое контейнера с помощью функции типа `LazyGridScope.() -> Unit`.

`LazyVerticalGrid` во многом аналогичен `LazyHorizontalGrid`.

```kotlin
data class Language(val name:String, val hexColor: Long)

val langs = listOf(
	Language("Kotlin", 0xff16a085),
    Language("Java", 0xff2980b9),
    Language("JavaScript", 0xff8e44ad),
    Language("Python", 0xff2c3e50),
    Language("Rust",0xffd35400),
    Language("C#",0xff27ae60),
    Language("C++",0xfff39c12),
    Language("Go",0xff1abc9c)
)

LazyVerticalGrid(
    columns = GridCells.Fixed(2),
    modifier = Modifier.fillMaxSize(),
    horizontalArrangement = Arrangement.Center
) {
    items(langs) { lang ->
        Column(
	        modifier = Modifier
				.padding(8.dp), 
			horizontalAlignment = Alignment
			.CenterHorizontally
		) {
            Box(
	            mofidier = Modifier
		            .size(100.dp)
			        .background(Color(lang.hexColor)))
            Text(
	            text = lang.name, 
		        fontSize = 24.sp,
		        modifier = Modifier.padding(5.dp)
		    )
        }
    }
}
```
![[lazy_vertical_grid.png]]

##### Адаптивные размеры

В примерах выше применялось точное количество строк/столбцов. Для этого использовался метод `GridCells.Fixed()`, в который передавалось количество строк/столбцов. Теперь рассмотрим другие возможности установки размеров. Так, метод `GridCells.Adaptive()`. Он определяет сетку с максимально возможным количеством строк или столбцов при условии, что каждая ячейка имеет как минимум размер `minSize` и все дополнительное пространство распределено равномерно. Значение `minSize`, которе указывает на минимаьную высоту строки или минимальную ширину столбца, передается в метод `GridCells.Adaptive()` в качестве единственного параметра.

```kotlin
data class Language(val name:String, val hexColor: Long)

val langs = listOf(
	Language("Kotlin", 0xff16a085),
    Language("Java", 0xff2980b9),
    Language("JavaScript", 0xff8e44ad),
    Language("Python", 0xff2c3e50),
    Language("Rust",0xffd35400),
    Language("C#",0xff27ae60),
    Language("C++",0xfff39c12),
    Language("Go",0xff1abc9c)
)
LazyVerticalGrid(
    columns = GridCells.Adaptive(minSize=120.dp),
    modifier = Modifier.fillMaxSize(),
    horizontalArrangement = Arrangement.Center
) {
    items(langs) {lang ->
        Column(
	        modifier = Modifier
		        .padding(7.dp), 
		    horizontalAlignment = Alignment
		    .CenterHorizontally
		) {
			Box(
               modifier = Modifier
					.size(100.dp)
					.background(Color(lang.hexColor))
			)
            Text(
	            text = lang.name, 
	            fontSize = 24.sp
	        )
        }
    }
}
```

Минимальную ширину в данном случае я определил на основе содержимого ячейки. В итоге при вертикальной ориентации на моем устройстве будет создано 3 столбца

![[adaptive_vertical_grid.png]]

Тогда как при горизонтальной ориентации будет создаваться аж 5 столбцов

![[adaptive_vertical_grid_h.png]]

##### Установка точных размеров

Еще один метод `GridCells.FixedSize()` позволяет задать точный размер ячейки (ширину столбцов или высоту строк). В этом случае создается максимально возможное количество строк или столбцов при условии, что каждая ячейка имеет указанный размер. Значение размера передается в метод `GridCells.FixedSize()` в качестве единственного параметра.

```kotlin
data class Language(val name:String, val hexColor: Long)

val langs = listOf(
	Language("Kotlin", 0xff16a085),
    Language("Java", 0xff2980b9),
    Language("JavaScript", 0xff8e44ad),
    Language("Python", 0xff2c3e50),
    Language("Rust",0xffd35400),
    Language("C#",0xff27ae60),
    Language("C++",0xfff39c12),
    Language("Go",0xff1abc9c)
)
LazyVerticalGrid(
	columns = GridCells.FixedSize(size=140.dp),
    modifier = Modifier.fillMaxSize(),
    horizontalArrangement = Arrangement.Center
) {
    items(langs) {lang ->
        Column(
	        modifier = Modifier
		        .padding(7.dp), 
		    horizontalAlignment = Alignment.CenterHorizontally
		) {
            Box(modifier = Modifier
	            .size(100.dp)
	            .background(Color(lang.hexColor))
	        )
            Text(
	            text = lang.name, 
		        fontSize = 24.sp
		    )
        }
    }
}
```

В итоге при вертикальной ориентации на моем устройстве будет создано 2 столбца

![[fixed_vertical_grid.png]]

Тогда как при горизонтальной ориентации будет создаваться 4 столбца

![[fixed_vertical_grid_h.png]]

See also: [[Sticky header]]