`Turbine` — библиотека cashapp для тестирования Flow (StateFlow/SharedFlow/cold Flow). Решает главную проблему: обычные терминальные операторы (`first`, `toList`) не позволяют проверить **последовательность** эмиссий и рискуют зависнуть на бесконечном потоке.

### Базовый пример
```kotlin
@Test
fun `state goes Loading then Success`() = runTest {
    val vm = FeedViewModel(fakeRepo)

    vm.state.test {
        assertEquals(State.Loading, awaitItem())   // первая эмиссия
        vm.refresh()                               // триггерим логику
        assertEquals(State.Success(data), awaitItem())
        cancelAndIgnoreRemainingEvents()
    }
}
```
`test { }` создаёт коллектора, `awaitItem()` приостанавливает тест до следующей эмиссии и возвращает её.

### Проверка отсутствия событий
```kotlin
events.test {
    vm.onSomethingHarmless()
    expectNoEvents() // ни тоста, ни навигации
    cancelAndConsumeRemainingEvents()
}
```

### Работа с таймаутами
```kotlin
awaitItem()              // ждёт эмиссию (в runTest — виртуальное время)
awaitComplete()          // поток завершился
awaitError()             // поток упал с ошибкой
```

### Тестирование SharedFlow-событий
```kotlin
@Test
fun `token expired navigates to login`() = runTest {
    val events = MutableSharedFlow<Event>(extraBufferCapacity = 1)
    val collector = launch { /* UI-коллектор */ }

    events.test {
        emitViaViewModelLogic()
        assertEquals(Event.NavigateToLogin, awaitItem())
    }
}
```

### Без Turbine (простые случаи)
```kotlin
assertEquals(expected, flow.first())          // одно значение
assertEquals(listOf(1, 2, 3), flow.take(3).toList())
```

### Частые ошибки
- Забыли `cancelAndIgnoreRemainingEvents()`/`cancelAndConsumeRemainingEvents()` — оставшийся коллектор удерживает scope, `runTest` падает с утечкой.
- `awaitItem()` без обновления состояния — тест виснет до таймаута. Это нормально: значит логика не эмитит, ищите баг.
- Тест StateFlow через многократный `value` — пропускает промежуточные состояния (Loading), проверяет только финальное. Turbine ловит всю последовательность.

См. также: [[Coroutines test]]
