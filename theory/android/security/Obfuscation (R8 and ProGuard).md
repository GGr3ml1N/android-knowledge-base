### R8 vs ProGuard
- **ProGuard** — классическая Java-обфускация (переименование, удаление неиспользуемого кода).
- **R8** — современный дефолтный компилятор Android (с Android Gradle Plugin 3.4+). Делает всё то же: shrinking (удаление неиспользуемых классов/методов), obfuscation (переименование в `a.b.c`), optimization, плюс DEX-компиляция. ProGuard-правила совместимы.

### Как включить
```groovy
android {
    buildTypes {
        release {
            minifyEnabled true      // R8: shrink + obfuscate + optimize
            shrinkResources true    // удалить неиспользуемые ресурсы (требует minifyEnabled)
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

### ProGuard/R8 правила
```proguard
# Сохранить класс (например, модели JSON-парсера)
-keep class com.example.app.data.dto.** { *; }

# Сохранить сериализуемые поля (Gson/Moshi)
-keepclassmembers class * {
    @com.google.gson.annotations.SerializedName <fields>;
}

# Сохранить нативные методы
-keepclasseswithmembernames class * {
    native <methods>;
}
```
Типичный сценарий: рефлексия (Gson, Room, Parcelable, ViewBinding) — R8 не знает, что доступ идёт через имена, поэтому модели нужно исключать из обфускации.

### Mapping file
R8 генерирует `mapping.txt` — таблицу `оригинальное имя -> обфусцированное`. **Сохраняйте его для каждого релиза** (в AGP можно автоматически загружать в Play или CI-артефакты) — без него stacktrace из продакшена нечитаем (Crashlytics умеет автоматически деобфусцировать при загрузке mapping).

### Это не «защита от взлома»
Обфускация замедляет reverse engineering, но не останавливает: `a.b.c()` прочитают по контексту вызовов, строки в коде видны. Поэтому:
- Секреты всё равно нельзя класть в код (см. [[!Security]]).
- Для чувствительных строк иногда применяют дополнительное шифрование/разбиение, но это затруднение, а не защита.

### Стандартные вопросы на собесе
- Что ломается при minifyEnabled? — Рефлексия, сериализация JSON по именам полей, ViewBinding-generated код (обычно уже имеет правила), Crashlytics-деобфускация.
- Чем R8 отличается от ProGuard? — R8 компилирует сразу в DEX, однопроходный, быстрее, лучше оптимизации; ProGuard отдельный инструмент работающий с байткодом JVM.
- Где хранить mapping.txt? — CI-артефакты, Crashlytics upload, отдельный репозиторий с версионированием по build number.
