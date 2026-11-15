### Задача
HTTPS защищает от пассивного перехвата, но устройство доверяет любому сертификату, подписанному CA из системного хранилища. Злоумышленник, установивший свой CA на устройство (или использующий скомпрометированный CA), может выполнить MITM. **SSL Pinning** — доверять только конкретным сертификатам/публичным ключам сервера, игнорируя системное хранилище.

### OkHttp CertificatePinner
```kotlin
val pinner = CertificatePinner.Builder()
    .add("api.example.com", "sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=")
    .build()

val client = OkHttpClient.Builder()
    .certificatePinner(pinner)
    .build()
```
Отпечаток (pin) — SHA-256 хэш публичного ключа (SPKI). Получить: `openssl s_client -connect api.example.com:443 | openssl x509 -pubkey -noout | openssl pkey -pubin -outform der | openssl dgst -sha256 -binary | openssl enc -base64`.

### Pin чего: leaf vs intermediate
- **Pin на leaf-сертификат** (сам сервер) — максимум безопасности, но сертификат живёт ~1 год: при ротации приложение перестанет работать, пока не обновят.
- **Pin на intermediate/root CA** — переживает ротацию серверного сертификата, но доверяет любому серверу с этим CA.
- **Компромисс:** pin на intermediate + запасной pin на backup-ключ, или **динамический pinning** — пины прилетают с сервера и хранятся в приложении.

### Ошибка: pin без стратегии ротации
Если сертификат сменится, а пин захардкожен — приложение **умрёт для всех пользователей** до выпуска обновления. Поэтому:
1. Два пина: основной + запасной (backup key).
2. Динамический pinning (обновляемые пины).
3. Флаг на бэкенде/фиче-тоггл: отключить pinning удалённо при инциденте.

### Network Security Config (альтернатива для certificate pinning на уровне системы)
```xml
<!-- res/xml/network_security_config.xml -->
<network-security-config>
    <domain-config>
        <domain includeSubdomains="true">api.example.com</domain>
        <pin-set expiration="2027-01-01">
            <pin digest="SHA-256">AAAA...</pin>
            <pin digest="SHA-256">BBBB...</pin> <!-- backup -->
        </pin-set>
    </domain-config>
</network-security-config>
```
Применяется к WebView и всем сетевым стекам процесса, но менее гибок, чем OkHttp pinner.

### Побочный эффект: debugging
С включённым pinning Charles/mitmproxy перестают работать. Решение: отдельный debug-build type без pinning или `debugOverrides` в network security config (отладочные CA только в debug-сборках).

### Стандартные вопросы
- «Что будет при истечении сертификата?» — SSLPeerUnverifiedException, ошибка сети у пользователей; нужен backup pin или динамический pinning.
- «Можно ли обойти pinning?» — да, через Xposed/Frida-перехват в рантайме на рутованном устройстве. Pinning — против массового MITM, не против целевой атаки на конкретное устройство.
