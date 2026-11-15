**Navigation Component** — фреймворк навигации: граф экранов ( destinations + actions), back stack, safe args, deep links.

### Состав
1. **NavGraph** (XML или Kotlin DSL) — декларативное описание экранов и переходов.
2. **NavController** — управляет back stack, navigate(), popBackStack(), currentDestination.
3. **NavHost** (FragmentContainerView / Compose NavHost) — контейнер, отображающий текущий destination.

```xml
<navigation ...>
    <fragment android:id="@+id/feedFragment" ...>
        <action android:id="@+id/toDetail"
                app:destination="@id/detailFragment" />
    </fragment>
    <fragment android:id="@+id/detailFragment" ...>
        <argument android:name="newsId" app:argType="string" />
    </fragment>
</navigation>
```

```kotlin
// View: навигация выполняет View (не ViewModel!)
findNavController().navigate(R.id.toDetail, bundleOf("newsId" to id))
```

### Safe Args (type-safe аргументы)
Генерирует классы Directions/Args. Для Compose Navigation — type-safe routes через kotlinx.serialization:
```kotlin
@Serializable data class Detail(val newsId: String)
navController.navigate(Detail(newsId))
```

### Compose Navigation
```kotlin
val navController = rememberNavController()
NavHost(navController = navController, startDestination = "feed") {
    composable("feed") { FeedScreen(onOpenDetail = { id -> navController.navigate("detail/$id") }) }
    composable("detail/{newsId}") { backStackEntry ->
        val newsId = backStackEntry.arguments?.getString("newsId")!!
        DetailScreen(newsId)
    }
}
```
Важно: получение ViewModel через `viewModel()` внутри composable — scoped на back stack entry (per-destination ViewModel; `navController.getBackStackEntry(route)` для shared ViewModel между destination'ами).

### Ключевые концепции
- **Back stack**: стандартная модель «очередь экранов»; system back = popBackStack. `popUpTo + inclusive` — удаление экранов из стека при переходе (паттерн «одноразовый экран», «logout → очистить стек до auth»).
- **Single Task поведение**: `launchSingleTop = true` — не дублировать destination, если он уже on top.
- **Deep links**: URI-pattern в графе (`<deepLink app:uri="app://detail/{newsId}" />`) + App Links (verified домены) / Custom Schemes. Поднимают соответствующий back stack (nested graph).
- **Nested graphs**: группировка экранов фичи (модульная навигация), каждый может быть корнем для вкладок (bottom nav + multiple back stacks с NavigationUI).
- **SavedStateHandle как источник аргументов**: destination arguments автоматически попадают в SavedStateHandle ViewModel — зачем передавать через конструктор вручную.

### Архитектурные решения (собес Senior)
- **Навигация из ViewModel?** Спорно. Чистый подход: VM эмитит событие навигации (SharedFlow/Channel), View вызывает NavController. Прямой вызов из VM привязывает её к фреймворку и ломает тестируемость.
- **Result между экранами**: через SavedStateHandle предыдущего entry (`getPreviousBackStackEntry()?.savedStateHandle?.set("key", result)`) или shared ViewModel scoped на родительский nav graph.
- **Single Activity**: рекомендуемая схема — одна Activity, фрагменты/Compose-destinations; навигация между Activity — только для редких случаев (процессы, большие изолированные фичи).

### Стандартные вопросы
- «Как пережить процесс death с back stack?» — NavController автоматически сохраняет back stack в Bundle (onSaveInstanceState), восстанавливается сами.
- «Как сделать bottom navigation с отдельными стеками?» — NavigationUI.setupWithNavController + multiple back stack (Androidx Navigation 2.4+, сохранение/восстановление каждой вкладки).
- «Жизненный цикл ViewModel при навигации?» — scoped на NavBackStackEntry: пока entry в стеке — VM жива.
