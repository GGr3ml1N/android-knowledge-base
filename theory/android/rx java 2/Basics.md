### 1. Пять базовых классов (Reactive Types)
В RxJava 2 выбор типа зависит от количества эмиссий и необходимости Backpressure (противодавления).

1. **`Observable<T>`**
   * **Эмиссия:** 0..N элементов.
   * **Backpressure:** Нет.
   * **Где использовать:** UI-события (клики, скроллы), быстрые потоки данных, где мы не можем замедлить источник.
2. **`Flowable<T>`**
   * **Эмиссия:** 0..N элементов.
   * **Backpressure:** Есть.
   * **Где использовать:** Чтение файлов, стриминг с БД, парсинг больших JSON.
3. **`Single<T>`**
   * **Эмиссия:** Ровно 1 элемент или `onError`.
   * **Где использовать:** Сетевые запросы (Retrofit), запросы к БД, которые всегда возвращают результат.
4. **`Maybe<T>`**
   * **Эмиссия:** 0 или 1 элемент, либо `onError`, либо `onComplete`.
   * **Где использовать:** Запросы к БД или кэшу, где результат может отсутствовать (null).
5. **`Completable`**
   * **Эмиссия:** Только `onComplete` или `onError` (нет данных).
   * **Где использовать:** Запись в БД, сохранение в `SharedPreferences`, void-методы в сети.

---

### 2. Базовые операторы
В Kotlin мы используем лямбды, что делает код максимально лаконичным.

#### Создание (Creation)
```kotlin
// Эмитит конкретные значения
Observable.just(1, 2, 3) 

// Эмитит элементы коллекции
Observable.fromIterable(listOf("A", "B", "C"))

// Ленивое создание (вычислится только при подписке)
Single.fromCallable { apiService.getUser() }

// Отложенное создание (создает новый Observable для каждого подписчика)
Observable.defer { Observable.just(System.currentTimeMillis()) }
```

#### Трансформация (Transformation)
```kotlin
// 1 к 1. Меняем тип или значение
.map { user -> user.name }

// 1 к N. "Сплющивает" вложенные Observable. Порядок не гарантируется.
.flatMap { id -> apiService.getUserDetails(id) }

// Аналог flatMap, но строго сохраняет порядок и ждет завершения предыдущего.
.concatMap { id -> db.getUser(id) }

// Отменяет предыдущую внутреннюю подписку, если пришла новая (идеально для поиска).
.switchMap { query -> searchApi.search(query) }
```

#### Фильтрация (Filtering)
```kotlin
// Пропускаем только четные
.filter { it % 2 == 0 }

// Берем только первые N элементов
.take(5)

// Пропускаем дубликаты, идущие подряд (Must have для UI, чтобы не перерисовывать список)
.distinctUntilChanged()
```

#### Комбинирование (Combining)
```kotlin
// Объединяет потоки, эмитит результат, когда ОКАЗЫВАТ оба потока завершатся (или эмитят)
Single.zip(singleA, singleB) { a, b -> a + b }

// Эмитит каждый раз, когда ЛЮБОЙ из потоков испустил значение
Observable.combineLatest(textObservable, checkboxObservable) { text, isChecked -> 
    text.isNotEmpty() && isChecked 
}

// Сливает несколько потоков в один (чередуя элементы)
Observable.merge(observable1, observable2)
```

#### Обработка ошибок (Error Handling)
```kotlin
// Возвращаем дефолтное значение при ошибке
.onErrorReturn { "Default" }

// Переключаемся на другой источник при ошибке
.onErrorResumeNext { fallbackObservable }

// Повторная подписка при ошибке (с задержкой)
.retry(3) { attempt, error -> 
    if (error is IOException) true else false 
}
```

#### Side Effects (Побочные эффекты)
Используются для логирования или отладки, не меняют поток.
```kotlin
.doOnNext { Log.d("TAG", "Next: $it") }
.doOnError { Log.e("TAG", "Error", it) }
.doOnSubscribe { Log.d("TAG", "Subscribed") }
.doOnDispose { Log.d("TAG", "Disposed") }
```

---

### 3. Потоки (Schedulers)
Классическая схема для Android: тяжелая работа в фоне, результат в UI.

```kotlin
apiService.getData()
    .subscribeOn(Schedulers.io())           // Где выполняем (подписку и работу)
    .observeOn(AndroidSchedulers.mainThread()) // Где получаем результат (UI)
    .subscribe(...)
```
* `Schedulers.io()`: Сеть, БД, файлы. (Пул потоков).
* `Schedulers.computation()`: Математика, фильтрация больших списков. (Фиксированный пул).
* `AndroidSchedulers.mainThread()`: UI поток.

---

### 4. Управление памятью (Disposables)
В RxJava 2 `Subscription` стала `Disposable`. Это критично для Android, чтобы избежать утечек памяти.

**Плохо (нарушает DRY, легко забыть):**
```kotlin
val disposable = observable.subscribe()
// ... где-то в onDestroy
disposable.dispose()
```

**Хорошо (Используем CompositeDisposable):**
```kotlin
class MyViewModel : ViewModel() {
    private val disposables = CompositeDisposable()

    fun loadData() {
        val disposable = repository.getData()
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(
                { data -> _state.value = State.Success(data) },
                { error -> _state.value = State.Error(error) }
            )
        disposables.add(disposable)
    }

    override fun onCleared() {
        super.onCleared()
        disposables.clear() // или dispose()
    }
}
```
---