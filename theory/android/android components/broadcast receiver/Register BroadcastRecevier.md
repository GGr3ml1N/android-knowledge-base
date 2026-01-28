_BroadcastReceiver_ можно зарегистрировать статически и динамически.

_Статическая регистрация_ – это добавление элемента \<receiver\> в AndroidManifest.
```
<receiver android:name=".MyBroadcastReceiver">  
    <intent-filter>  
        <action android:name="android.intent.action.LOCALE_CHANGED"/>  
        <action android:name="android.intent.action.BOOT_COMPLETED"/>  
    </intent-filter>  
</receiver>
```
Элемент \<intent-filter\> объявляет действия (\<action\>) на которые реагирует BroadcastReceiver. Система регистрирует ресиверы, прописанные в манифесте и вызывает их независимо от того запущено приложение или нет.

Динамическая регистрация – это регистрация на контексте в коде приложения. Выполняется методом **context.registerReceiver(receiver: BroadcastReceiver, intentFilter: IntentFilter)**. Этот метод принимает параметром объект класса IntentFilter, который определяет на какие действия будет реагировать зарегистрированный ресивер.
```
val receiver: BroadcastReceiver = MyBroadcastReceiver()  
val filter = IntentFilter().apply {  
    addAction(ConnectivityManager.CONNECTIVITY_ACTION)  
    addAction(Intent.LOCALE_CHANGED)  
    addAction("my_custom_action")  
}  
context.registerReceiver(receiver, filter)
```
Для разрегистрации динамических ресиверов используется метод **context.unregisterReceiver(receiver: BroadcastReceiver)**.

Ресиверы, зарегистрированные динамически, живут не дольше чем объект context, на котором они зарегистрированы. Если метод registerReceiver() вызывается на активити, то ресивер будет получать события, пока система не уничтожит активити. Если вызвать registerReceiver() на application context, то ресивер останется зарегистрированным, пока запущено приложение. Пост о различии activity и application контекстов.