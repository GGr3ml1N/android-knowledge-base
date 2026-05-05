# Окно фиксированной длины
#### Пример
Дан массив и число k. Нужно вернуть максимальное число k-подряд идущих элементов
```Kotlin
val list = listOf(3, 2, 0, 9, 1, 2, 8, 5, 2)
val k = 5 //Ответ 25
```

```Kotlin
fun kElementsMaxSum(intList: List<Int>, k: Int): Int {  
    var windowSum: Int = 0  
    for (i in 0 until k)  
        windowSum += intList[i]  
  
    var maxSum = windowSum  
  
    for (i in k until intList.size){  
        val l = i - k  
        windowSum = windowSum - intList[l] + intList[i]  
        maxSum = maxSum.coerceAtLeast(windowSum)  
    }  
    return maxSum  
}
```
#### Признаки решения задач этим методом
- дана переменная обозначающая размер заданного окна
- требуется работать с подряд идущими элементами 

# Непересекающиеся окна
#### Пример
Дан отсортированный по возрастанию массив уникальных целых чисел. Нужно сжать его идущие подряд целые числа
```Kotlin
val list = listOf(1, 2, 3, 5, 8, 9, 14) //Ответ в формате [1->3, 5, 8->9, 14]
```

```Kotlin
fun compressRanges(intList: List<Int>): List<String> {  
    var l = 0  
    var r = 0  
    val result = mutableListOf<String>()  
  
    while (l < intList.size) {  
        while (r + 1 < intList.size && intList[r] + 1 == intList[r + 1])  
            r++  
  
        if (r != l) {  
            result.add("${intList[l]}->${intList[r]}")  
        } else {  
            result.add(intList[l].toString())  
        }  
        l = r + 1  
        r += 1  
    }  
    return result  
}
```
#### Признаки решения задач этим методом
- нужно работать с подряд идущими элементами
- один элемент принадлежит к одной группе (группы не пересекаются)

# Пересекающиеся окна
#### Пример
Дан массив состоящий из нулей и единиц
```Kotlin
val list = listOf(1, 0, 1, 0, 1, 0, 1, 1)
```