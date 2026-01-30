Кроме автоматической прокрутки с помощью пролистывания пальцев на экране устройства Jetpack Compose также поддерживает программную прокрутку, которую можно выполнить из кода Kotlin. Это может быть полезно, когда нам надо изменить текущую позицию в списке, перейти к какому-то определенному элементу в этом списке.

##### Column и Row
При работе со контейнерами `Column` и `Row` программную прокрутку можно выполнить с помощью функций объекта `ScrollState`:
- `animateScrollTo(value: Int)` - плавная прокрутка до указанной позиции пикселя в списке с использованием анимации
- `scrollTo(value: Int)` - мгновенная прокрутка до указанной позиции пикселя

Печальная сторона этих функций заключается в том, что их параметры представляют позицию в пикселях, а не номер элемента. То есть начало списка представлено позицией пикселя 0, но позиция пикселя, представляющая конец списка, может быть менее очевидной. К счастью, максимальную позицию прокрутки можно определить, обратившись к свойству `maxValue`:

```kotlin
val maxScrollPosition = scrollState.maxValue
```
Также стоит учитывать, что это suspend-функции, поэтому их необходимо вызывать из корутин.

```kotlin
val scrollState = rememberScrollState()
val coroutineScope = rememberCoroutineScope()

Column(Modifier.verticalScroll(scrollState)) {
	Text(
		text = "В конец", Modifier.padding(8.dp).background(Color.DarkGray).padding(5.dp).clickable {
                    coroutineScope.launch() {
                        scrollState.animateScrollTo(scrollState.maxValue)
                    }
                }, fontSize = 28.sp, color = Color.White)
                repeat(20){
                    Text("Item $it", Modifier.padding(8.dp), fontSize = 28.sp)
                }
            }
```