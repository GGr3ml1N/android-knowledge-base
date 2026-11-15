**Бинарный поиск** — поиск в отсортированном массиве за O(log n): сравниваем с серединой, отбрасываем половину.

### Каноническая реализация (избегаем overflow)
```kotlin
fun binarySearch(a: IntArray, target: Int): Int {
    var lo = 0
    var hi = a.size - 1
    while (lo <= hi) {
        val mid = lo + (hi - lo) / 2   // а не (lo + hi) / 2 — переполнение!
        when {
            a[mid] == target -> return mid
            a[mid] < target -> lo = mid + 1
            else -> hi = mid - 1
        }
    }
    return -1
}
```

### Главная мысль для собеса
Бинарный поиск — это не «поиск элемента», а **поиск границы в монотонном пространстве ответов**. Монотонность: predicate P(x) меняется с false на true ровно один раз → ищем первый true.

### Поиск границы (lower/upper bound) — шаблон
```kotlin
// Первый индекс, где a[i] >= target (lower bound)
fun lowerBound(a: IntArray, target: Int): Int {
    var lo = 0
    var hi = a.size // важно: hi = size, ответ может быть n
    while (lo < hi) {
        val mid = lo + (hi - lo) / 2
        if (a[mid] >= target) hi = mid else lo = mid + 1
    }
    return lo // lo == hi — точка перехода
}
```
- `lowerBound(x)` — первое `>= x`.
- `upperBound(x)` — первое `> x` (условие `a[mid] > target`).
- Вставка с сохранением порядка: `add(lowerBound(x), x)`.

### Ответ в монотонной проверке (minimum feasible value)
Классика: «минимальная скорость, чтобы успеть до дедлайна», «минимальный размер корабля для грузов».
```kotlin
// минимальное x в [lo..hi], при котором feasible(x) == true
fun minFeasible(lo: Int, hi: Int, feasible: (Int) -> Boolean): Int {
    var l = lo; var r = hi
    while (l < r) {
        val mid = l + (r - l) / 2
        if (feasible(mid)) r = mid else l = mid + 1
    }
    return l
}
```
Если feasible монотонно, ответ за O(log n × cost(feasible)).

### Частые ошибки
1. `lo <= hi` vs `lo < hi`: первый — поиск элемента (инвариант: ответ в [lo,hi]), второй — поиск границы (ответ в [lo,hi)).
2. `mid + 1` / `mid - 1` против `mid` — уход в бесконечный цикл: в шаблоне границы никогда не пишем `lo = mid` вместе с `hi = mid` без mid+1/mid-1.
3. Переполнение `(lo + hi) / 2` → `lo + (hi - lo) / 2` (в Kotlin Int переполнение молча заворачивается!).

### Производные
- Поиск в «rotated sorted array» (массив сдвинут): определяем, какая половина отсортирована, отбрасываем ту, где точно нет ответа.
- Поиск пика в битоническом массиве.
- Экспоненциальный поиск: неизвестный размер / unbounded array.
