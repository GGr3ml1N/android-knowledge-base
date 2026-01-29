Используя различные типы контейнеров, можно создавать более сложные композиции компоновок. Например, возьмем приложение со следующим кодом:

```kotlin
data class Language(val name: String, val hexColor:Long)

val langs = listOf(Language("Kotlin",0xff16a085),
        Language("Java",0xff2980b9),
        Language("JavaScript",0xffe74c3c),
        Language("Python", 0xffd35400))

setContent {
    Column {
        for(lang in langs) {
            Row(
	            modifier = Modifier
		            .padding(10.dp)
		            .fillMaxWidth()
		    ) {   
			    Box(
					modifier = Modifier
						.size(50.dp)
						.background(Color(lang.hexColor))
				)
                Text(
	                text = lang.name, 
	                fontSize = 28.sp, 
	                modifier = Modifier.padding(10.dp)
                )
            }
        }
    }
}
```