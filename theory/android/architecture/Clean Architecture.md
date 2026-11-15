**Clean Architecture** (Роберт Мартин, «Чистая архитектура») — концентрические слои с правилом **зависимостей: внутренние слои не знают о внешних**. UI и БД — детали, которые можно заменить; бизнес-правила — ядро.

### Слои (в Android-адаптации)
```
Presentation (ViewModel, MVI/MVVM)  ->  Domain (UseCases, Entities)  ->  Data (RepositoryImpl, DAO, API)
```
- **Domain**: юзкейсы (один класс = одна операция: `GetNewsFeed`, `LoginUser`), чистые entity. Не зависит от Android SDK (кроме, спорно, аннотаций). Для небольших проектов может быть избыточен.
- **Data**: репозиторий — единая точка доступа к данным (network + cache strategy: cache-first, network-first, stale-while-revalidate).
- **Presentation**: ViewModel + StateHolder.

### Правило зависимостей
`Presentation --> Domain <-- Data`. Presentation и Data зависят от Domain (интерфейсы), Domain ни от кого не зависит.

```kotlin
// Domain — ничего не знает про сеть/БД
interface NewsRepository {
    suspend fun getNews(): List<News>
    fun observeNews(): Flow<List<News>>
}

class GetNewsFeed(private val repo: NewsRepository) {
    suspend operator fun invoke(): List<News> = repo.getNews()
}

// Data — реализация, знает про Retrofit/Room
class NewsRepositoryImpl @Inject constructor(
    private val api: NewsApi,
    private val dao: NewsDao
) : NewsRepository { ... }
```

### UseCase — зачем
- Имя операции становится частью кода (`GetNewsFeed`, а не `repo.getNews()` + логика).
- Единое место для бизнес-правил (валидация, маппинг домен-модели).
- Тонкая ViewModel: VM вызывает юзкейсы, не зная про источники.

Критика: на маленьких экранах юзкейсы-обёртки над одним вызовом репозитория — бойлерплейт. Реалистичный подход: юзкейс появляется, когда есть бизнес-правило или композиция.

### Mapper'ы и модели
Каждый слой имеет свои модели: `NewsDto` (data, для парсинга JSON), `NewsEntity` (data, для Room), `News` (domain). Маппинг — в data-слое. Не тащим DTO через все слои: схема API меняется — меняем один модуль.

### Что ответить на собесе
- «Зачем Clean Architecture?» — тестируемость бизнес-логики без фреймворка, независимость от внешних деталей (сменили Retrofit на Ktor — тронули один модуль), параллельная работа команд по слоям.
- «Когда НЕ нужна?» — прототип, маленькая команда, CRUD-экраны: трёхслойная архитектура (UI -> VM -> Repository) с юзкейсами «по необходимости».
- «Как связано с MVVM/MVI?» — MVVM/MVI это Presentation-слой; Clean — границы слоёв. Они не конкурируют, а комбинируются: MVI-экран в Clean Architecture.
