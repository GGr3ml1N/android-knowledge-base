### Базовая терминология
Корень, лист, родитель/ребёнок, высота (h), глубина. Полное двоичное дерево: n узлов → высота ~log n.

### BST (Binary Search Tree)
Свойство: для каждого узла левое поддерево < узел < правое поддерево.
- Поиск/вставка/удаление: O(h) — O(log n) при балансе, O(n) при вырождении (в отсортированный список).
- Обходы (порядок посещения):
  - **Inorder** (лево-корень-право) → отсортированная последовательность. Проверка BST = inorder-обход с проверкой возрастания.
  - Preorder (корень-лево-право) → копия дерева/сериализация.
  - Postorder (лево-право-корень) → безопасное удаление, размер поддеревьев.
  - Level-order (BFS по уровням) → очередь.

```kotlin
fun inorder(root: TreeNode?, out: MutableList<Int>) {
    if (root == null) return
    inorder(root.left, out)
    out.add(root.`val`)
    inorder(root.right, out)
}
```

### Сбалансированные деревья (знать на уровне идей)
Деревья, гарантирующие O(log n): AVL (строгий баланс по высоте, rotations), **Red-Black Tree** (Java TreeMap/TreeSet — баланс по цветам, меньше вращений), B-tree (БД, много потомков). В Android/Java повседневно: TreeSet/TreeMap — red-black.

### Heap vs BST
| | Heap | BST (balanced) |
| :--- | :--- | :--- |
| min/max за | O(1) | O(log n) |
| поиск произвольного | O(n) | O(log n) |
| удаление min/max | O(log n) | O(log n) |

### Trie (префиксное дерево)
Дерево по символам строки: общий префикс — общий путь от корня. Поиск/вставка O(L) длины строки. Применение: автодополнение, spell-check, IP-routing (binary trie). Частый вопрос «как реализовать autocomplete».

### Segment Tree / Fenwick (BIT)
- **Segment tree**: сумма/минимум на отрезке за O(log n), обновление O(log n), построение O(n). Массив-дерево размера 2n.
- **Fenwick**: префиксные суммы, O(log n) запрос/обновление, компактнее и проще.
Использование: range sum queries, counting inversions.

### Классические задачи на собесах
1. **Validate BST** — inorder + возрастание (или min/max границы рекурсией).
2. **LCA** (lowest common ancestor) — спуск по границам val.
3. **Max depth / balanced check** — postorder, возвращаем высоту или -1 при дисбалансе.
4. **Level order traversal** — BFS с очередью + разделение по уровням (size до обработки уровня).
5. **Serialize/deserialize BST**.
6. **Diameter of binary tree** — postorder, max через корень.

### Двоичная куча в массиве (связь с [[Stack Queue Heap]])
Родитель i → дети 2i+1, 2i+2; ребёнок i → родитель (i-1)/2. Именно так устроен `PriorityQueue` в Java.
