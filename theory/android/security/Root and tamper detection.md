### Root detection
Цель: определить модифицированную ОС и ограничить критичный функционал (или заблокировать вход, как в банковских приложениях).

**Признаки root:**
- Наличие su-бинарника и пакетов: `/system/bin/su`, `/system/xbin/su`, `com.noshufou.android.su`, `eu.chainfire.supersu`, `com.topjohnwu.magisk`.
- `Build.TAGS` содержит `test-keys` (не всегда).
- Разрешённая запись в системные пути: попытка создать файл в `/system`, `/data/data` чужих пакетов.
- Запущенные процессы/порты, свойственные root-инструментам.
- Флаги `ro.debuggable`, `ro.secure`.

**Ложные срабатывания:** эмуляторы, некоторые китайские прошивки, корпоративные устройства — поэтому обычно «подозреваем по совокупности признаков» + даём пользователю объяснение, а не безусловный бан.

**Инструменты:** SafetyNet Attestation (устарел, заменён) -> **Play Integrity API** — серверная проверка целостности устройства и приложения (проверяет: `MEETS_DEVICE_INTEGRITY`, `MEETS_BASIC_INTEGRITY`, `MEETS_STRONG_INTEGRITY` на StrongBox-устройствах).

### Tamper detection (целостность APK)
- Проверка подписи: получить signing certificate через PackageManager и сравнить с известным хэшем.
- Play Integrity API также подтверждает, что приложение установлено из Google Play (app recognition verdict) и не модифицировано.
- Самопроверка целостности файлов (хэши dex/assets) — легко обходится, скорее маркер.

```kotlin
val info = context.packageManager.getPackageInfo(
    context.packageName,
    PackageManager.GET_SIGNING_CERTIFICATES
)
val sig = info.signingInfo.apkContentsSigners[0].toByteArray()
val digest = MessageDigest.getInstance("SHA-256").digest(sig)
// сравнить digest с эталоном
```

### Противодействие отладке и инструментам
- `android:debuggable="false"` в релизе (по умолчанию для release-build).
- Проверка наличия Frida/Xposed (порты 27042, процессы `frida-server`, классы `de.robv.android.xposed`).
- Обнаружение эмулятора (Build.MODEL/FINGERPRINT, телефония-сервисы, сенсоры).

### Важная оговорка
Вся клиентская проверка обходится патчингом APK (удаление проверки). Настоящая защита — **серверная**: Play Integrity с серверной валидацией (nonce, anti-replay), сервер не доверяет клиенту, device binding токенов. Клиентские проверки — фильтр от массовой аудитории.

### Стандартные вопросы
- «Чем root detection отличается от SafetyNet/Play Integrity?» — локальные эвристики vs серверная криптографическая аттестация Google.
- «Как обойдут вашу проверку?» — патчинг, Frida-hook, эмулятор. Ответ: серверная верификация Play Integrity + минимизация ценности локально хранимых данных.
