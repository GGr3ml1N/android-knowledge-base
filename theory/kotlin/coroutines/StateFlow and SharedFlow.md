Оба — **горячие** реализации Flow: хранят значение/эмиссии независимо от коллекторов.

### StateFlow — состояние экрана
Хранит **одно текущее значение**, всегда есть значение, дедуплицирует (`equals`).
```kotlin
private val _state = MutableStateFlow<ScreenState>(ScreenState.Loading)
val state: StateFlow<ScreenState> = _state.asStateFlow()

fun load() {
    viewModelScope.launch {
        _state.value = ScreenState.Loading
        try {
            _state.value = ScreenState.Success(repository.getData())
        } catch (e: CancellationException) {
            throw e
        } catch (e: Exception) {
            _state.value = ScreenState.Error(e)
        }
    }
}
```
Сбор в UI: `collect`/`collectAsStateWithLifecycle()` — получите текущее состояние сразу, а потом обновления.

### SharedFlow — события
Хранит **буфер последних эмиссий**, нет «текущего» значения. Идеален для одноразовых событий (навигация, тосты), но нужен буфер, чтобы событие не потерялось между подпиской и сбором.
```kotlin
private val _events = MutableSharedFlow<Event>(extraBufferCapacity = 1)
val events: SharedFlow<Event> = _events.asSharedFlow()

fun onTokenExpired() {
    _events.tryEmit(Event.NavigateToLogin) // tryEmit не приостанавливает
}
```
Классика-ошибка: событие навигации через StateFlow — при перевороте экрана событие «пере-соберётся» и произойдёт повторная навигация. Решение: SharedFlow с буфером, или Channel в ViewModel + `receiveAsFlow()` в UI.

### Таблица: что выбрать
| Нужно | Выбор |
| :--- | :--- |
| Состояние экрана (Loading/Data/Error) | StateFlow |
| Одноразовые события (toast, navigate) | SharedFlow (extraBufferCapacity=1) или Channel |
| Поток данных с трансформациями | cold Flow |
| Коммуникация корутин | Channel |

### collectAsStateWithLifecycle
В Compose всегда собирать через `collectAsStateWithLifecycle()` — при уходе экрана в бэкстек сбор останавливается, `WhileSubscribed` гаснет upstream, экономим батарею и запросы.

### Параметры SharedFlow
- `replay` — сколько последних значений получит новый подписчик (для StateFlow = 1).
- `extraBufferCapacity` — буфер сверх replay; при переполнении (без `onBufferOverflow = DROP_OLDEST`) `emit` приостанавливается или `tryEmit` возвращает false.
- `onBufferOverflow` — SUSPEND (по умолчанию) / DROP_OLDEST / DROP_LATEST.

### Сравнение с LiveData
| LiveData | StateFlow |
| :--- | :--- |
| Android-специфичен | Multiplatform |
| Автоматически lifecycle-aware в XML | Нужен `collectAsStateWithLifecycle`/repeatOnLifecycle |
| Нет `combine` из коробки | Полноценные Flow-операторы |
| Main-thread ограничения исторически | Любой диспетчер |

### Горячий vs холодный — стандартный вопрос
- Cold (Flow): выполняется при collect, каждый коллектор — отдельный запуск.
- Hot (StateFlow/SharedFlow): работает независимо, новые подписчики получают текущее/replayed значения.
