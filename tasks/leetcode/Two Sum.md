Given an array of integers `nums` and an integer `target`, return _indices_ of the two numbers such that they add up to `target`.
You may assume that each input would have **_exactly_ one solution**, and you may not use the _same_ element twice.
You can return the answer in any order.

Перевод:
Даны массив целых чисел `nums` и целое число `target`. Верните _индексы_ двух чисел так, чтобы их сумма была равна `target`.
Можно считать, что для каждого входного набора данных существует ровно одно решение, и нельзя использовать один и тот же элемент дважды.
Ответ можно вернуть в любом порядке.
<br>

Ответ: 
```kotlin
fun twoSum(nums: IntArray, target: Int): IntArray {  
    val map = mutableMapOf<Int, Int>()  
  
    for (i in nums.indices) {  
        val result = target - nums[i]  
  
        if (map.containsKey(result)) {  
            return intArrayOf(map[result]!!, i)  
        }  
  
        map[nums[i]] = i  
    }  
  
    return intArrayOf()  
}
```

