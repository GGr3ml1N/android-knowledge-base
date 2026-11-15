### Проблема обычного Job
В обычном `Job` ошибка в одной дочерней корутине **отменяет всех сиблингов** и роняет родителя:
```kotlin
val scope = CoroutineScope(Job() + Dispatchers.Default)
scope.launch {
    delay(100); throw RuntimeException("boom")
}
scope.launch {
    delay(1000); println("не выполнится — отменили") 
}
```

### SupervisorJob
`SupervisorJob` отличается одним: **отказ ребёнка не влияет на остальных детей и на родителя**.
```kotlin
val scope = CoroutineScope(SupervisorJob() + Dispatchers.Default)
scope.launch { throw RuntimeException() }  // упала эта
scope.launch { delay(1000); println("живём") } // эта доработает
```
Это критично для UI-скоупов: ошибка загрузки аватарки не должна убивать корутину обработки кликов.

**Стандартный вопрос:** отличается ли поведение от `try/catch` внутри корутины? Нет — супервизор работает на уровне иерархии Job, а не кода.

### Где уже есть SupervisorJob
- `viewModelScope` — SupervisorJob + Main.immediate. Поэтому ошибка в одном `launch` не убивает ViewModel целиком.
- `supervisorScope { }` — builder, аналогичный `coroutineScope`, но с SupervisorJob внутри.

### Важный нюанс: SupervisorJob защищает только детей
Если супервизор — это **весь скоуп** (корень), упавшая корутина не роняет остальные. Но если SupervisorJob добавлен как **родитель внутри** иерархии, эффект тот же — только для уровня ниже.

### SupervisorJob не лечит structured concurrency
Ошибка `async` всё равно вылетит при `await()` — SupervisorJob не превращает `async` в терпимый к ошибкам. Для «отработать и забыть» используй `launch` + SupervisorJob, для результатов — обрабатывай ошибки у каждого `await`.

### Когда использовать
1. Scope уровня приложения/экрана (ViewModel, Presenter).
2. Несколько независимых фоновых задач (аналитика, логирование — ошибка одной не важна для других).

### Когда НЕ нужен
Внутри `coroutineScope { }` для последовательной композиции: если один из параллельных запросов упал, второй уже не нужен — пусть отменяется (это и есть structured concurrency, и она тут работает на нас).
