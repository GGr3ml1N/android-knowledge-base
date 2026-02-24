### Прокрутка по горизонтали

Для добавления прокрутки содержимого по горизонтали применяется модификатор `Modifier.horizontalScroll()`:

```kotlin
fun Modifier.horizontalScroll(
    state: ScrollState,
    enabled: Boolean = true,
    flingBehavior: FlingBehavior? = null,
    reverseScrolling: Boolean = false
): Modifier
```

Параметры модификатора:
- `state`: представляет объект `ScrollState` и описывает состояние полосы прокрутки
- `enabled`: значение типа `Boolean`, которое указывает, будет ли прокрутка доступна. По умолчанию имеет значение `true`
- `flingBehavior`: представляет объект `FlingBehavior` и описывает поведение при завершении прокрутки. По умолчанию имеет значение `ScrollableDefaults.flingBehavior`
- `reverseScrolling`: устанавливает направление. При значении `true` прокрутка идет в обратном направлении. По умолчанию имеет значение `false`

### Прокрутка по вертикали

Для добавления прокрутки содержимого по вертикали применяется модификатор `Modifier.verticalScroll()`:

```kotlin
fun Modifier.verticalScroll(
    state: ScrollState,
    enabled: Boolean = true,
    flingBehavior: FlingBehavior? = null,
    reverseScrolling: Boolean = false
): Modifier
```

#done