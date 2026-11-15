Structured Concurrency (структурированная конкурентность) — принцип: **корутины не живут дольше, чем scope, их породивший**. Это гарантия, что `viewModelScope.launch { }` не утечёт после `onCleared`.

### Три гарантии
1. **Отмена распространяется вниз:** `scope.cancel()` отменяет все дочерние корутины (рекурсивно).
2. **Ошибка распространяется вверх:** необработанная ошибка ребёнка роняет родителя и сиблингов (см. [[SupervisorJob]]).
3. **Родитель ждёт детей:** `coroutineScope { }`/`supervisorScope { }` не возвращаются, пока не завершатся все дочерние корутины.

### Практический смысл
Не нужно вручную следить за Job'ами — достаточно держать правильный scope:
```kotlin
class FeedViewModel : ViewModel() {
    init {
        viewModelScope.launch { loadFeed() }   // отменится в onCleared
        viewModelScope.launch { loadStories() } // отменится тоже
    }
}
```

### Антипаттерн: GlobalScope
```kotlin
// ПЛОХО: корутина переживёт Activity/ViewModel, утечёт память, 
// крашнет при обращении к мёртвому контексту
GlobalScope.launch { repository.sync() }
```
Исключение — задачи уровня приложения (синхронизация при выходе), и то лучше свой scope с SupervisorJob в Application-классе.

### Антипаттерн: scope.launch из suspend без structured scope
```kotlin
// ПЛОХО: fun не suspend, запустила фоновую работу и "забыла" про неё
fun refresh() {
    GlobalScope.launch { repository.refresh() }
}

// ХОРОШО: suspend fun + coroutineScope — вызывающий ждёт завершения
suspend fun refresh(): Unit = coroutineScope {
    launch { repository.refresh() }
}
```

### Композиция suspend-функций
`coroutineScope` позволяет внутри одной suspend-функции запустить несколько параллельных веток и собрать результаты:
```kotlin
suspend fun loadDashboard() = coroutineScope {
    val profile = async { api.getProfile() }
    val balance = async { api.getBalance() }
    val promos  = async { api.getPromos() }
    Dashboard(profile.await(), balance.await(), promos.await())
}
// Если profile упал — balance и promos отменятся автоматически. 
// Это фича: освобождаем ресурсы раньше.
```

### Стандартный вопрос
«Чем `coroutineScope` отличается от `supervisorScope`?» — поведением при ошибке ребёнка: coroutineScope роняет всех, supervisorScope изолирует (см. [[SupervisorJob]]).
