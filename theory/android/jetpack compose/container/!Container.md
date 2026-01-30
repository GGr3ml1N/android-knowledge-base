
В Jetpack Compose за расположение и компоновку компонентов в приложении отвечает специальный компонент - Layout. Он имеет следующую сигнатуру:

```kotlin
@Composable 
inline fun Layout(
    content: @Composable () -> Unit,
    modifier: Modifier = Modifier,
    measurePolicy: MeasurePolicy
)
```

Он имеет следующие параметры:
- `content`: Composable-функция, которая хранит все вложенные компоненты
- `modifier`: объект Modifier, которые устанавливает функции-модификаторы, применяемые для стилизации компонента
- `measurePolicy`: объект, который отвечает за вычисление размеров компонентов и их определение их расположения

Создание своего алгоритма вычисления размеров компонентов внутри компонента-контейнера, должная компоновка вложенных компонентов представляют трудоемкую работу, поэтому фреймворк Compose предоставляет ряд встроенных компонентов-контейнеров, которые автоматически выполняют эту работу. Основными из них являются `Box`, `Row` и `Column`.

See also: [[Box]], [[Column]], [[Row]], [[StaggeredGrid]], [[Container compositions]], [[LazyColumn, LazyRow]], [[Grid]], [[Surface]], [[FlowRow, FlowColumn]], [[IntrinsicSize]],
[[Scrolling]]