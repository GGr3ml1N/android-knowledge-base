```kotlin
fun binarySearch(
	list: List<Int>, 
	target: Int
): Int {  
    var left = 0  
    var right = list.size - 1  
  
    while (left <= right) {  
        val mid = (left + right) / 2  
        val guess = list[mid]  
  
        if (guess == target) {  
            return mid  
        } else if (guess < target) {  
            left = mid + 1  
        } else {  
            right = mid - 1  
        }  
    }  
    return -1  
}
```

```kotlin
fun quickSortHeavy(
	list: List<Int>
): List<Int> {  
    if (list.size <= 1) return list  
  
    val pivot = list[list.size / 2]  
    val left = mutableListOf<Int>()  
    val right = mutableListOf<Int>()  
  
    list.forEach { item ->  
        if (item < pivot)  
            left.add(item)  
        if (item > pivot)  
            right.add(item)  
    }  
    return quickSortHeavy(left) + pivot + quickSortHeavy(right)  
}
```

```kotlin
fun quickSort(  
    list: MutableList<Int>,  
    left: Int = 0,  
    right: Int = list.size - 1  
): List<Int> {  
    if (left < right) {  
        val pivot = partition(list, left, right)  
        quickSort(list, left, pivot)  
        quickSort(list, pivot + 1, right)  
    }  
    return list  
}  
  
fun partition(  
    list: MutableList<Int>,  
    left: Int,  
    right: Int  
): Int {  
    if (left >= right) return 0  
    val pivot = list[(left + right) / 2]  
    var i = left - 1  
    var j = right + 1  
  
    while (true) {  
        do {  
            i++  
        } while (list[i] < pivot)  
        do {  
            j--  
        } while (list[j] > pivot)  
  
        if (i >= j) return j  
  
        list[i] = list[j].also { list[j] = list[i] }  
    }  
}
```