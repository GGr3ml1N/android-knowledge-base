`Flow<T>` — это **холодный асинхронный поток данных** (аналог Observable/Flowable в Rx, но на корутинах). Cold = код внутри выполняется заново для каждого коллектора.

```kotlin
val flow = flow {
    emit(1)     // emit вместо onNext
    delay(100)
    emit(2)
}
// Ничего не происходит, пока нет терминального оператора:
flow.collect { println(it) }
```

### Правило: Flow — не для горячих событий
Одноразовые события (тост, навигация) через обычный Flow легко теряются при перевороте экрана. Для этого — SharedFlow/Channel (см. [[StateFlow and SharedFlow]]).

### Создание Flow
```kotlin
flow { emit(api.getData()) }                        // универсальный
flowOf(1, 2, 3)                                     // из значений
listOf(1, 2, 3).asFlow()                            // из коллекции
flow { }.flowOn(Dispatchers.IO)                     // смена диспетчера
```

### Ключевые операторы
```kotlin
flow
    .map { it.name }                        // трансформация 1->1
    .filter { it.isNotEmpty() }
    .flatMapConcat { id -> repo.details(id) }  // последовательно, сохраняет порядок
    .flatMapMerge { id -> repo.details(id) }   // параллельно, порядок не гарантирован
    .flatMapLatest { q -> search(q) }       // отменяет предыдущий при новом запросе (как switchMap в Rx!)
    .combine(flow2) { a, b -> a to b }      // эмитит при обновлении ЛЮБОГО источника
    .zip(flow2) { a, b -> a to b }          // ждёт оба
    .catch { e -> emit(fallback) }          // обработка ошибок в цепочке
    .flowOn(Dispatchers.IO)                 // ВСЁ выше — на IO
    .collect { render(it) }                 // терминальный оператор
```

### catch vs try/catch
- `.catch { }` ловит ошибки **выше по цепочке** (вверх по flow). Полезно в репозитории: один catch на всю цепочку трансформаций.
- try/catch вокруг `collect` — ловит всё, включая ошибки в самом коллекторе.
```kotlin
repository.search(query)   // ошибка здесь
    .catch { emit(emptyList()) }
    .collect { render(it) } // ошибка здесь — catch выше её не поймает
```

### stateIn / shareIn — превращение cold в hot
```kotlin
// Кешируем результат: один запрос на всех коллекторов
val news: StateFlow<List<News>> = repository.getNews()
    .stateIn(
        scope = viewModelScope,
        started = SharingStarted.WhileSubscribed(5000), // слушатели есть -> активен; пауза 5с на отписку
        initialValue = emptyList()
    )
```

### Backpressure
У Flow нет явного backpressure-протокола: если emit быстрее, чем collect, по умолчанию корутина-производитель просто приостанавливается (`SuspendingEmitter`). Варианты:
- `.buffer()` — производитель не ждёт медленного коллектора (буферизация).
- `.conflate()` — коллектор получает только последнее значение (пропускает промежуточные). Идеально для прогресса.
- `.collectLatest { }` — отменяет обработку при новом значении.

### Многократный сбор
Один Flow можно собирать в нескольких местах — каждый запуск независим (cold!). Если не нужно дублировать сетевые запросы — `shareIn`/`stateIn` (кеш в scope).

### Suspend-функции внутри операторов
Всё внутри `flow { }`/`map { }` и т.д. — suspend, значит можно вызывать другие suspend-функции без `withContext`, если диспетчер уже правильный (`flowOn`).
