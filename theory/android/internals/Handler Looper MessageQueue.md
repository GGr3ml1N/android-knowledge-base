Триада, на которой работает главный поток Android.

### Message
Единица работы: `what` (int-код), `obj`/`data` (payload), `target` (Handler, который будет его обрабатывать), `when` (время исполнения).

### MessageQueue
Приоритетная очередь (по `when`) сообщений главного потока. `Looper` постоянно крутит цикл:
```java
for (;;) {
    Message msg = queue.next(); // блокируется, если очередь пуста
    msg.target.dispatchMessage(msg);
    msg.recycle();
}
```
`queue.next()` вызывает `nativePollOnce()` — epoll-ожидание: поток спит без нагрузки, пока нет сообщений (поэтому UI-поток «просыпается» на события).

### Handler
Точка отправки и обработки:
- `sendMessage()` / `post(Runnable)` — ставит сообщение в очередь (с задержкой — по `when` + `SystemClock.uptimeMillis()`).
- `dispatchMessage()` — выполняет Runnable или `handleMessage()`.

```kotlin
val handler = Handler(Looper.getMainLooper()) {
    // handleMessage
    true
}
handler.post { updateUi() }
handler.postDelayed({ hideTooltip() }, 3000)
```

### Почему нельзя создавать Handler без Looper
В не-UI потоке вызов `Handler()` бросит `RuntimeException: Can't create handler inside thread that has not called Looper.prepare()`. Либо подготавливаем цикл потока (`Looper.prepare()`/`Looper.loop()`), либо указываем чужой Looper (`Handler(Looper.getMainLooper())`), либо используем `HandlerThread` (поток с готовым Looper'ом и очередью).

### HandlerThread
Поток с Looper/MessageQueue: берём очередь заданий почти бесплатно.
```kotlin
val thread = HandlerThread("db").apply { start() }
val handler = Handler(thread.looper)
handler.post { dao.insert(item) }
```
Уходит в прошлое с корутинами (Dispatcher + Room suspend-функции), но вопрос «что такое HandlerThread» — классика.

### IdleHandler
`MessageQueue.addIdleHandler { ... }` — код, выполняющийся, когда очередь пуста. Используется библиотеками для отложенной инициализации (в духе «сделать после первой отрисовки»).

### Связь с корутинами
`Dispatchers.Main` — это scheduler поверх Handler/Looper: `dispatch` = `handler.post`. Отсюда следствия: корутина на Main не выполняется мгновенно; `post` vs `sendMessage` — очередь FIFO, но `post` на тот же тик может обогнать `sendMessage`, отправленный позже (нет гарантий строгого порядка для разных API).

### Стандартные вопросы
- «Что будет, если в postDelayed указать отрицательную задержку?» — выполнится немедленно (when = now).
- «Как устроен цикл сообщений?» — Looper.prepare (TLS, по одному Looper на поток), loop (бесконечный цикл next/dispatch), quit для остановки.
- «Почему нельзя трогать View из другого потока?» — View не thread-safe; проверка `checkThread()` в `ViewRootImpl` бросает `CalledFromWrongThreadException` (UI-фреймворк спроектирован single-threaded, без синхронизации для скорости).
