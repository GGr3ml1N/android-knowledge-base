### Задача
Write a function to find the longest common prefix string amongst an array of strings.

If there is no common prefix, return an empty string `""`.
### Перевод 
Напишите функцию, которая находит самую длинную общую префикс‑строку (начальный фрагмент, совпадающий у всех строк) среди массива строк.

Если общего префикса нет, верните пустую строку `""`.
### Решение
#### Не мое, потому что я тупой
```kotlin
fun longestCommonPrefix(strings: Array<String>): String {  
    if (strings.isEmpty()) return ""  
    var prefix = strings[0]  
    strings.forEach { s ->  
        while (s.indexOf(prefix) != 0) {  
            prefix = prefix.substring(0, prefix.length - 1)  
        }  
    }  
    return prefix  
}
```