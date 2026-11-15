### Что такое Android Keystore System
Системный сервис для хранения криптографических ключей в **отделённой зоне** (TEE — Trusted Execution Environment / StrongBox). Ключи не покидают хранилище: приложение выполняет криптооперации через API, а сами ключи неизвлекаемы. Защита на уровне: аппаратном (TEE/StrongBox) или на уровне ядра с привязкой к lockscreen-у пользователя.

### Генерация ключа
```kotlin
val keyGen = KeyGenParameterSpec.Builder(
    "app_master_key",
    KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT
)
    .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
    .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
    .setKeySize(256)
    // Опционально: ключ доступен только после разблокировки
    // .setUserAuthenticationRequired(true)
    // .setUserAuthenticationValidityDurationSeconds(300)
    .build()

KeyGenerator.getInstance(KeyProperties.KEY_ALGORITHM_AES, "AndroidKeyStore")
    .init(keyGen)
    .generateKey()
```

### Зачем это в приложении
Шифровать локально хранимые чувствительные данные: токены, ПИН-коды, данные для автозаполнения. Мастер-ключ в Keystore шифрует данные, а сами данные лежат в обычном файле/SharedPreferences.

### EncryptedSharedPreferences (Jetpack Security)
Готовая обёртка: SharedPreferences с прозрачным AES-шифрованием значений (и ключей), мастер-ключ — в Keystore.
```kotlin
val masterKey = MasterKey.Builder(context)
    .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
    .build()

val prefs = EncryptedSharedPreferences.create(
    context,
    "secrets",
    masterKey,
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
)
prefs.edit().putString("access_token", token).apply()
```
Важно: **на рутованном устройстве / с бэкапом Keystore может быть компрометация** — не считаем это абсолютной защитой. Также данные непереносимы между устройствами.

### Что делать с refresh-токеном (практика банков)
- Хранить в Keystore-зашифрованном хранилище.
- Дополнительно: привязка токена к устройству на бэкенде (device binding), fingerprint/PIN-подтверждение для критичных операций.
- Access-токен — в памяти процесса, с коротким TTL; refresh — в защищённом хранилище.

### Про бэкапы
`android:allowBackup="false"` (или правила backup rules) для БД/файлов с чувствительными данными — иначе `adb backup` или Google Backup может вытащить файлы на другое устройство.

### Стандартные вопросы
- «Чем Keystore отличается от EncryptedSharedPreferences?» — Keystore хранит КЛЮЧИ (и выполняет криптооперации), EncryptedSharedPreferences хранит ДАННЫЕ, шифруя их мастер-ключом из Keystore.
- «Что если устройство рутовано?» — Keystore на TEE/StrongBox всё ещё защищён аппаратно, но можно перехватить данные до/после расшифровки в памяти приложения. Поэтому root detection + device attestation (Play Integrity).
