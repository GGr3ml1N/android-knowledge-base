Компонент `Box` является наиболее простым контейнером, позволяя позиционировать вложенное содержимое. Он представляет некоторую область экрана. Функция данного компонента принимает четыре параметра:

```kotlin
@Composable
inline fun Box(
    modifier: Modifier = Modifier,
    contentAlignment: Alignment = Alignment.TopStart,
    propagateMinConstraints: Boolean = false,
    content: @Composable BoxScope.() -> Unit
): @Composable Unit
```

- `modifier`: объект `Modifier`, который позволяет настроить внешний вид и поведение компонента с помощью модификаторов
- `contentAlignment`: объект `Alignment`, который устанавливает расположение компонента. По умолчанию имеет значение `Alignment.TopStart` (расположение вначале контейнера в верхнем углу)
- `propagateMinConstraints`: значение типа `Boolean`, который указывает, надо ли применять к содержимому ограничения по минимальным размерам. По умолчанию равно `false` (ограничения не применяются)
- `content`: объект интерфейса `BoxScope`, который представляет вложенное содержимое