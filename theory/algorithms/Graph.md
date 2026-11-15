### Представление
- **Список смежности**: `Map<V, List<V>>` / `Array<List<Int>>` — стандарт для разреженных графов, память O(V + E).
- Матрица смежности: O(V²) памяти, быстрая проверка ребра O(1).
- Список рёбер: для Kruskal, DSU-задач.

### DFS — в глубину
Стек (рекурсия или явный). O(V + E).
```kotlin
fun dfs(graph: Map<Int, List<Int>>, start: Int, visited: MutableSet<Int>) {
    if (!visited.add(start)) return
    for (next in graph[start].orEmpty()) dfs(graph, next, visited)
}
```
Применение: топологическая сортировка, поиск компонент связности, циклы, backtracking (пути, судоку).

### BFS — в ширину
Очередь. O(V + E). **Коротчайший путь в невзвешенном графе** — только BFS, DFS не гарантирует.
```kotlin
fun bfs(graph: Map<Int, List<Int>>, start: Int): Map<Int, Int> {
    val dist = mutableMapOf(start to 0)
    val q = ArrayDeque(listOf(start))
    while (q.isNotEmpty()) {
        val v = q.removeFirst()
        for (next in graph[v].orEmpty()) {
            if (next !in dist) { dist[next] = dist[v]!! + 1; q.addLast(next) }
        }
    }
    return dist
}
```

### Топологическая сортировка (DAG)
Порядок вершин, при котором каждое ребро идёт «вперёд». Применение: сборка задач (Gradle!), планирование, dependency resolution.
- Kahn: indegree-очередь (BFS-подобно), O(V + E). Цикл = в очереди остались вершины.
- DFS: добавляем в стек на выходе из рекурсии, разворачиваем.

### Dijkstra — кратчайший путь с весами (нет отрицательных)
Жадный выбор ближайшей вершины через min-heap: O((V + E) log V).
```kotlin
fun dijkstra(graph: Map<Int, List<Pair<Int, Int>>>, start: Int): Map<Int, Int> {
    val dist = mutableMapOf(start to 0).withDefault { Int.MAX_VALUE }
    val pq = PriorityQueue<Pair<Int, Int>>(compareBy { it.second })
    pq.offer(start to 0)
    while (pq.isNotEmpty()) {
        val (v, d) = pq.poll()
        if (d > dist.getValue(v)) continue // устаревшая запись в heap
        for ((to, w) in graph[v].orEmpty()) {
            val nd = d + w
            if (nd < dist.getValue(to)) { dist[to] = nd; pq.offer(to to nd) }
        }
    }
    return dist
}
```
С отрицательными весами — Bellman-Ford O(V·E). Между всеми парами — Floyd-Warshall O(V³).

### Union-Find (DSU)
Объединение множеств + проверка связности за почти O(1) (inverse Ackermann).
```kotlin
class DSU(n: Int) {
    val parent = IntArray(n) { it }
    fun find(x: Int): Int = if (parent[x] == x) x else find(parent[x]).also { parent[x] = it } // path compression
    fun union(a: Int, b: Int) { parent[find(a)] = find(b) }
}
```
Применение: Kruskal (MST), number of provinces, redundant connection.

### Классические задачи
- Number of Islands (DFS/BFS flood fill — паттерн сеток!).
- Course Schedule (топосорт + цикл).
- Clone Graph (hash map old→copy).
- Cheapest Flights Within K Stops (BFS/Dijkstra с ограничением).
- Network Delay Time (Dijkstra).

### Вопрос «DFS vs BFS — что выбрать»
- Невзвешенный кратчайший путь, уровни, ближайший узел → BFS.
- Все пути, структурные свойства (цикл, компоненты), память экономим (глубокое дерево → DFS итеративный), backtracking → DFS.
