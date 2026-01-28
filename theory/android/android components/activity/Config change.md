При смене конфигурации активити уничтожается и создается заново. Вызываются коллбэки onPause(), onStop(), onSaveInstanceState(), onDestroy() – onCreate(), onStart(), onRestoreInstanceState(), onResume().

Чтобы сохранить состояние активити, вы должны переопределить метод onSaveInstanceState() и положить данные в Bundle.

При реинициализации активити, Bundle с сохраненным состоянием передается в onCreate() и в onRestoreInstanceState().

Система вызывает onSaveInstanceState() и onRestoreInstanceState() только в том случае, когда необходимо сохранить состояние, например при повороте экрана или при убийстве активити для освобождения памяти. Данные коллбэки не вызываются, если пользователь выходит из активити нажав Back или если активити убивается вызовом finish().

onSaveInstanceState() вызывается после onStop() на версии API ≥ 28. На API < 28 этот коллбэк вызывается перед onStop() и нет гарантий до или после onPause().

onRestoreInstanceState() вызывается после onStart().