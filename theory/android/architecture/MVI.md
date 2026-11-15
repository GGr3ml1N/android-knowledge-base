**Model-View-Intent** — паттерн с **строго однонаправленным потоком данных (UDF)**. Не путать Intent в MVI с android.content.Intent: здесь Intent = «намерение пользователя» (действие/событие).

### Цикл
```
User action --> Intent (Action) --> Reducer (State, Action) --> new State --> View
                                      |
                                 Side effects (эффекты: сеть, навигация)
```

### Ключевые свойства
1. **Единое состояние экрана** — один data class `ViewState`, описывающий экран целиком: `data class FeedState(val isLoading: Boolean, val items: List<Item>, val error: String?)`. Экран = функция состояния.
2. **Состояние иммутабельно** — новое состояние создаётся копированием (`state.copy(...)`), никогда не мутируется.
3. **Reducer-логика** — чистая функция `(State, Action) -> State` (+ эффекты). Чистая = тестируется тривиально.
4. **Все события пользователя** — экземпляры sealed class `Action`.

### Реализация на корутинах (твоя концепция [[Elementary State Manager]] — это MVI)
```kotlin
data class FeedState(val loading: Boolean = false, val items: List<News> = emptyList(), val error: String? = null)

sealed interface FeedAction {
    data object Refresh : FeedAction
    data class ItemClicked(val id: String) : FeedAction
}

class FeedViewModel(private val repo: NewsRepository) : ViewModel() {
    private val _state = MutableStateFlow(FeedState())
    val state = _state.asStateFlow()

    private val _effects = MutableSharedFlow<FeedEffect>(extraBufferCapacity = 1)
    val effects = _effects.asSharedFlow()

    fun dispatch(action: FeedAction) {
        when (action) {
            FeedAction.Refresh -> refresh()
            is FeedAction.ItemClicked -> _effects.tryEmit(FeedEffect.NavigateToDetail(action.id))
        }
    }

    private fun refresh() {
        viewModelScope.launch {
            _state.update { it.copy(loading = true, error = null) }
            try {
                val items = repo.getNews()
                _state.update { it.copy(loading = false, items = items) }
            } catch (e: CancellationException) {
                throw e
            } catch (e: Exception) {
                _state.update { it.copy(loading = false, error = e.message) }
            }
        }
    }
}
```

### MVI vs MVVM
| | MVVM | MVI |
| :--- | :--- | :--- |
| Состояние | Несколько LiveData/Flow, синхронизация вручную | Один иммутабельный State |
| События | Публичные методы VM | Sealed Action, dispatch() |
| Сайд-эффекты | Где придётся (легко размазать) | Явные: Effect-канал |
| Тестируемость | Хорошая | Отличная: reducer — чистая функция |
| Бойлерплейт | Меньше | Больше (Action/State/Effect на экран) |

### Когда что выбирать (для собеса)
- Экран простой, команда маленькая → MVVM, не усложнять.
- Сложные экраны со множеством состояний/событий, требования к предсказуемости и тестам → MVI (или «MVI-lite»: один StateFlow + sealed Actions без полноценного редьюсера).
- Важно: на практике это спектр, а не религия. Умение объяснить, где в проекте что и почему — это и есть сеньорность.

### Подводные камни MVI
- Бойлерплейт: на каждый экран Action/State/Effect.
- `data class` со списками: `copy(items = newList)` — если список большой и меняется часто, в Compose могут быть лишние рекомпозиции (см. stability в [[!Jetpack Compose]]).
- Один гигантский State на весь экран: при 30+ полях дробят на под-состояния (паттерн подсостояний, ESM-подход).
