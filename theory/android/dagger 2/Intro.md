### 1. Святая троица Dagger 2

Чтобы Dagger понял, как собирать объекты, ему нужны три вещи:
1. **Кто требует?** (`@Inject`)
2. **Как создавать?** (`@Module` + `@Provides` / `@Binds`)
3. **Кто связывает?** (`@Component`)

#### `@Inject` (Конструктор — это ООП)
Всегда используй **инъекцию через конструктор**. Это делает класс независимым от Dagger, позволяет легко писать юнит-тесты (просто передаешь моки в конструктор) и соблюдает принцип Dependency Inversion.

```kotlin
class UserRepository @Inject constructor(
    private val api: Api,
    private val db: UserDao
) {
    // Логика
}
```
*Примечание: Инъекция в поля (`@Inject lateinit var ...`) используй **только** для классов Android Framework (Activity, Fragment, View), где ты не контролируешь конструктор.*

#### `@Module` и `@Provides` (Для чужих классов)
Если класс не имеет `@Inject` в конструкторе (например, `Retrofit`, `OkHttpClient` или интерфейсы), ты создаешь его в Модуле.

```kotlin
@Module
object NetworkModule {
    
    // Используем object, так как состояние не нужно (DRY & KISS)
    @Provides
    @Singleton
    fun provideOkHttpClient(): OkHttpClient {
        return OkHttpClient.Builder()
            .addInterceptor(HttpLoggingInterceptor())
            .build()
    }

    @Provides
    @Singleton
    fun provideRetrofit(client: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .client(client)
            .baseUrl("https://api.example.com/")
            .build()
    }
}
```

#### `@Component` (Мост между мирами)
Компонент — это интерфейс, который Dagger реализует за тебя. Он знает, как создавать объекты из Модулей и классов с `@Inject`.

```kotlin
@Singleton
@Component(modules = [NetworkModule::class, DatabaseModule::class])
interface AppComponent {
    // Точки входа (Members Injection или getters)
    fun inject(activity: MainActivity)
    
    // Или просто геттеры для ViewModel'ей
    fun userRepository(): UserRepository
}
```

---

### 2. ООП и `@Binds` (Избегаем бойлерплейта)

В чистом коде мы зависим от абстракций (интерфейсов), а не от реализаций. 
Если тебе нужно связать Интерфейс с Реализацией, **не используй `@Provides`** (это нарушение DRY, так как ты пишешь лишний код). Используй `@Binds`.

```kotlin
// Интерфейс
interface UserRepository { ... }

// Реализация
class UserRepositoryImpl @Inject constructor(...) : UserRepository { ... }

// Модуль для связывания (Обязательно abstract class или interface!)
@Module
abstract class RepositoryBindingsModule {
    
    // Dagger сам поймет, что нужно создать UserRepositoryImpl 
    // и вернуть его как UserRepository
    @Binds
    abstract fun bindUserRepository(impl: UserRepositoryImpl): UserRepository
}
```

---

### 3. Скоупы (Scopes) и время жизни

Скоупы в Dagger привязаны **не к классам, а к Компонентам**. 
Если Компонент помечен `@Singleton`, то все зависимости внутри него, помеченные `@Singleton`, будут созданы **один раз** и жить столько, сколько живет Компонент.

```kotlin
@Singleton // Скоуп Компонента
@Component(...)
interface AppComponent { ... }

@Module
object NetworkModule {
    @Provides
    @Singleton // Скоуп зависимости. Создастся 1 раз на весь AppComponent.
    fun provideRetrofit(): Retrofit = ...
}
```

Никогда не делай всё `@Singleton`. Если ты пометим `Activity`-зависимость (например, `Presenter` или `ViewModel`) как `@Singleton` в `AppComponent`, ты получишь **утечку памяти**, так как `AppComponent` живет вечно (в `Application`), и будет держать ссылку на убитую `Activity`.

---

### 4. Иерархия Компонентов и Subcomponents

В Android у нас есть иерархия: `Application` -> `Activity` -> `Fragment`. 
Dagger 2 позволяет строить иерархию компонентов, чтобы зависимости нижнего уровня могли использовать зависимости верхнего.

**Правильный подход (Subcomponents):**
`AppComponent` (живет вечно) -> `ActivityComponent` (живет пока жива Activity) -> `FragmentComponent`.

```kotlin
// 1. Родительский компонент
@Singleton
@Component(modules = [AppModule::class])
interface AppComponent {
    // Фабрика для создания дочернего компонента
    fun activityComponentFactory(): ActivityComponent.Factory
}

// 2. Дочерний компонент
@ActivityScope // Кастомный скоуп
@Subcomponent(modules = [ActivityModule::class])
interface ActivityComponent {
    fun inject(activity: MainActivity)
    
    @Subcomponent.Factory
    interface Factory {
        fun create(@BindsInstance activity: MainActivity): ActivityComponent
    }
}
```
*Зачем это нужно?* Чтобы `Fragment` мог получить `Presenter`, который зависит от `Activity`, но не был `@Singleton`.

---

### 5. Qualifiers (Решаем неоднозначности)

Что если тебе нужно два разных `String` или два разных `OkHttpClient`? 
**Плохо (Нарушает типобезопасность):** `@Named("base_url")`
**Хорошо (ООП, компилятор проверит):** Создаем свои аннотации.

```kotlin
@Qualifier
@Retention(AnnotationRetention.RUNTIME)
annotation class BaseUrl

@Qualifier
@Retention(AnnotationRetention.RUNTIME)
annotation class CdnUrl

// В Модуле:
@Provides
@BaseUrl
fun provideBaseUrl(): String = "https://api.example.com/"

// В Конструкторе:
class ApiClient @Inject constructor(
    @BaseUrl private val url: String,
    private val client: OkHttpClient
)
```

---