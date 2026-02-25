### Задача
Given a string `s` containing just the characters `'('`, `')'`, `'{'`, `'}'`, `'['` and `']'`, determine if the input string is valid.

An input string is valid if:
1. Open brackets must be closed by the same type of brackets.
2. Open brackets must be closed in the correct order.
3. Every close bracket has a corresponding open bracket of the same type.

### Перевод
Дана строка s, содержащая только символы `'('`, `')'`, `'{'`, `'}'`, `'['` и `']'`. Определите, является ли входная строка корректной.

Входная строка считается корректной, если:
1. Открывающие скобки должны закрываться скобками того же типа.
2.  Открывающие скобки должны закрываться в правильном порядке.
3.  Каждая закрывающая скобка имеет соответствующую ей открывающую скобку того же типа.

### Решение
```kotlin
fun isValid(s: String): Boolean {  
    if (s.length <= 1) return false  
    val expected: LinkedList<Char> = LinkedList()  
  
    s.forEach { char ->  
        when (char) {  
            '(', '[', '{' -> {  
                expected.addLast(char.closing())  
                return@forEach  
            }  
            ')', ']', '}' -> {  
                if (expected.isEmpty() || char != expected.last()) return false  
                expected.removeLast()  
            }  
        }  
    }  
    return expected.isEmpty()  
}  
  
fun Char.closing(): Char = when (this) {  
    '(' -> ')'  
    '{' -> '}'  
    '[' -> ']'  
    else -> throw IllegalArgumentException()  
}
```