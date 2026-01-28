Для сдвига содержимого компонента по горизонтали и вертикали применяется модификатор `offset()`. Он имеет следующие версии:

```kotlin
fun Modifier.offset(x: Dp = 0.dp, y: Dp = 0.dp): Modifier
fun Modifier.offset(offset: Density.() -> IntOffset): Modifier
```

В качестве значения отступов применяются единицы `dp`.