`mockk` — основная библиотека моков для Kotlin (аналог Mockito). Mockito на Kotlin имеет friction: final-классы, null-safety, проверка вызовов. Mockk решает из коробки.

### Создание моков
```kotlin
val repo = mockk<NewsRepository>()
val relaxed = mockk<NewsRepository>(relaxed = true) // любой вызов вернёт default (0, null, empty)
```

### Стаббинг
```kotlin
coEvery { repo.getNews() } returns listOf(news)          // suspend-функция
every { mapper.map(any()) } returns uiModel              // обычная
every { repo.getById(any()) } answers {                  // вычисление от аргументов
    News(id = firstArg(), title = "t")
}
every { repo.getById(-1) } throws IOException("net")     // ошибка
```

### Verify — проверка взаимодействий
```kotlin
coVerify { repo.getNews() }                        // вызывался ровно 1 раз
coVerify(exactly = 0) { repo.getNews() }           // НЕ вызывался
coVerify(atLeast = 1) { analytics.track(any()) }
coVerifyOrder { repo.getNews(); repo.saveToCache(any()) } // порядок
confirmVerified(repo)                              // ничего лишнего не вызвали
```

### Проверка аргументов
```kotlin
coVerify { repo.save(match { it.isNotEmpty() }) }
coVerify { repo.save(withArg { assertEquals("a", it.title) }) }
```

### clearMocks / unmockkAll
Между тестами: `clearMocks(repo)` — сбросить стаббинги и вызовы. В `@After` — `unmockkAll()`.

### Mockk vs Mockito
| | Mockk | Mockito (+mockito-kotlin) |
| :--- | :--- | :--- |
| Kotlin-first | да (DSL под Kotlin) | Java-библиотека, обёртки |
| suspend | `coEvery`/`coVerify` | `runBlocking`/`whenever` |
| final классы | из коробки | нужен mockito-inline |
| verify | богатый | богатый, но синтаксис тяжелее |

### Антипаттерны
- **Мокаем всё подряд** — тест превращается в проверку собственных моков, а не логики. Лучше реальный объект (mapper), чем мок.
- **verify на каждый чих** — ломается при рефакторинге. Проверяем значимые взаимодействия (side effects): сохранил в кэш, отправил аналитику.
- **Логика в ответах моков** — если в `answers` появляется `if`/цикл, скорее всего надо выделить fake-реализацию класса.

### Fake vs Mock
- **Mock** — заданное поведение, проверка вызовов. Для внешних зависимостей (сеть, analytics).
- **Fake** — упрощённая реальная реализация (in-memory кэш, fake DAO на HashMap). Для репозиториев/хранилищ — тестирует больше реальной логики.
