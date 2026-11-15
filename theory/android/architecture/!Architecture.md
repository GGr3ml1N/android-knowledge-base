Архитектура приложения — способ организации кода, при котором изменения в одном месте не ломают всё приложение. На Senior-собесах ждут не пересказ паттернов, а умение **обосновать выбор**: почему в проекте MVVM, а не MVI; где границы слоёв; как масштабируется решение на команду из 10+ человек.

### Общие принципы (к ним сводятся все ответы)
- **Separation of Concerns** — каждый класс делает одно (UI не знает про сеть, репозиторий не знает про Activity).
- **Dependency Inversion** — зависим от абстракций: `ViewModel` зависит от `NewsRepository` (интерфейс), не от `NewsRepositoryImpl` (см. [[Intro|@Binds в Dagger]]).
- **Unidirectional Data Flow (UDF)** — данные текут в одну сторону: View -> Intent/Action -> State -> View (см. [[Elementary State Manager]]).
- **Testability** — архитектура хороша, если юнит-тесты пишутся без Android-фреймворка.

1. [[MVVM]]
2. [[MVI]]
3. [[Clean Architecture]]
4. [[Modularization]]

#root
