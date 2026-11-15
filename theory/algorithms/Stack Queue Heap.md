### Stack (стек)
LIFO — последний пришёл, первый ушёл.
```kotlin
val stack = ArrayDeque<Int>()
stack.addLast(x)      // push
stack.removeLast()    // pop
stack.last()          // peek
```
**Применение**: проверка сбалансированности скобок (Valid Parentheses), обратная польская нотация, отмена операций, DFS (итеративный), монотонный стек (next greater element).

Собес-факт: рекурсия = неявный стек вызовов; итеративный DFS со своим стеком — эквивалент.

### Queue (очередь)
FIFO — первый пришёл, первый ушёл.
```kotlin
val q = ArrayDeque<Int>()
q.addLast(x)       // offer / enqueue
q.removeFirst()    // poll / dequeue
q.first()          // peek
```
**Применение**: BFS, кэш первого вошедшего (FIFO-буфер), обработка событий, round-robin.

В Java: `ArrayDeque` предпочтительнее `Stack` (Vector-синхронизация — лишний overhead) и `LinkedList` (как queue — кэш-френдли ArrayDeque лучше).

### Deque (двусторонняя очередь)
Добавление/удаление с обеих сторон за O(1): sliding window maximum (монотонный deque), palindrome check, работа как стек и как очередь одновременно.

### Приоритетная очередь / Heap (куча)
**Двоичная куча** — полное двоичное дерево в массиве, где родитель ≥ детей (max-heap) или ≤ (min-heap).
```kotlin
val minHeap = PriorityQueue<Int>()          // минимум наверху
val maxHeap = PriorityQueue<Int>(compareByDescending { it })
minHeap.offer(x)      // O(log n)
minHeap.poll()        // извлечь min, O(log n)
minHeap.peek()        // min, O(1)
```
Почему log n: sift up/down — высота дерева log n. Построение кучи из массива — O(n) (не O(n log n)!), извлечение всех элементов — O(n log n).

**Применение — топ-K задачи** (главный паттерн):
```kotlin
// K самых частых элементов: min-heap размера K
// O(n log K) вместо O(n log n) полной сортировки
fun topK(nums: IntArray, k: Int): List<Int> {
    val freq = nums.toList().groupingBy { it }.eachCount()
    val heap = PriorityQueue<Map.Entry<Int, Int>>(compareBy { it.value })
    for (e in freq.entries) {
        heap.offer(e)
        if (heap.size > k) heap.poll()
    }
    return heap.map { it.key }
}
```
Другие применения: merge k sorted lists, медиана потока (two heaps), Dijkstra, Huffman.

### Сравнение на собесе: что выбрать
- Нужен мин/макс постоянно → heap.
- Нужен поиск произвольного элемента → heap не подходит (O(n) поиск) → tree set/hash set.
- «Последние N элементов» → deque с ограничением.

### Задачи для практики
- Min Stack (стек с O(1) min — два стека).
- Sliding Window Maximum (deque, O(n)).
- Task Scheduler / reorganize string (heap + greedy).
- Kth Largest Element in a Stream (heap размера K).
- Largest Rectangle in Histogram (монотонный стек — hard, но классика).
