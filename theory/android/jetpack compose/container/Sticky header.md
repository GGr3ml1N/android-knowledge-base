`LazyColumn` и `LazyRow` позволяют использовать прикрепленные заголовки или `sticky headers`. То есть мы можем сгруппировать элементы списка под соответствующим заголовком. И припрепленные заголовки остаются видимыми на экране во время прокрутки текущей группы. Как только группа прокручивается из поля зрения, ее место занимает заголовок следующей группы.

Для создания прикрепленных заголовков применяется функция `LazyListScope.stickyHeader()`. А содержимое списка должно храниться в массиве или списке, который трансформируется в группы с помощью функции Kotlin `groupBy()`. Функция `groupBy()` принимает лямбду-селектор, которая определяет, как данные должны быть сгруппированы. Этот селектор затем служит ключом для доступа к элементам каждой группы.

```kotlin
val phones = 
	listOf(
		"Apple iPhone 15 Pro", 
		"Realme 11 PRO", 
		"Google Pixel 5", 
		"Samsung Galaxy S24 Ultra", 
		"Google Pixel 6",
		"Samsung Galaxy S21 FE", 
		"Apple iPhone 15 Pro Max", 
		"Xioami Redmi Note 12", 
		"Xiaomi Redmi 12",
        "Apple iPhone 13", 
        "Google Pixel 6", 
        "Apple iPhone 14",
        "Realme C30s", 
        "Realme Note 50"
    )
// создаем группы
val groups = phones.groupBy { it.substringBefore(" ") }
LazyColumn(
    contentPadding = PaddingValues(5.dp)
) {
    groups.forEach { (brand, models) ->
        stickyHeader {
            Text(
	            text = brand,
                fontSize = 28.sp,
                color = Color.White,
	            modifier = Modifier
		            .background(Color.Gray)
		            .padding(5.dp)
		            .fillMaxWidth()
            )
        }
        items(models) { model ->
            Text(
	            text = model, 
	            modifier = Modifier.padding(5.dp), 
	            fontSize = 28.sp
	        )
        }
    }
}
```

![[stiky_header.png]]

#done