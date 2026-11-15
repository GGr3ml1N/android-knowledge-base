**Room** — ORM/обёртка над SQLite: compile-time проверка SQL-запросов, integration с корутинами/Flow.

### Три основных компонента
```kotlin
@Entity(tableName = "news")
data class NewsEntity(
    @PrimaryKey val id: String,
    val title: String,
    val publishedAt: Long
)

@Dao
interface NewsDao {
    @Query("SELECT * FROM news ORDER BY publishedAt DESC")
    fun observeNews(): Flow<List<NewsEntity>>   // реактивно: эмит при изменении таблицы

    @Query("SELECT * FROM news WHERE id = :id")
    suspend fun getById(id: String): NewsEntity?

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun upsertAll(items: List<NewsEntity>)

    @Query("DELETE FROM news")
    suspend fun clear()
}

@Database(entities = [NewsEntity::class], version = 3)
abstract class AppDatabase : RoomDatabase() {
    abstract fun newsDao(): NewsDao
}
```

### Миграции
При `version` bump без миграции — `IllegalStateException` (fallbackToDestructiveMigration — только в dev).
```kotlin
val MIGRATION_2_3 = object : Migration(2, 3) {
    override fun migrate(db: SupportSQLiteDatabase) {
        db.execSQL("ALTER TABLE news ADD COLUMN source TEXT NOT NULL DEFAULT ''")
    }
}
Room.databaseBuilder(context, AppDatabase::class.java, "app.db")
    .addMigrations(MIGRATION_2_3)
    .build()
```
Собес-вопросы: как мигрировать с бэкапом данных (создать новую таблицу → скопировать → удалить старую → rename), тестирование миграций (`MigrationTestHelper`).

### Реактивность: Flow в DAO
`Flow<List<T>>` эмитит заново при ЛЮБОЙ записи в таблицу (invalidation tracker). Паттерн single source of truth:
```
ViewModel -> observeNews() (Flow) -> UI
        ^- репозиторий пишет свежие данные после сетевого запроса
```
UI всегда смотрит в БД; сеть — источник обновлений БД. Проброс настроек/транзакций: `@Transaction` для мультизапросных операций (consistency + single invalidation event).

### Производительность и ограничения
- Запросы выполняются не на главном потоке при использовании suspend/Flow (Room сам переключается на его executor; свой executor — через `setQueryExecutor`).
- `@Transaction` + Flow: транзакционные DAO-методы с Flow недопустимы (@Transaction для suspend-функций ок).
- Пагинация: Paging 3 + `PagingSource` из DAO (`@Query(... LIMIT :size OFFSET :offset)` или `RoomPagingSource` по ключу).
- Индексы: `@Entity(indices = [Index("publishedAt")])`, составные индексы, `UNIQUE`.
- Отношения: `@Embedded` (flatten), `@Relation` + `@Transaction` (one-to-many), Junction для many-to-many.
- Type converters для нестандартных типов: `@TypeConverter class DateConverter { @TypeConverter fun fromTimestamp(v: Long?) = v?.let { Date(it) } ... }`.

### Стандартные вопросы
- «Room vs SQLiteOpenHelper/ContentProvider» — compile-time safety, корутины/Flow из коробки, маппинг в data class'ы, миграции как код.
- «Flow эмитит при записи, даже если данные не изменились?» — да, invalidation на уровне таблицы; фильтрация — в репозитории (`distinctUntilChanged`).
- «Как тестировать DAO?» — in-memory database (`Room.inMemoryDatabaseBuilder`), важно: она не персистентна между тестами и не тестирует миграции (для них — MigrationTestHelper с реальным файлом).
