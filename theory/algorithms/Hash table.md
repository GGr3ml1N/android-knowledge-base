**Хеш-таблица** — структура с доступом по ключу за O(1) в среднем. Идея: хеш-функция превращает ключ в индекс массива (bucket).

### Устройство и коллизии
- Коллизия: разные ключи → один bucket. Решения: **separate chaining** (список/дерево в bucket) и **open addressing** (пробинг: linear, quadratic, double hashing).
- Load factor = n / capacity. При превышении — rehash: увеличение массива и перераспределение (амортизированно O(1) на вставку).
- Худший случай O(n): все ключи в один bucket (или при плохой хеш-функции / DoS-атаке).

### hashCode и equals (Java/Kotlin — обязательный блок)
Контракт:
1. `a.equals(b)` → `a.hashCode() == b.hashCode()` (равные объекты — равные хеши).
2. Если hashCode разный → объекты точно не равны (быстрый выход).
3. Равные хеши НЕ гарантируют равенство → дальше проверяем equals.

Инвариант: если поля, участвующие в equals, неизменны — hashCode стабилен (поэтому ключи в HashMap должны быть immutable/эффективно immutable).
```kotlin
data class User(val id: String, val email: String) // hashCode по всем полям конструктора

// Ручной equals/hashCode по соглашению Java:
override fun equals(other: Any?): Boolean =
    other is User && other.id == id && other.email == email
override fun hashCode(): Int = 31 * id.hashCode() + email.hashCode()
```
Почему 31: нечётное простое, маленькое, `31 * i = (i << 5) - i` — JIT заменяет сдвигом.

### Java collections на хешах
| Класс | Дубликаты | Null | Порядок |
| :--- | :--- | :--- | :--- |
| HashMap | ключи нет | 1 ключ, n значений | нет |
| LinkedHashMap | ключи нет | да | insertion order |
| HashSet | нет | да | нет |
| IdentityHashMap | по ссылке | — | — |

### Что произойдёт в HashMap при плохом hashCode
Все в один bucket → O(n) на операцию. С Java 8 bucket после порога (TREEIFY_THRESHOLD = 8) превращается в сбалансированное дерево (красно-чёрное) → O(log n) в худшем случае (при Comparable ключах).

### Паттерны задач на собесах
1. **Two Sum**: комплемент в хеш-таблице — один проход O(n) (уже есть в [[tasks]]: Two Sum).
2. Подсчёт частот: `map.merge(x, 1, Int::plus)` — anagrams, most frequent.
3. Group by ключ: `groupingBy / groupBy` — все анаграммы вместе.
4. Скользящее окно + множество: longest substring without repeating characters.
5. Кэш с порядком: LinkedHashMap с `removeEldestEntry` — реализация LRU за O(1) (или `accessOrder = true`).

### HashMap vs ArrayList вопрос
Поиск по значению в ArrayList — O(n); в HashMap по ключу — O(1). Но: итерация по HashMap дороже (cache misses из-за разброса), память больше. Для маленьких объёмов (≤ десятков) ArrayList с линейным поиском часто быстрее на практике.

### В Kotlin
`hashMapOf`, `mutableMapOf` (LinkedHashMap по умолчанию — сохраняет порядок вставки!), `data class` генерирует корректные equals/hashCode автоматически.
