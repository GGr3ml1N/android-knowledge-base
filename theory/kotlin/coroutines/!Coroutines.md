`Coroutine` (сопрограмма) — это лёгкая потоко-независимая единица исполнения. В отличие от потока, корутина не привязана к конкретному потоку: она может стартовать на одном потоке, приостановиться (suspend) и продолжиться на другом.

Ключевые отличия от потоков:
- Дёшевы в создании: сотни тысяч корутин против тысяч потоков (поток — объект ОС, ~1 МБ стека).
- Приостановка (suspend) не блокирует поток — поток освобождается для другой работы.

Корутина всегда запускается внутри **CoroutineScope**. Scope = CoroutineContext + жизненный цикл. Контекст содержит: Job, Dispatcher, CoroutineName, CoroutineExceptionHandler.

1. [[Builders]]
2. [[Dispatchers]]
3. [[CoroutineContext]]
4. [[Job and Cancellation]]
5. [[SupervisorJob]]
6. [[Exception Handling]]
7. [[Structured Concurrency]]
8. [[Channels]]
9. [[Flow]]
10. [[StateFlow and SharedFlow]]
11. [[Coroutines test]]

#root
