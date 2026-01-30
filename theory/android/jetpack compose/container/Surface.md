Компонент Surface является ключевым компонентом компоновки в Material Design, предоставляя для вложенного содердимого множество стилизаций по умолчанию. Он имеет несколько версий. Возьмем простейшую из них:

```kotlin
@Composable
@NonRestartableComposable
fun Surface(
    modifier: Modifier = Modifier,
    shape: Shape = RectangleShape,
    color: Color = MaterialTheme.colorScheme.surface,
    contentColor: Color = contentColorFor(color),
    tonalElevation: Dp = 0.dp,
    shadowElevation: Dp = 0.dp,
    border: BorderStroke? = null,
    content: @Composable () -> Unit
)
```

Данная версия функции компонента принимает следующие параметры:
- `modifier`: применяемые к контейнеру функции модификатора
- `shape`: форма компонента в виде объекта `Shape`
- `color`: фоновый цвет
- `contentColor`: предпочитаемый цвет содержимого
- `tonalElevation`: эффект анимации при нажатии
- `shadowElevation`: высота тени
- `border`: параметры границы в виде объекта `BorderStroke`

При этом `Surface` может одновременно содержать только один компонент, поэтому при использовании множества компонентов, их следует оборачивать в другой контейнер типа `Box`, `Row` или `Column`

See also: [[Container compositions]]