**Multibindings** — возможность Dagger собирать **коллекцию зависимостей из нескольких модулей**: `Set<T>` (@IntoSet/@ElementsIntoSet) и `Map<K, V>` (@IntoMap). Ключевой механизм для плагинной архитектуры и фреймворков вроде ViewModelFactory/навигации.

### Set multibinding
Каждый модуль добавляет элемент в общий `Set<T>`:
```kotlin
@Module
object AnalyticsModule {
    @Provides @IntoSet
    fun provideFirebaseTracker(): AnalyticsTracker = FirebaseTracker()
}

@Module
object AppMetricaModule {
    @Provides @IntoSet
    fun provideAppMetrica(): AnalyticsTracker = AppMetricaTracker()
}

// Потребитель получает все сразу:
class Analytics @Inject constructor(
    private val trackers: Set<AnalyticsTracker>
)
```
`@ElementsIntoSet` — добавить несколько элементов одним `@Provides`.

### Map multibinding (@IntoMap)
```kotlin
@Module
abstract class ViewModelModule {
    @Binds @IntoMap
    @ViewModelKey(FeedViewModel::class)
    abstract fun bindFeedViewModel(vm: FeedViewModel): ViewModel

    @Binds @IntoMap
    @ViewModelKey(DetailViewModel::class)
    abstract fun bindDetailViewModel(vm: DetailViewModel): ViewModel
}

// Кастомный ключ:
@MapKey
annotation class ViewModelKey(val value: KClass<out ViewModel>)
```
Это ровно то, как устроен `ViewModelFactory` в Dagger-проектах:
```kotlin
class DaggerViewModelFactory @Inject constructor(
    private val viewModels: Map<Class<out ViewModel>, Provider<ViewModel>>
) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        val provider = viewModels[modelClass]
            ?: viewModels.entries.firstOrNull { modelClass.isAssignableFrom(it.key) }?.value
            ?: throw IllegalArgumentException("Unknown ViewModel: $modelClass")
        @Suppress("UNCHECKED_CAST")
        return provider.get() as T
    }
}
```
`Provider<T>` — ленивая версия: инстанс создаётся только при `get()` (важно для ViewModel'ей: без @Singleton, каждый раз новая, как требует ViewModelStore).

### Практическое применение
- **ViewModelFactory** — фабрика одна, мапа всех VM (см. выше).
- **Feature-модули регистрируют себя**: каждый фиче-модуль в своём Dagger-модуле делает `@IntoSet fun provideFeatureApi(): FeatureApi = FeedFeatureApi()`; app собирает `Set<FeatureApi>` для навигации/маршрутизации — модуляризация без правок app-модуля (см. [[Modularization]]).
- **Плагины/стратегии**: набор `SyncStrategy`, `Parser`, `Migration` — новая реализация добавляется без изменения консьюмера.

### Set vs Map
- Set: порядок не гарантирован (на практике — порядок инициализации модулей, не полагаться), все элементы одного типа.
- Map: явный ключ (Class, String, enum), получение конкретной реализации по ключу.

### Ограничения
- Ключи в map — одного типа (`@MapKey` с `allowKeysWithNoMapping` исключениями для примитивов).
- Нельзя смешивать обычный `@Provides Set<T>` с multibindings той же коллекции.
- Для lazy-коллекций: `Map<K, Provider<V>>`/`Set<Provider<T>>` — вариации, которые Dagger тоже собирает автоматически.
