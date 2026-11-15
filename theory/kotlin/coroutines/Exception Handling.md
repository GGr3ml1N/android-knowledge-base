Обработка ошибок в корутинах — один из самых частых блоков вопросов на собесе.

### Главное правило
**ExceptionHandler ловит только ошибки, которые никто не перехватил, в корутинах launch-типа (root), на SupervisorJob.** Всё остальное — проброс вверх по иерархии.

### Схема поведения
1. Ошибка внутри `launch` (с обычным Job) → отмена братьев и родителя → исключение передаётся родителю → если нигде не поймано, в **root scope** с SupervisorJob его ловит `CoroutineExceptionHandler`, иначе — в `Thread.uncaughtExceptionHandler` (краш).

2. Ошибка в `async` → хранится в `Deferred` и вылетает **при `await()`**. Handler её не увидит, пока не вызовут `await`.

### CoroutineExceptionHandler
```kotlin
val handler = CoroutineExceptionHandler { ctx, throwable ->
    Log.e("App", "Uncaught in ${ctx[CoroutineName]}", throwable)
}
val scope = CoroutineScope(SupervisorJob() + Dispatchers.Default + handler)

scope.launch(CoroutineName("analytics")) {
    throw RuntimeException("boom") // попадёт в handler
}
```
Кладётся в контекст **корня** скоупа (или в launch уровня top-level). В дочерних корутинах бесполезен.

### Что делают в продакшене
```kotlin
class NewsViewModel : ViewModel() {
    private val _state = MutableStateFlow<State>(State.Idle)
    val state = _state.asStateFlow()

    fun load() {
        viewModelScope.launch {
            try {
                _state.value = State.Loading
                val news = repository.getNews()
                _state.value = State.Success(news)
            } catch (e: CancellationException) {
                throw e // обязательно!
            } catch (e: Exception) {
                _state.value = State.Error(e)
            }
        }
    }
}
```
Во ViewModel обрабатывают через `try/catch` вокруг репозитория — потому что результат нужно отразить в состоянии, а не просто залогировать.

### Хендлер в SupervisorJob-скоупе: async + handler
```kotlin
scope.launch {
    val a = async { api1() } // упал
    val b = async { api2() }
    // SupervisorJob: b не отменится, но a вылетит при await -> краш без try/catch
}
```
Поэтому паттерн: SupervisorJob для независимости + try/catch (или handler) для каждой ветки.

### CancellationException — сигнал, а не ошибка
`delay`, `withContext`, Flow-сбор при отмене бросают `CancellationException`. Его никогда не логируем как ошибку и не показываем пользователю — пробрасываем (см. [[Job and Cancellation]]).

### SupervisorJob + CoroutineExceptionHandler = связка
Именно в таком контексте корутины типа `launch` живут «вечно» по отдельности: упала одна — Handler её обработал, остальные работают.
