### Что такое Job
`Job` — это **дескриптор жизненного цикла** корутины. Состояния: `New -> Active -> Completing -> Completed` или `Cancelling -> Cancelled`.
```kotlin
val job = scope.launch { ... }
job.cancel()        // отмена
job.join()          // приостановиться до завершения
job.cancelAndJoin() // отменить и дождаться
job.isActive        // ещё работает?
job.isCancelled     // была отменена?
```

### Отмена — кооперативная
`cancel()` не убивает корутину мгновенно, а ставит **флаг отмены**. Корутина завершится только в точке приостановки (`suspend point`), где она проверит флаг:
- `delay()`, `withContext()`, `await()`, сбор Flow — всё это cancellable points.
```kotlin
val job = launch {
    repeat(1000) { i ->
        println(i)
        delay(500) // здесь корутина проверит флаг и выбросит CancellationException
    }
}
delay(1300)
job.cancel() // выведет ~2-3 итерации и остановится
```

### CancellationException
При отмене внутри корутины бросается `kotlinx.coroutines.CancellationException` (наследник `IllegalStateException`). **Его нельзя глотать:**
```kotlin
// ПЛОХО: исключение перехвачено, корутина "воскреснет" и доработает
try {
    doSomething()
} catch (e: Exception) { }

// ХОРОШО: пробрасываем CancellationException
try {
    doSomething()
} catch (e: CancellationException) {
    throw e
} catch (e: Exception) {
    // обрабатываем реальные ошибки
}
```

### Необрабатываемая CPU-работа
Вычисления без suspend-точек отмену не проверяют:
```kotlin
launch {
    var next = System.currentTimeMillis()
    var i = 0
    while (i < 1_000_000) {
        // check for cancellation в горячем цикле
        ensureActive() // или yield()
        i++
    }
}
```

### Таймаут
```kotlin
val result = withTimeoutOrNull(3000) {
    api.getData() // через 3 сек вернёт null
}
```

### Корутина не останавливается сама
Если забыть отменить Job — корутина продолжит работу и удержит ресурсы. Поэтому всегда используем скоупы с жизненным циклом: `viewModelScope` (отменяется в `onCleared`), `lifecycleScope` (отменяется при destroy).
