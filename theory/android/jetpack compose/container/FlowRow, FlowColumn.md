Flow-контейнеры (`FlowRow` и `FlowColumn`) предназначены для потокового размещения содержимого, когда содержимое автоматические помещается на следующую строку или столбец, если в текущей строке/столбце место заканчилось.

##### FlowRow
`FlowRow` располагает элементы слева направо при левосторонней ориентации или справа налево при правосторонней ориентации, а когда заканчивается место, переходит к следующей строке и продолжает заполнять компоненты, пока они не закончатся. Этот компонент имеет следующие параметры:

```kotlin
@Composable
@ExperimentalLayoutApi
fun FlowRow(
    modifier: Modifier = Modifier,
    horizontalArrangement: Arrangement.Horizontal = Arrangement.Start,
    verticalArrangement: Arrangement.Vertical = Arrangement.Top,
    maxItemsInEachRow: Int = Int.MAX_VALUE,
    maxLines: Int = Int.MAX_VALUE,
    overflow: FlowRowOverflow = FlowRowOverflow.Clip,
    content: @Composable FlowRowScope.() -> Unit
)
```

- `modifier`: применяемые к компоненту функции модификатора.
- `horizontalArrangement`: расположение вложенных компонентов по горизонтали.
- `verticalArrangement`: расположение вложенных компонентов по вертикали.
- `maxItemsInEachRow`: максимальное количество элементов в строке
- `maxLines`: максимальное количество строк
- `overflow`: принцип переноса элементов
- `content`: содержимое контейнера

```kotlin
data class Rect(val width: Float, val hexColor: Long)

val rects = listOf(
    Rect(50f,0xff16a085), Rect(100f,0xff8e44ad),
    Rect(75f,0xff2980b9), Rect(125f,0xff2c3e50),
    Rect(100f,0xfff39c12), Rect(75f,0xff27ae60),
    Rect(50f,0xffd35400), Rect(110f,0xfff6b93b),
    Rect(100f,0xff0a3d62), Rect(75f,0xffb71540)
)
FlowRow(
	modifier = Modifier.fillMaxSize(), 
	maxItemsInEachRow = 4) {
        rects.forEach {rect ->
            Box(
	            modifier = Modifier.size(Dp(rect.width), 100.dp)
	            .padding(5.dp)
	            .background(Color(rect.hexColor))
	        )
    }
}
```
![[flow_row.png]]

##### FlowColumn
`FlowColumn` располагает элементы сверху вниз, а когда в столбце заканчивается место, переходит к следующему столбцу и продолжает размещать элементы, пока они не закончатся. Этот компонент имеет следующие параметры:

```kotlin
@Composable
@ExperimentalLayoutApi
fun FlowColumn(
    modifier: Modifier = Modifier,
    verticalArrangement: Arrangement.Vertical = Arrangement.Top,
    horizontalArrangement: Arrangement.Horizontal = Arrangement.Start,
    maxItemsInEachColumn: Int = Int.MAX_VALUE,
    maxLines: Int = Int.MAX_VALUE,
    overflow: FlowColumnOverflow = FlowColumnOverflow.Clip,
    content: @Composable FlowColumnScope.() -> Unit
)
```

- `modifier`: применяемые к компоненту функции модификатора.
- `horizontalArrangement`: расположение вложенных компонентов по горизонтали.
- `verticalArrangement`: расположение вложенных компонентов по вертикали.
- `maxItemsInEachColumn`: максимальное количество элементов в столбце
- `maxLines`: максимальное количество строк
- `overflow`: принцип переноса элементов
- `content`: содержимое контейнера

```kotlin
val rects = listOf(
    Rect(50f,0xff16a085), Rect(100f,0xff8e44ad),
    Rect(75f,0xff2980b9), Rect(125f,0xff2c3e50),
    Rect(100f,0xfff39c12), Rect(75f,0xff27ae60),
    Rect(50f,0xffd35400), Rect(110f,0xfff6b93b),
    Rect(100f,0xff0a3d62), Rect(75f,0xffb71540)
)
FlowColumn(
	modifier = Modifier.fillMaxSize(), 
	maxItemsInEachColumn = 4
) {
    rects.forEach {rect ->
        Box(
	        modifier = Modifier
		        .size(100.dp, Dp(rect.height))
		        .padding(5.dp)
		        .background(Color(rect.hexColor))
        )
    }
}
```

