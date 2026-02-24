### Задача
Roman numerals are represented by seven different symbols: `I`, `V`, `X`, `L`, `C`, `D` and `M`.

| **Symbol** | **Value** |
|:----------:|:---------:|
|     I      |     1     |
|     V      |     5     |
|     X      |    10     |
|     L      |    50     |
|     C      |    100    |
|     D      |    500    |
|     M      |   1000    |

For example, `2` is written as `II` in Roman numeral, just two ones added together. `12` is written as `XII`, which is simply `X + II`. The number `27` is written as `XXVII`, which is `XX + V + II`.

Roman numerals are usually written largest to smallest from left to right. However, the numeral for four is not `IIII`. Instead, the number four is written as `IV`. Because the one is before the five we subtract it making four. The same principle applies to the number nine, which is written as `IX`. There are six instances where subtraction is used:

- `I` can be placed before `V` (5) and `X` (10) to make 4 and 9. 
- `X` can be placed before `L` (50) and `C` (100) to make 40 and 90. 
- `C` can be placed before `D` (500) and `M` (1000) to make 400 and 900.

Given a roman numeral, convert it to an integer.

### Перевод:
Римские цифры обозначаются семью различными символами: `I`, `V`, `X`, `L`, `C`, `D` и `M`.

|**Символ**|**Значение**|
|---|---|
|I|1|
|V|5|
|X|10|
|L|50|
|C|100|
|D|500|
|M|1000|

Например, число `2` записывается как `II` в римской нумерации — просто две единицы, сложенные вместе. Число `12` записывается как `XII`, что означает просто `X + II`. Число `27` записывается как `XXVII`, то есть `XX + V + II`.

Обычно римские цифры записываются от наибольшего значения к наименьшему — слева направо. Однако цифра для обозначения числа четыре — это не `IIII`. Вместо этого число четыре записывается как `IV`: поскольку единица стоит перед пятёркой, мы вычитаем её, получая четыре. Тот же принцип применяется к числу девять, которое записывается как `IX`. Существует шесть случаев, когда используется вычитание:

- `I` можно поставить перед `V` (5) и `X` (10), чтобы получить 4 и 9.
- `X` можно поставить перед `L` (50) и `C` (100), чтобы получить 40 и 90.
- `C` можно поставить перед `D` (500) и `M` (1000), чтобы получить 400 и 900.

Дано римское число — преобразуйте его в целое число.

### Ответ 
#### Заколхозил сам
```kotlin
fun romanToInt(s: String): Int {
    val chars = s.toList()
    var tempInt = 0
    s.indices.forEach up@{ i ->
        (i + 1 until s.length).forEach { j ->
            when (val char = chars[i]) {
                'I' -> when (chars[j]) {
                    'V', 'X' -> {
                        tempInt += (char.digit * -1)
                        return@up
                    }

                    else -> {
                        tempInt += (char.digit)
                        return@up
                    }
                }

                'X' -> when (chars[j]) {
                    'L', 'C' -> {
                        tempInt += (char.digit * -1)
                        return@up
                    }

                    else -> {
                        tempInt += (char.digit)
                        return@up
                    }
                }

                'C' -> when (chars[j]) {
                    'D', 'M' -> {
                        tempInt += (char.digit * -1)
                        return@up
                    }

                    else -> {
                        tempInt += (char.digit)
                        return@up
                    }
                }

                else -> {
                    tempInt += (char.digit)
                    return@up
                }
            }
        }
    }
    tempInt += (chars[chars.size - 1].digit)
    return tempInt
}

val Char.digit: Int
    get() = when (this) {
        'I' -> 1
        'V' -> 5
        'X' -> 10
        'L' -> 50
        'C' -> 100
        'D' -> 500
        'M' -> 1000
        else -> throw IllegalArgumentException()
    }
```

#### Как по красоте
```kotlin
fun romanToInt(s: String): Int {  
    var result = 0  
    var prevValue = 0  
  
    for (char in s.reversed()) {  
        val currentValue = char.digit  
        if (currentValue < prevValue) {  
            result -= currentValue  
        } else {  
            result += currentValue  
        }  
        prevValue = currentValue  
    }  
  
    return result  
}  
  
val Char.digit: Int  
    get() = when (this) {  
        'I' -> 1  
        'V' -> 5  
        'X' -> 10  
        'L' -> 50  
        'C' -> 100  
        'D' -> 500  
        'M' -> 1000  
        else -> throw IllegalArgumentException()  
    }
```
