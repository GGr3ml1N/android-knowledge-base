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

# Пересекающиеся окна