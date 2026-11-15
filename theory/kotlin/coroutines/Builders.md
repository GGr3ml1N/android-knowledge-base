### 1. launch
Запускает корутину, которая **ничего не возвращает** (аналог `void` / `Completable` в Rx).
```kotlin
val job = scope.launch {
    doNetworkCall()
}
```
Возвращает `Job` — ссылку на запущенную корутину, через которую можно отменить её или дождаться.

### 2. async
Запускает корутину, которая **возвращает результат** через `Deferred<T>` (аналог `Single`/`Observable` в Rx).
```kotlin
val deferred = scope.async {
    loadUser()
}
val user = deferred.await() // приостанавливает корутину, не блокируя поток
```

**Классическая ловушка:** `async` не выполняется, пока не вызван `await()`? Нет — выполняется сразу (eager по умолчанию), `await()` только приостанавливает вызывающий код до готовности результата.

**Параллельное выполнение двух запросов:**
```kotlin
val user = scope.async { api.getUser() }
val orders = scope.async { api.getOrders() }
// оба запроса идут ПАРАЛЛЕЛЬНО, пока мы не дойдем до await
render(user.await(), orders.await())
```
Если написать `render(scope.async { ... }.await(), ...)` последовательно — запросы пойдут друг за другом.

### 3. runBlocking
Блокирует текущий поток, пока корутина внутри не завершится.
```kotlin
runBlocking { delay(1000) } // поток стоит 1 секунду
```
**Только для main()-функций и тестов/скриптов. НИКОГДА не использовать в Android-коде** — это приведёт к jank или ANR.

### 4. coroutineScope / supervisorScope
Подходят для **композиции** suspend-функций: запускают блок кода в новом scope и приостанавливают вызывающий код до завершения всех дочерних корутин.
```kotlin
suspend fun loadDashboard(): Dashboard = coroutineScope {
    val user = async { api.getUser() }
    val news = async { api.getNews() }
    Dashboard(user.await(), news.await())
}
```
Отличие `supervisorScope` — дочерняя ошибка не роняет сиблингов (см. [[SupervisorJob]]).

### Когда какой builder?
| Нужно | Builder |
| :--- | :--- |
| «Просто выполнить» (fire-and-forget) | `launch` |
| Получить результат из корутины | `async` + `await()` |
| Скомпоновать несколько suspend-вызовов | `coroutineScope` |
| Тесты, main() | `runBlocking` |
