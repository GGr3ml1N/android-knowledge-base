Тестирование корутин — обязательный блок, если в вакансии есть unit tests.

### Базовые инструменты
`kotlinx-coroutines-test`: `runTest`, `StandardTestDispatcher`, `UnconfinedTestDispatcher`, `advanceUntilIdle`, `advanceTimeBy`.

### runTest вместо runBlocking
`runTest` — main dispatcher в тестах подменяется виртуальным временем: `delay()` исполняется мгновенно.
```kotlin
@Test
fun `refresh updates state`() = runTest {
    val vm = FeedViewModel(fakeRepo)
    vm.refresh()
    advanceUntilIdle() // выполнить всё отложенное
    assertEquals(State.Success(data), vm.state.value)
}
```

### TestDispatcher
Чтобы контролировать диспетчеры, которые использует ViewModel:
```kotlin
@OptIn(ExperimentalCoroutinesApi::class)
class FeedViewModelTest {
    private val dispatcher = StandardTestDispatcher()

    @get:Rule
    val mainRule = MainDispatcherRule(dispatcher) // подменяет Dispatchers.Main

    @Test
    fun test() = runTest(dispatcher) {
        // код VM теперь на тестовом диспетчере
    }
}
```
`MainDispatcherRule` (свой JUnit Rule):
```kotlin
class MainDispatcherRule(val dispatcher: TestDispatcher = StandardTestDispatcher()) :
    TestWatcher() {
    override fun starting(description: Description) {
        Dispatchers.setMain(dispatcher)
    }
    override fun finished(description: Description) {
        Dispatchers.resetMain()
    }
}
```

### StandardTestDispatcher vs UnconfinedTestDispatcher
- **Standard**: корутины запускаются только когда вызываешь `advanceUntilIdle()`/`runCurrent()`. Детерминизм — знаешь, что выполняется.
- **Unconfined**: стартует корутину немедленно до первой приостановки. Быстрее писать, но порядок менее явный.

### Частые грабли
1. **Тест зависает** — где-то в коде `Dispatchers.Main`/`Default`/`IO` захардкожен, а не инжектится. Правило: диспетчеры инжектить в конструктор (удобно с Dagger/Hilt).
2. **Assertion до завершения** — забыли `advanceUntilIdle()` после вызова метода VM.
3. **`runTest` ловит leaked jobs** — если scope не отменяется в `onCleared`, тест упадёт с «A child of test scope was not cancelled» — это фича, показывающая утечку.

### Тестирование Flow с Turbine
```kotlin
vm.state.test {
    assertEquals(State.Loading, awaitItem())
    vm.refresh()
    assertEquals(State.Success(data), awaitItem())
    cancelAndIgnoreRemainingEvents()
}
```
Без Turbine: `first()`, `toList()`, `take(3).toList()` — для простых случаев достаточно.

### Проверка отмены
```kotlin
@Test
fun `cancelling scope cancels request`() = runTest {
    val job = launch { vm.refresh() }
    job.cancel()
    assertTrue(fakeRepo.wasCancelled)
}
```
См. также [[Turbine]] в разделе тестирования.
