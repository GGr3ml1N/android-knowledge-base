### Структура тестируемого ViewModel (предпосылка)
VM должен быть спроектирован для тестов:
```kotlin
class FeedViewModel @Inject constructor(
    private val repository: NewsRepository,
    @DefaultDispatcher private val dispatcher: CoroutineDispatcher, // инжектим!
    savedStateHandle: SavedStateHandle
) : ViewModel() {

    private val _state = MutableStateFlow<State>(State.Idle)
    val state: StateFlow<State> = _state.asStateFlow()

    private val _events = MutableSharedFlow<Event>(extraBufferCapacity = 1)
    val events = _events.asSharedFlow()

    init { load() }

    fun load() {
        viewModelScope.launch {
            _state.value = State.Loading
            try {
                _state.value = State.Success(repository.getNews())
            } catch (e: CancellationException) {
                throw e
            } catch (e: Exception) {
                _state.value = State.Error(e.message)
            }
        }
    }

    override fun onCleared() {
        // viewModelScope отменяется автоматически — тест проверяем
    }
}
```

### Пример теста
```kotlin
@OptIn(ExperimentalCoroutinesApi::class)
class FeedViewModelTest {

    private val dispatcher = StandardTestDispatcher()
    private val repo = mockk<NewsRepository>()

    @get:Rule
    val mainRule = MainDispatcherRule(dispatcher)

    @Test
    fun `load emits Loading then Success`() = runTest(dispatcher) {
        val news = listOf(News("1", "a"))
        coEvery { repo.getNews() } returns news

        val vm = FeedViewModel(repo, dispatcher, SavedStateHandle())
        advanceUntilIdle()

        assertEquals(State.Success(news), vm.state.value)
        coVerify { repo.getNews() }
    }

    @Test
    fun `load emits Error when repository throws`() = runTest(dispatcher) {
        coEvery { repo.getNews() } throws IOException("no network")

        val vm = FeedViewModel(repo, dispatcher, SavedStateHandle())
        advanceUntilIdle()

        val state = vm.state.value
        assertTrue(state is State.Error)
        assertEquals("no network", (state as State.Error).message)
    }
}
```

### Чек-лист сценариев для VM
- [ ] Успешная загрузка (Loading -> Success)
- [ ] Сетевая ошибка (Loading -> Error)
- [ ] Неожиданная ошибка
- [ ] Повторный вызов load() (ретрай)
- [ ] Эмиссия события (SharedFlow/Channel) — через Turbine
- [ ] onCleared отменяет работу (проверить, что после onCleared состояние не меняется)

### Проверка отмены в onCleared
```kotlin
@Test
fun `state does not update after onCleared`() = runTest(dispatcher) {
    val vm = FeedViewModel(repo, dispatcher, SavedStateHandle())
    vm.onCleared() // или через ViewModelStore
    advanceUntilIdle()
    coVerify(exactly = 0) { repo.getNews() } // запрос отменён
}
```

### Зависимости, которые мокают в тестах VM
Репозиторий, юзкейсы, SavedStateHandle, аналитика, Clock/TimeProvider. **Не мокают:** StateFlow (это часть тестируемого), мапперы (реальные).

См. также: [[Coroutines test]], [[Turbine]]
