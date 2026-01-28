Для установки фонового цвета применяется модификатор `background()`. Есть две версии этой функции. Первая версия:

```kotlin
fun Modifier.background(color: Color, shape: Shape = RectangleShape): Modifier
```

- `color`: значение цвета в виде объекта Color
- `shape`: закрашиваемая форма - объект Shape. По умолчанию это прямоугольная форма

Вторая версия:

```kotlin
fun Modifier.background(
    brush: Brush,
    shape: Shape = RectangleShape,
    alpha: Float = 1.0f
): Modifier
```

- `brush`: применяемая для покрытия цветом кисть в виде объекта Brush
- `shape`: закрашиваемая форма - объект Shape. По умолчанию это прямоугольная форма
- `alpha`: значение прозрачности. Оно должно находится в диапазоне от 0 (полностью прозрачно) до 1 (отсутствие прозрачности)

### Color. Определение цвета
В Jetpack Compose цвет представлен классом Color из пакета `androidx.compose.ui.graphics..`

Есть несколько способов создания цвета. Рассмотрим некоторые:

- `Color(value)`: в конструктор передается шестнадцатеричное значение цвета. 

```kotlin
val myColor: Color = Color(0xFF0000FF)
```

- `Color(red, green, blue, alpha)`: в конструктор передается числовые значения для каждой отдельной компоненты цвета.

```kotlin
val myColor: Color = Color(red = 0xFF, green = 0xFF, blue = 0xFF, alpha = 0xFF)
```