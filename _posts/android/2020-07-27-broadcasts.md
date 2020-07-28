---
title: Broadcasts
author: Leslie M.
date: "2020-07-27 18:46:00 +0800"
categories: [Android, Theory]
tags: [android, theory, broadcasts, broadcast receiver]
---

Broadcasts - это широковещательные сообщения, которые отправляются, когда происходит определённое событие. Приложения могут отправлять их сами, либо получать сообщения, отправляемые системой Android и другими приложениями. Когда происходит трансляция сообщения, система автоматически будет направлять его всем, кто на него подписался.

***

## Системные сообщения

Системные сообщения отправляются автоматически, когда происходят различные системные события: включение системы или выход из режима полёта. Такие сообщения отправляются всем приложениям, которые подписаны на их получение.

Полный список системных событий можно посмотреть в файле `BROADCAST_ACTION.TXT` в Android SDK.

***

## Получение сообщений

Приложения могут получать сообщения двумя способами: объявив в манифесте приёмник широковещательных сообщений (broadcast receiver), либо зарегистрировать его в контексте.


### Объявление в манифесте

Если приёмник широковещательных сообщений будет объявлен в манифесте, то система запустит приложение при отправке сообщения.

Алгоритм объявления приёмника состоит из двух шагов:
- Создать класс, расширяющий базовый класс `BroadcastReceiver` и реализовать в нём метод обратного вызова `onReceive()`.

  ```
  class MyReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        val msg = intent.getStringExtra("msg")
        Toast.makeText(context, "New message received: $msg", Toast.LENGTH_LONG).show()
    }
  }
  ```

- Добавить элемент [`<receiver>`](https://bimlibik.github.io/posts/manifest-file/#receiver) в манифест. При этом с помощью [intent-фильтров](https://bimlibik.github.io/posts/manifest-file/#intent-filter) нужно объявить события, на которые подписывается этот приёмник. Они могут быть как системные, так и пользовательские (как в данном случае).

    ```
    <receiver android:name=".MyBroadcastReceiver"  android:exported="true">
      <intent-filter>
          <action android:name="com.foxy.NEW_MSG"/>
      </intent-filter>
    </receiver>
    ```


### Регистрация в контексте

Алгоритм объявления приёмника состоит из трёх шагов:
- Создать класс, расширяющий базовый класс `BroadcastReceiver` и реализовать в нём метод обратного вызова `onReceive()`. Создать экземпляр этого класса.

  ```
  class MyReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        StringBuilder().apply {
            append("Action: ${intent.action}\n")
            append("URI: ${intent.toUri(Intent.URI_INTENT_SCHEME)}\n")
            toString().also { log ->
                Toast.makeText(context, log, Toast.LENGTH_LONG).show()
            }
        }
    }
  }

  ...

  val br: BroadcastReceiver = MyReceiver()
  ```

- Создать intent-фильтр и зарегистрировать приёмник, вызвав метод `registerReceiver(BroadcastReceiver, IntentFilter)`.

  ```
  val filter = IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION).apply {
    addAction(Intent.ACTION_AIRPLANE_MODE_CHANGED)
  }

  registerReceiver(br, filter)
  ```

- Отменить регистрацию приёмника при помощи метода `unregisterReceiver(android.content.BroadcastReceiver)`.

Такие приёмники будут работать до тех пор, пока существует контекст, в котором они были зарегистрированы. Например, если приёмник был зарегистрирован в контексте активити, то он будет получать сообщения, пока активити не будет уничтожена. Если приёмник зарегистрирован в контексте приложения, то он будет получать сообщения, пока работает приложение.

Чтобы лишний раз не тратить ценные ресурсы, следует отменять регистрацию приёмника, когда в нём больше нет необходимости, либо когда контекст, в котором он был зарегистрирован, перестаёт существовать.

Также важно и то, в каком месте отменять регистрацию. Например, если приёмник был зарегистрирован в методе активити `onCreate()`, то отменять регистрацию следует в методе `onDestroy()`, чтобы предотвратить утечку приёмника из контекста. Если же приёмник был зарегистрирован в методе `onResume()`, то отменять регистрацию следует в методе `onPause()`, чтобы не получать сообщения пока активити "приостановлена".


### Общий принцип работы

Все широковещательные сообщения помещаются в `Intent`, в котором параметр `action` отвечает за произошедшее событие. Также он может включать какие-либо дополнительные данные, в зависимости от типа события.

Сообщение получают все приложения, которые на него подписались. Когда сообщение получает непосредственно приёмник, система вызывает метод `onReceive()` и передаёт в него `Intent`, который содержит сообщение.

В период выполнения метода `onReceive()` приёмник считается процессом переднего плана, поэтому система будет поддерживать выполнение этого процесса. Но как только ваш код вернётся из `onReceive()`, приёмник перестаёт быть активным, то есть его приоритет становится низким и система может его уничтожить в любое время.

По этой причине не следует запускать длительные фоновые потоки из приёмника: они, как и приёмник, для системы являются неактивными и уничтожаются вместе с ним.

Есть несколько способов, чтобы этого избежать:
- [`goAsync()`](https://developer.android.com/reference/android/content/BroadcastReceiver#goAsync() "developer.android.com"). Как правило приёмники могут работать до 10 секунд. Если же требуется немного больше времени, но не более 30 секунд, то следует использовать `goAsync()`;
- [`JobScheduler`](https://developer.android.com/reference/android/app/job/JobScheduler "developer.android.com"), [`Service`](https://developer.android.com/reference/android/app/Service "developer.android.com") или [`JobIntentService`](https://developer.android.com/reference/androidx/core/app/JobIntentService "developer.android.com"). Данные системные средства должны использоваться, когда требуется выполнить более длительную работу.

***

## Отправка сообщений

В основном смысл отправки сообщений из вашего приложения заключается в том, чтобы пока пользователь совершает какие-либо действия, обрабатывать их в другой части вашего приложения для выполнения определенной логики.

Отправлять сообщения можно тремя способами:
- `sendOrderedBroadcast(Intent, String)` - отправляет сообщение только одному приёмнику за раз. При этом приёмник может передать результат следующему приёмнику или остановить распространение сообщения. Это возможно благодаря тому, что каждый приёмник работает по очереди, а управлять очерёдностью можно с помощью атрибута [`android:priority`](https://bimlibik.github.io/posts/manifest-file/#intent-filter) элемента `<intent-filter>`. Приёмники с одинаковым приоритетом будут работать в произвольном порядке.

  Вторым параметром данного метода являются разрешения, которые приёмник должен иметь для получения сообщения. Может равняться `null` - это означает, что разрешение не требуется.

  ```
  val requiredPermission = "com.example.MY_BROADCAST_PERMISSION"
  Intent().also { intent ->
    intent.action = "com.example.MY_NOTIFICATION"
    intent.putExtra("msg","Hi! I'm message.")
    sendOrderedBroadcast(intent, requiredPermission)
    // sendOrderedBroadcast(intent, null)  
  }
  ```

- `sendBroadcast(Intent)` - отправляет сообщение всем приёмникам в произвольном порядке. Этот способ также называют "нормальной трансляцией сообщений". Несмотря на свою эффективность, имеет некоторые недостатки по сравнению с первым способом: приёмники не могут передавать и считывать результаты других приёмников, а также прерывать трансляцию сообщений.

  ```
  Intent().also { intent ->
    intent.action = "com.example.MY_NOTIFICATION"
    intent.putExtra("msg","Hi! I'm message.")
    sendBroadcast(intent)
  }
  ```

- `LocalBroadcastManager.sendBroadcast` - отправляет сообщения только тем приёмникам, которые определены в том же приложении, что и отправитель. При использовании данного способа не нужно будет беспокоиться, что стороннее приложение поймает ваше сообщение.

  Для использования этого метода нужно зарегистрировать приёмник с помощью `LocalBroadcastManager`.

  ```
  LocalBroadcastManager.getInstance(this).registerReceiver(
    object : BroadcastReceiver() {
      override fun onReceive(context: Context?, intent: Intent?) {
        val msg = intent?.getStringExtra("msg")
        Toast.makeText(context, "New message received: $msg", Toast.LENGTH_LONG).show()
      }

    },
    IntentFilter("com.example.MY_NOTIFICATION")
  )
  ```

  И отправить сообщение:

  ```
  Intent().also { intent ->
    intent.action = "com.example.MY_NOTIFICATION"
    intent.putExtra("msg", "Hi! I'm message.")
    LocalBroadcastManager.getInstance(this).sendBroadcast(intent)
  }
  ```

  Если `LocalBroadcastManager` не импортируется, то следует добавить зависимость:

  ```
  implementation 'androidx.localbroadcastmanager:localbroadcastmanager:1.0.0'
  ```

  В версии [`1.1.0-alpha01`](https://developer.android.com/jetpack/androidx/releases/localbroadcastmanager) Google признали `LocalBroadcastManager` устаревшим (deprecated).

***

## Установка разрешений

Разрешения позволяют ограничить рассылку сообщений определённым набором приложений. Ограничения можно добавить как к отправителю, так и к получателю сообщений.


### Отправка сообщений с разрешениями

Когда вы отправляете сообщение с помощью методов `sendBroadcast(Intent, String)` или `sendOrderedBroadcast(Intent, String)`, то в качестве второго параметра им можно передать разрешение. Тогда это сообщение дойдёт только до тех приёмников, у которых в манифесте указано это разрешение. При этом разрешение может быть как системным, так и пользовательским.

Например, при отправке сообщения было указано системное разрешение:

```
sendBroadcast(intent, Manifest.permission.SEND_SMS)
```

Это сообщение получат только те, у кого в манифесте есть такая строчка:

```
<uses-permission android:name="android.permission.SEND_SMS"/>
```

Пользовательские разрешения добавляются в манифест при помощи элемента [`<permission>`](https://bimlibik.github.io/posts/manifest-file/#permission).


### Получение сообщений с разрешением

При регистрации приёмника в манифесте или с помощью метода `registerReceiver()` можно задать разрешение. Тогда такому приёмнику смогут отправлять сообщения только такие отправители, у которых в манифесте указано это разрешение.

Допустим, у вас в манифесте объявлен приёмник с разрешением:

```
<receiver android:name=".MyBroadcastReceiver"
          android:permission="android.permission.SEND_SMS">
    <intent-filter>
        <action android:name="android.intent.action.AIRPLANE_MODE"/>
    </intent-filter>
</receiver>
```

Или вы зарегистрировали приёмник с разрешением в контексте:

```
var filter = IntentFilter(Intent.ACTION_AIRPLANE_MODE_CHANGED)
registerReceiver(receiver, filter, Manifest.permission.SEND_SMS, null )
```

Отправлять сообщения подобным приёмникам смогут такие приложения, у которых в манифесте присутствует строчка:

```
<uses-permission android:name="android.permission.SEND_SMS"/>
```

***

## Полезные ссылки

[Broadcasts overview](https://developer.android.com/guide/components/broadcasts) - официальная документация про широковещательные сообщения.  
[Implicit Broadcast Exceptions](https://developer.android.com/guide/components/broadcast-exceptions).  
