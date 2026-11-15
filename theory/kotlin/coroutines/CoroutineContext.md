`CoroutineContext` — это множество элементов, определяющих, **как** и **где** выполняется корутина. Аналогия: это «контекст окружения» корутины.

### Составляющие контекста (ключевые)
```kotlin
CoroutineContext =
    Job                    // жизненный цикл и отмена
    + CoroutineDispatcher  // где выполняется (поток)
    + CoroutineName        // имя для отладки
    + CoroutineExceptionHandler // глобальная обработка необработанных ошибок
    + CoroutineId          // уникальный идентификатор (отладка)
```

### Операции над контекстом
```kotlin
val ctx1: CoroutineContext = Dispatchers.IO + CoroutineName("network")

// "+" переопределяет элемент с тем же ключом
val ctx2 = ctx1 + Dispatchers.Default // теперь диспетчер Default, имя сохранилось
```

### Откуда берётся контекст
`coroutineContext` — наследуемый элемент: дочерняя корутина получает контекст родителя (job заменяется на дочерний, остальное копируется).
```kotlin
launch(Dispatchers.Main + CoroutineName("parent")) {
    launch { 
        // унаследует Main + имя "parent", но Job уже свой, дочерний
        println(coroutineContext[CoroutineName]) // "parent"
    }
}
```

### Зачем знать на собесе
- **Стандартный вопрос:** «что такое CoroutineContext и чем отличается от CoroutineScope» — Scope = Context + жизненный цикл (Job); scope владеет корутинами, контекст описывает их исполнение.
- `SupervisorJob` передаётся именно через контекст: `CoroutineScope(SupervisorJob() + Dispatchers.Main)`.
- Своими элементами контекста реализуются `viewModelScope` (SupervisorJob + Main.immediate + job на onCleared).
