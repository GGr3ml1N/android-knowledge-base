**Полиморфизм** — «много форм»: один интерфейс обращения, разное поведение реализаций.

### Виды полиморфизма
1. **Подтиповый (runtime, динамический)** — переопределённые методы, выбор реализации в рантайме по фактическому типу объекта:
```java
Animal a = new Dog(); // статический тип Animal, динамический Dog
a.speak(); // "Woof" — JVM смотрит на объект, не на ссылку
```
2. **Ad-hoc (compile-time)** — перегрузка (overloading): выбор метода по типам аргументов на этапе компиляции; дженерики (параметрический полиморфизм).

### Как работает подтиповый полиморфизм (JVM)
- Каждый класс имеет **vtable** (таблицу виртуальных методов) со ссылками на переопределённые методы. Вызов `invokevirtual` идёт через vtable объекта — поэтому цена вызова чуть выше, но JIT-оптимизации (inline caching, monomorphic call sites) сводят её к минимуму.
- Статические, private и final методы не виртуальны — вызываются напрямую (`invokestatic`/`invokespecial`).

### Правила приведения типов
```java
Animal a = new Dog();
Dog d = (Dog) a;        // ок — объект действительно Dog
Animal c = new Cat();
Dog e = (Dog) c;        // ClassCastException в рантайме!
```
`instanceof` (pattern matching с Java 16: `if (a instanceof Dog d)`) — проверка перед кастом.

### Полиморфизм и абстракции
```java
interface Payment { void pay(BigDecimal amount); }

class CardPayment implements Payment { public void pay(...) { /* карта */ } }
class SbpPayment  implements Payment { public void pay(...) { /* СБП */ } }

void checkout(Payment p) { p.pay(amount); } // не знаем и не важно, какая реализация
```
Это и есть [[!Architecture|Dependency Inversion]] в действии: код зависит от абстракции, реализации подставляются (в Android — через Dagger, см. [[Intro]]).

### Переопределение equals/hashCode (вопрос-спутник)
Полиморфизм в equals ломается при смешивании иерархий (symmetry violation): `a.equals(b)` не равно `b.equals(a)`, если b — подкласс с дополнительными полями. Решение: `getClass() != o.getClass()` вместо `instanceof`, или использовать final-классы/композицию. В Kotlin data class генерирует equals по primary constructor — той же проблемы нет, если использовать data class.

### Kotlin: где полиморфизм сильнее
- Sealed class/интерфейсы + `when` (exhaustive) — compile-time безопасность вместо visitor.
- Extension functions — «статический полиморфизм» без наследования.
- Высшие функции: поведение передаётся параметром, иерархии классов не нужны.
