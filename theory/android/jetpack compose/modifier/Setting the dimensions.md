Для установки размеров компонентов в Jetpack Compose определен целый ряд модификаторов:

- `Modifier.height()`: устанавливает высоту
- `Modifier.width()`: устанавливает ширину
- `Modifier.fillMaxHeight()`: растягивает компонент по всей длине контейнера
- `Modifier.heightIn()`: устанавливает минимальную и максимальную высоту
- `Modifier.widthIn()`: устанавливает минимальную и максимальную ширину
- `Modifier.size()`: устанавливает размер
- `Modifier.sizeIn()`: устанавливает минимальный и максимальный размер

Для установки применяются единицы `dp` (*device-independent pixels/density-independent pixels* или независимые от устройства/плотности пиксели).

### Установка минимальных и максимальных размеров
Модификаторы `heightIn()` и `widthIn()` принимают два значения - минимальные и максимальные значения. Например:

```kotlin
modifier = Modifier
	.background(color=Color.LightGray)
	.widthIn(min = 100.dp, max = 400.dp)
    .heightIn(min=50.dp, max=300.dp)
```

С помощью `sizeIn()` можно сократить определение размеров:

```kotlin
modifier = Modifier
	.background(color=Color.LightGray)
    .sizeIn(minWidth = 100.dp, maxWidth = 400.dp, minHeight= 50.dp, maxHeight= 300.dp)
```

### Растяжение по всей длине и ширине контейнера

Отдельная группа модификаторов позволяет растянуть компонент по все длине и(или) ширине контейнера:
- `Modifier.fillMaxWidth()`: растягивает компонент по всей ширине контейнера
- `Modifier.fillMaxHeight()`: растягивает компонент по всей высоте контейнера
- `Modifier.fillMaxSize()`: растягивает компонент по всей длине и ширине контейнера

В качестве параметра модификаторы `Modifier.fillMaxWidth()`, `Modifier.fillMaxHeight()` и `Modifier.fillMaxSize()` принимают множитель, который устанавливает, какую часть от размеров контейнера займет компонент. Это значение имеет тип Float и находится в диапазоне от 0.0 до 1.0.

#done