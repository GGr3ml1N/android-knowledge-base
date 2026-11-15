`Channel` — это **очередь для передачи данных между корутинами** (аналог BlockingQueue, но suspend). Hot источник: существует независимо от подписчиков.

### Базовый пример: producer/consumer
```kotlin
val channel = Channel<Int>()

// Producer
launch {
    for (x in 1..5) channel.send(x * x)
    channel.close() // сигнал конца
}

// Consumer
launch {
    for (y in channel) println(y) // печатает 1, 4, 9, 16, 25
}
```

### Виды каналов
| Тип | Буфер | Поведение send |
| :--- | :--- | :--- |
| `Channel.RENDEZVOUS` (по умолчанию) | 0 | Блокируется, пока нет получателя |
| `Channel.BUFFERED` | 64 (по умолчанию) | При переполнении — приостанавливается |
| `Channel.CONFLATED` | 1 | Новый элемент заменяет старый, send не ждёт |
| `Channel.UNLIMITED` | ∞ | Никогда не ждёт (осторожно с памятью) |

### Producer pattern (обёртка)
```kotlin
fun CoroutineScope.squares(): ReceiveChannel<Int> = produce {
    for (x in 1..5) send(x * x)
}

launch {
    squares().consumeEach { println(it) }
}
```

### fan-out и fan-in
Несколько корутин могут получать из одного канала (каждое сообщение достаётся ровно одному). Несколько producer'ов могут отправлять в один канал.

### Отмена
`channel.cancel()` закрывает канал с CancellationException. Producer/consumer останавливаются кооперативно.

### Channel vs Flow — что ответить на собесе
| Channel | Flow |
| :--- | :--- |
| Hot (живёт сам по себе) | Cold (запускается при сборе) |
| Служит для **коммуникации** корутин | Служит для **потоков данных** |
| Может иметь несколько независимых получателей | Коллекторы независимы, каждый запускает поток заново |
| send/receive — ручное управление | Операторы, декларативно |

**Практика:** почти всё, что раньше делали через Channel, в Android лучше делать через Flow/StateFlow. Каналы остаются для низкоуровневых сценариев: актёр-модель, передача событий между строго ограниченными корутинами, callback-адаптеры (`callbackFlow` внутри использует канал).

### callbackFlow — мост callback API -> Flow
Стандартный способ обернуть callback-based API (LocationManager, старые SDK):
```kotlin
fun locationFlow(context: Context): Flow<Location> = callbackFlow {
    val manager = context.getSystemService(LocationManager::class.java)
    val listener = object : LocationListener {
        override fun onLocationChanged(loc: Location) {
            trySend(loc)
        }
    }
    manager.requestLocationUpdates(GPS, 0, 0f, listener)
    awaitClose { manager.removeUpdates(listener) } // обязательно! очистка при отмене сбора
}
```
`awaitClose` — критичен: без него поток не отреагирует на отмену и утечёт listener.
