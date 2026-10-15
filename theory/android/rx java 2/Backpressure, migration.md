### Часть 1. RxJava 2: Backpressure (Противодавление)

**Проблема:** Источник данных (Producer) генерирует события быстрее, чем потребитель (Consumer) может их обработать. Буфер переполняется, и RxJava 2 бросает `MissingBackpressureException`.

**Решение:** В RxJava 2 для этого введен `Flowable`.

#### Стратегии Backpressure
Когда ты создаешь `Flowable`, ты должен указать стратегию через `onBackpressureXXX()`:

```kotlin
// 1. BUFFER (По умолчанию для Flowable)
// Копит все в бесконечную очередь. Риск: OutOfMemoryError.
Flowable.create({ emitter -> ... }, BackpressureStrategy.BUFFER)

// 2. DROP
// Если буфер полон, новые события просто выбрасываются. 
// Идеально для сенсоров, кликов, где важна только актуальность.
.onBackpressureDrop()

// 3. LATEST
// Хранит только самое последнее событие. Если потребитель медленный, 
// он просто пропустит промежуточные и возьмет последнее.
.onBackpressureLatest()
```
---

### Часть 2. Schedulers: Глубокое погружение и подводные камни

Главное правило, которое спрашивают на собеса и которое часто путают джуны:
* **`subscribeOn()`** — указывает, в каком потоке будет выполняться **источник** (подписка и генерация данных). *Вызывается один раз, повторные вызовы игнорируются.*
* **`observeOn()`** — указывает, в каком потоке будут выполняться **операторы ниже по цепочке** (и подписчик). *Можно вызывать много раз, переключая потоки на лету.*

#### Подводные камни (Error Handling & Performance)

```kotlin
api.getUser()
    .subscribeOn(Schedulers.io()) // 1. Сеть в IO
    .map { user -> 
        // 2. ОШИБКА НОВИЧКА: Тяжелая операция (парсинг, БД) в том же потоке, 
        // что и сеть. Если нужно в другой поток - меняем observeOn.
        heavyImageProcessing(user) 
    }
    .observeOn(AndroidSchedulers.mainThread()) // 3. Результат в UI
    .subscribe { ... }
```
---

### Часть 3. Миграция на Kotlin Coroutines

Coroutines — это не просто "замена Rx". Это **структурная конкурентность**. Они решают главные боли Rx: утечки памяти (через `CoroutineScope`), сложный Backpressure и огромный бойлерплейт.

#### Словарь миграции (Rx -> Coroutines)

| RxJava 2 | Kotlin Coroutines | Примечание |
| :--- | :--- | :--- |
| `Observable<T>` | `Flow<T>` | Холодный поток. Эмитит 0..N значений. |
| `Single<T>` | `suspend fun` (возвращает `T`) | Просто функция, ждущая результата. |
| `Maybe<T>` | `suspend fun` (возвращает `T?`) | Nullable тип в Kotlin заменяет Maybe. |
| `Completable` | `suspend fun` (возвращает `Unit`) | Просто suspend-функция без return. |
| `Subject` (Behavior) | `StateFlow<T>` | Хранит состояние, горячий поток. |
| `Subject` (Publish) | `SharedFlow<T>` | Горячий поток событий (аналог EventBus). |
| `Disposable` | `Job` / `CoroutineScope` | Автоматическая отмена через Scope. |
| `Schedulers.io()` | `Dispatchers.IO` | Пул потоков для сети/БД. |
| `AndroidSchedulers.main()`| `Dispatchers.Main` | UI поток. |

#### Пример миграции: Сетевой запрос

**Было (RxJava 2):**
```kotlin
private val disposables = CompositeDisposable()

fun loadUser() {
    val disposable = api.getUser()
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(
            { user -> view.showUser(user) },
            { error -> view.showError(error) }
        )
    disposables.add(disposable)
}

override fun onCleared() {
    disposables.clear() // Ручное управление памятью
}
```

**Стало (Coroutines):**
```kotlin
fun loadUser() {
    // viewModelScope автоматически отменит запрос при onDestroy
    viewModelScope.launch {
        try {
            // withContext переключает поток, не блокируя Main
            val user = withContext(Dispatchers.IO) { 
                api.getUser() 
            }
            view.showUser(user)
        } catch (e: Exception) {
            view.showError(e)
        }
    }
}
// onCleared() больше не нужен!
```

#### Пример миграции: UI-события (Поиск)

**Было (RxJava 2):**
```kotlin
disposables.add(
    RxTextView.textChangeEvents(searchEditText)
        .debounce(300, TimeUnit.MILLISECONDS)
        .distinctUntilChanged()
        .switchMap { query -> api.search(query.toString()) }
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe { results -> adapter.update(results) }
)
```

**Стало (Coroutines Flow):**
```kotlin
// 1. Создаем Flow из UI (в Fragment/Activity)
val searchQuery = searchEditText.textChanges() // Extension-функция

// 2. Собираем в Scope (ViewModel или LifecycleOwner)
viewLifecycleOwner.lifecycleScope.launch {
    searchQuery
        .debounce(300)
        .distinctUntilChanged()
        .flatMapLatest { query -> 
            // flatMapLatest аналог switchMap. Отменяет старый запрос, если пришел новый
            flow { emit(api.search(query)) } 
        }
        .flowOn(Dispatchers.IO) // Аналог subscribeOn/observeOn
        .collect { results -> 
            adapter.update(results) 
        }
}
```

---

### Часть 4. Как мигрировать постепенно (Interoperability)

Никто не переписывает проект за неделю. Придется миксовать Rx и Coroutines. Для этого есть библиотека **`kotlinx-coroutines-rx2`**.

#### 1. Rx -> Coroutines
Если UseCase возвращает `Single`, а ты пишешь новый код на Coroutines:
```kotlin
// Single.await() приостанавливает корутину, но не блокирует поток
viewModelScope.launch {
    try {
        val user = rxRepository.getUser().await()
        _state.value = State.Success(user)
    } catch (e: Throwable) { ... }
}
```

Если UseCase возвращает `Observable`, конвертируем в `Flow`:
```kotlin
val flow: Flow<User> = rxRepository.observeUsers().asFlow()
```

#### 2. Coroutines -> Rx
Если легаси-View подписывается на `Observable`, а ты переписал Repository на `Flow`:
```kotlin
// Flow.asObservable()
val observable: Observable<User> = repository.observeUsers().asObservable()
```

---

### Часть 5. Архитектурные советы

1. **Границы слоев (Clean Architecture):**
   * **Data/Domain слой:** Мигрируй на Coroutines в первую очередь. UseCase должен возвращать `suspend fun` или `Flow`.
   * **Presentation слой (ViewModel):** Используй `StateFlow` для UI-стейта и `SharedFlow` для одноразовых событий (навигация, тосты).
   * **Legacy View:** Если Activity/Fragment все еще на Rx, используй `.asObservable()` на границе. Но лучше перевести View на `collectLatest` в `lifecycleScope`.

2. **StateFlow vs BehaviorSubject:**
   Забудь про `BehaviorSubject` в новых фичах. `StateFlow` всегда имеет начальное значение (нет состояния "пусто"), он thread-safe и интегрирован в Jetpack Lifecycle (`repeatOnLifecycle`).

3. **Обработка ошибок в Flow:**
   В Rx мы использовали `onErrorResumeNext`. В Flow используй `catch`:
   ```kotlin
   repository.getData()
       .catch { e -> 
           // Логирование, эмим ошибку как часть UI-стейта
           emit(Resource.Error(e)) 
       }
       .collect { ... }
   ```