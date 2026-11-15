### Аннотации
```kotlin
@Test          // тестовый метод
@Before         // перед КАЖДЫМ тестом
@After          // после каждого
@BeforeClass    // один раз перед всеми ( companion object / @JvmStatic )
@Ignore         // пропустить
@Rule           // внешние ресурсы/поведение (MainDispatcherRule, TemporaryFolder)
```

### Assert
```kotlin
assertEquals(expected, actual)
assertNotEquals(a, b)
assertTrue(x > 0)
assertNull / assertNotNull
assertThrows(IllegalStateException::class.java) { code() }
fail("не должны были сюда попасть")
```

### Именование
Формат `method_condition_expectedResult` или осмысленные фразы через backticks:
```kotlin
@Test fun `load fails with error state when network throws`()
@Test fun load_networkError_errorState() // альтернатива
```

### Структура теста: AAA
```kotlin
@Test
fun `sum returns total for positive numbers`() {
    // Arrange (given)
    val calc = Calculator()

    // Act (when)
    val result = calc.sum(2, 3)

    // Assert (then)
    assertEquals(5, result)
}
```

### Один тест — один сценарий
Плохо: проверять success и error в одном методе. Хорошо: минимум 3 теста на метод — happy path, error path, edge case (null/пусто/граница).

### Параметризованные тесты (JUnit 4: `Parameterized` / JUnit 5: `@ParameterizedTest` + `@ValueSource`/`@MethodSource`)
Позволяют прогнать один тест на наборе входных данных без копипасты.

### Правило хорошего теста
- **F.I.R.S.T.**: Fast, Independent, Repeatable, Self-validating, Timely.
- Тест не должен зависеть от порядка выполнения и внешнего состояния (сеть, файлы, текущее время — мокаем `Clock`/`Instant`).
- Покрытие (coverage) — инструмент, а не цель: 100% на generated-коде хуже, чем 70% на критичной логике.

### Пример edge cases для ViewModel/репозитория
- Успех, сетевая ошибка (IOException), неожиданная ошибка, отмена корутины, повторный вызов до завершения первого (debounce/защита от даблклика), пустой ответ.
