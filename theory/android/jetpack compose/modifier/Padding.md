Для установки отступов внутри компонента применяется модификатор `padding()`, которые имеет несколько вариантов:

```kotlin 
// устанавливает отступы от каждой стороны по отдельности
fun Modifier.padding(start: Dp = 0.dp, top: Dp = 0.dp, end: Dp = 0.dp, bottom: Dp = 0.dp): Modifier
// устанавливает отступы по вертикали и по горизонтали
fun Modifier.padding(horizontal: Dp = 0.dp, vertical: Dp = 0.dp): Modifier
// устанавливает одно значение для отступов от всех четырех сторон
fun Modifier.padding(all: Dp): Modifier
// устанавливает отступы в виде объекта PaddingValues
fun Modifier.padding(paddingValues: PaddingValues): Modifier
```

В качестве значения отступов применяются единицы `dp`.

#done