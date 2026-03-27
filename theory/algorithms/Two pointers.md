# From two sides
Дан отсортированный массив и число `target`. Нужно найти индексы двух чисел, сумма которых будет равна `target` иначе вернуть null
#### Пример
```kotlin
val list = listOf(-2, 1, 6, 9, 12, 21)
val target = 18 //в данном случае ответ 2 и 4
```

Концепция в том, чтобы поставить два указателя в начало и конец списка, складывать их сумму. Если сумма больше, чем `target`, то двигать правый, если меньше, то левый, если индексы равны - вывести их. 

```kotlin
fun twoSum(list: List<Int>, target: Int): Pair<Int, Int>? {  
    var left = 0  
    var right = list.size - 1  
    var sum = 0  
  
    while (left < right) {  
        sum = list[left] + list[right]  
  
        if (target == sum)  
            return left to right  
        if (sum > target)  
            right--  
        else  
            left++  
    }  
    return null  
}
```
Работает только для отсортированного массива

#### Признаки решения задач этим методом
- по условию дан отсортированный массив
- задача на палиндром
- ответ формируется за счет сужения области с двух сторон

# Fast and slow


# To each its own pointer

