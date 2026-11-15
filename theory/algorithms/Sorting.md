### Сводная таблица
| Алгоритм | Среднее | Худшее | Память | Стабильность | In-place |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Bubble / Selection | O(n²) | O(n²) | O(1) | нет/нет | да |
| Insertion | O(n²) | O(n²) | O(1) | да | да |
| Merge sort | O(n log n) | O(n log n) | O(n) | да | нет |
| Quick sort | O(n log n) | O(n²) | O(log n) стек | нет | да |
| Heap sort | O(n log n) | O(n log n) | O(1) | нет | да |
| Arrays.sort в Java | O(n log n) | — | O(n) | (TimSort/ DualPivot) | — |

### Quick sort — рабочая лошадка
Разделяй-и-властвуй: pivot, partition (всё меньше pivot — влево), рекурсия.
```kotlin
fun quickSort(a: IntArray, lo: Int = 0, hi: Int = a.size - 1) {
    if (lo >= hi) return
    val p = partition(a, lo, hi)
    quickSort(a, lo, p - 1)
    quickSort(a, p + 1, hi)
}

fun partition(a: IntArray, lo: Int, hi: Int): Int {
    val pivot = a[hi]
    var i = lo
    for (j in lo until hi) {
        if (a[j] < pivot) { a.swap(i, j); i++ }
    }
    a.swap(i, hi)
    return i
}
```
- Худший случай O(n²) — уже отсортированный массив + плохой pivot. Решения: случайный pivot, median-of-three.
- Почему быстрее merge: in-place, меньше памяти, кэш-френдли — в среднем выигрывает на практике.

### Merge sort
Разделяем пополам до базы, сливаем отсортированные половины. 
- Стабильность важна (где порядок равных элементов имеет значение).
- O(n) памяти — главный минус; на связных списках сортируется in-place по ссылкам.

### Heap sort — когда память критична
На двоичной куче: строим max-heap за O(n), n раз извлекаем максимум в конец. O(1) памяти, гарантированный O(n log n), но медленнее quick sort на практике (плохая локальность кэша).

### Сортировки за линейное время (знать существование и условия)
- **Counting sort** O(n + k): целые числа из диапазона k.
- **Radix sort** O(nk): поразрядно.
- **Bucket sort** O(n): равномерное распределение.
Условие: дополнительная информация о данных, не сравнения. Нижняя граница для сравнивающих сортировок — Ω(n log n).

### Что выбрать на собесе
- «Отсортировать 1 млн int» → quicksort/DualPivot (Arrays.sort) или radix sort.
- «Стабильная сортировка объектов» → merge sort / TimSort (Java: `Arrays.sort(Object[])` — TimSort стабильный; `Arrays.sort(int[])` — dual-pivot quicksort).
- «Ограниченная память» → heap sort.
- «Почти отсортированный массив» → insertion sort почти O(n).

### Связанные задачи
- K-й по величине элемент: quickselect — O(n) среднее (partition как в quicksort, но рекурсия только в одну сторону).
- Объединение k отсортированных списков: min-heap k элементов — O(n log k).
- Подсчёт инверсий: merge sort с подсчётом на слиянии — O(n log n).
