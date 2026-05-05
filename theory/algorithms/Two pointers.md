# С двух сторон
#### Пример
Дан отсортированный массив и число `target`. Нужно найти индексы двух чисел, сумма которых будет равна `target` иначе вернуть null
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

# Быстрый и медленный
#### Пример
Дан массив чисел
```Kotlin
val list = listOf(0, 1, 0, 0, 3, 12, 2)
```
Без создания нового массива переместить все нули в конец списка
```Kotlin
fun moveZeros(list: List<Int>): List<Int> {  
    val tempList = list.toMutableList()  
    var fast = 0  
    var slow = 0  
  
    while (fast < tempList.size) {  
        if (list[fast] != 0) {  
            tempList[slow] = tempList[fast].also { tempList[fast] = tempList[slow] }  
            slow++  
        }  
        fast++  
    }  
    return tempList  
}
```
#### Признаки решения задач этим методом
- нужна `in place` модификация для строки или массива
- требуется сохранить исходный порядок


# Каждому по указателю
#### Пример
Даны два отсортированных массива. Нужно вернуть их общие элементы
```kotlin
val firstList = listOf(0, 2, 4, 8, 8)
val secondList = listOf(1, 2, 2, 7, 8, 8, 8) //Ответ: 2, 8, 8
```

Идея в том, чтобы поставить указатель на начало первого массива и на начало второго массива и сравнивать значения. Если элемент в каком то массиве меньше, чем в другом - двигаем его указатель. Если значения равны - выписываем его

```kotlin
fun commonElements(firstList: List<Int>, secondList: List<Int>): List<Int> {  
    var firstPointer = 0  
    var secondPointer = 0  
    val resultList = mutableListOf<Int>()  
  
    while (firstPointer < firstList.size && secondPointer < secondList.size) {  
        when {  
            firstList[firstPointer] < secondList[secondPointer] -> firstPointer++  
            firstList[firstPointer] > secondList[secondPointer] -> secondPointer++  
            firstList[firstPointer] == secondList[secondPointer] -> {  
                resultList.add(firstList[firstPointer])  
                firstPointer++  
                secondPointer++  
            }  
        }  
    }  
    return resultList  
}
```
#### Признаки решения задач этим методом
- даны несколько массивов или строк
- нужно искать объединение/пересечение и т.д. у двух последовательностей 