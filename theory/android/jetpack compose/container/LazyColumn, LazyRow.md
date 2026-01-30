Если необходимо отобразить большое количество элементов (или список неизвестной длины), использование стандартных контейнеров как `Column` или `Row` может вызвать проблемы с производительностью, так как все вложенные элементы будут скомпонованы внутри контейнера независимо от того, видимы они или нет. Для более эффективной работы со вложенными компонентами `Jetpack Compose` предоставляет такие компоненты-контейнеры как `LazyColumn` и `LazyRow`. Они компонуют и добавляют только те элементы, которые видны в окне просмотра компонента. При прокрутке в контейнер компонуются новые элементы, а старые удаляются. При обратной прокрутке происходит повторная компоновка старых элементов. Собственно название контейнеров уже говорит о том, что они производят так называюмую lazy-загрузку (или ленивую загрузку), когда элементы загружаются не сразу, а по мере необходимости.

##### LazyColumn

LazyColumn создает список с вертикальной прокруткой, а LazyRow создает список с горизонтальной прокруткой и имеет следующие параметры:

```kotlin
@Composable
fun LazyColumn(
    modifier: Modifier = Modifier,
    state: LazyListState = rememberLazyListState(),
    contentPadding: PaddingValues = PaddingValues(0.dp),
    reverseLayout: Boolean = false,
    verticalArrangement: Arrangement.Vertical = if (!reverseLayout) Arrangement.Top else Arrangement.Bottom,
    horizontalAlignment: Alignment.Horizontal = Alignment.Start,
    flingBehavior: FlingBehavior = ScrollableDefaults.flingBehavior(),
    userScrollEnabled: Boolean = true,
    content: LazyListScope.() -> Unit
)
```

- `modifier`: применяемые к контейнеру модификаторы
- `state`: объект состояния LazyListState, применяемый для управления состоянием контейнера
- `contentPadding`: отступы вокруг содержимого
- `reverseLayout`: при значении `true` располагает элементы в обратном порядке
- `verticalArrangement`: настройки расположения элементов по вертикали
- `horizontalAlignment`: выравнивание элементов по горизонтали
- `flingBehavior`: описывает поведение при таком типе прокрутки, когда пользователь быстро перетаскивает что-то и поднимает палец. Представляет объект `FlingBehavior`
- `userScrollEnabled`: указывает, доступна ли прокрутка жестами либо через специальные инструменты управления доступом
- `content`: устанавливает содержимое контейнера с помощью функции типа `LazyListScope.() -> Unit`.

Одно из отличий lazy-контейнеров, в частности, `LazyColumn` от других контейнеров, например, от `Column` состоит в принципе установки содержимого. За это отвечает функция типа `LazyListScope.() -> Unit`. Внутри блока этой функции можно использовать специальные методы для добавления других компонентов:

- `LazyListScope.item()`: для добавления одного элемента
- `LazyListScope.items()`: для добавления нескольких элементов
- `LazyListScope.itemsIndexed()`: для добавления нескольких элементов с использованием индексов

##### LazyRow
Контейнер LazyRow создает список с горизонтальной прокруткой. Он имеет следующие параметры:

```kotlin
@Composable
fun LazyRow(
    modifier: Modifier = Modifier,
    state: LazyListState = rememberLazyListState(),
    contentPadding: PaddingValues = PaddingValues(0.dp),
    reverseLayout: Boolean = false,
    horizontalArrangement: Arrangement.Horizontal = if (!reverseLayout) Arrangement.Start else Arrangement.End,
    verticalAlignment: Alignment.Vertical = Alignment.Top,
    flingBehavior: FlingBehavior = ScrollableDefaults.flingBehavior(),
    userScrollEnabled: Boolean = true,
    content: LazyListScope.() -> Unit
)
```

- `modifier`: применяемые к контейнеру модификаторы
- `state`: объект состояния LazyListState, применяемый для управления состоянием контейнера
- `contentPadding`: отступы вокруг содержимого
- `reverseLayout`: при значении true располагает элементы в обратном порядке
- `horizontalArrangement`: настройки расположения элементов по горизонтали
- `verticalAlignment`: выравнивание элементов по вертикали
- `flingBehavior`: описывает поведение при таком типе прокрутки, когда пользователь быстро перетаскивает что-то и поднимает палец. Представляет объект FlingBehavior
- `userScrollEnabled`: указывает, доступна ли прокрутка жестами либо через специальные инструменты управления доступом
- `content`: устанавливает содержимое контейнера с помощью функции типа LazyListScope.() -> Unit.

В целом LazyRow похож на LazyColumn, таким же образом устанавливает элементы, только располагает их по горизонтали. Например, простейший горизонтальный список:

```kotlin
val langs = listOf("Kotlin", "Java", "JavaScript", "Python")
LazyRow {
    items(langs) {lang -> Text(lang)}
}
```

See also: [[Sticky header]], [[Scroll state]]