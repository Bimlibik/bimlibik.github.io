---
title: Реактивное программирование. ReactiveX
author: Leslie M.
date: "2021-04-20 20:05"
categories: [Libraries, ReactiveX]
tags: [reactive programming, reactive, reactivex, rxjava, rxkotlin, rxandroid]
---

Реактивное программирование - это программирование, основанное на асинхронных потоках данных и на распространении изменений. Под **потоком** здесь понимается массив данных, отсортированных по времени, который может сообщать, что данные изменились. Потоки могут транслировать данные или подписываться на них. В течение своего жизненного цикла потоки могут транслировать три сигнала: данные, ошибка, завершение.

Можно сказать, что смысл реактивного программирования заключается в том, чтобы **реактивно** реагировать на какие-либо события, при необходимости преобразовывать их и использовать / распространять результат. При этом каждая такая задача может выполняться в собственном потоке, несколько задач может выполняться одновременно. С помощью этого подхода мы можем избежать блокировки основного потока приложения.

Самая популярная реализация реактивного подхода - это библиотеки [**ReactiveX**][reactivex]. Библиотек много, каждая из них написана для использования с конкретным языком программирования. Например, библиотека [RxJava][rxjava-github] может быть использована для написания кода на Java / Kotlin.

***

## Общая концепция ReactiveX

**ReactiveX** - это реализация принципов реактивного программирования для создания асинхронных программ, основанных на событиях, путем наблюдения за потоками/последовательностями.

Основана библиотека на паттерне проектирования **Observer** (или Наблюдатель), который поддерживает поток данных и позволяет добавлять к нему различные операторы (для трансформации, фильтрации данных итд.). Благодаря этому мы можем абстрагироваться от таких вещей как потоки (threads) и их безопасность (thread-safety), синхронизация, неблокирующий ввод/вывод.

***

## Ключевые типы

Rx базируется на двух фундаментальных типах - **Observable** и **Observer**.

### Observable

`Observable` - это источник данных, который может сообщать три вида событий:
- Данные.
- Сигнал о завершении. Означает, что данные больше не будут поступать.
- Ошибка.

Подписаться на источник данных можно с помощью метода `subscribe()`.

`Observable` бывают:
- `hot` - порождает данные постоянно, даже если на него никто не подписан. Таким образом существует зависимость от того, когда именно произошла подписка на такой источник данных, т.к. наблюдатели могут пропустить какую-то часть элементов. Наиболее распространённым примером являются события пользовательского интерфейса в Android (например, клики по кнопке или по экрану).
  Существует много способов реализации горячих `Observable`. Один из них - `Subjects`. Также можно реализовать при помощи `ConnectableObservable`.
- `cold` - порождает данные только если у него есть хотя бы один подписчик. У каждого наблюдателя будет свой набор элементов. Идея здесь в том, что данные или операция _воспроизводятся_ для каждого наблюдателя.

### Observer

`Observer` - это потребитель данных. У него есть методы, которые вызываются в зависимости от поступившего события от `Observable`:
- `onNext()` - вызывается для каждого элемента из потока данных.
- `onComplete()` - вызывается, если поток завершён и данные больше не будут поступать.
- `onError()` - вызывается, если произошла ошибка.


### Реализация Observable и Observer

Чтобы подписаться на `Observable`, совсем нет необходимости в реализации `Observer`. Существуют различные перегрузки метода `subscribe()`, которые в качестве аргументов принимают функции `onNext()`, `onComplete()`, `onSubscribe()` и `onError()`. При этом можно предоставить их все, либо только часть из них.

### Отмена подписки

При вызове метода `Observable.subscribe()` возвращается объект класса `Disposable`. Он представляет собой связь между вашими `Observable` и `Observer`. В дальнейшем его можно использовать для отмены подписки - `disposable.dispose()`.

При отмене подписки останавливается вся цепочка вне зависимости от того, какой именно код сейчас выполняется.

***

## Источники данных

### Observable

`Observable` - наиболее универсальный источник данных. Умеет и генерировать данные, и не производить их вовсе. Если не подходит ни один из других источников данных, то смело использовать его.

Недостаток: не умеет обрабатывать [**backpressure**][backpressure-bimlibik].

При подписке можно реализовать метода:
- `onNext()` - вызывается при поступление элемента из потока данных.
- `onError()` - уведомление об ошибке.
- `onComplete()` - уведомление о завершении.

### ConnectableObservable

`ConnectableObservable` - начинает выдавать данные в момент вызова метода `connect()`. Сделано это для того, чтобы несколько наблюдателей могли обозревать один поток событий, не перезапуская его при каждой подписке.

### Flowable

`Flowable` - источник, предоставляющий дополнительные операторы для обработки [backpressure][backpressure-bimlibik]. Когда требуется обрабатывать более 10000 событий, происходящих быстро одно за другим, рекомендуется использовать `Flowable` вместо `Observable`.

### ConnectableFlowable

`ConnectableFlowable` - источник, который открывает те же возможности, что и `ConnectableObservable`, т.е. начинает выдавать данные в момент вызова метода `connect()`. Но при этом имеет преимущества `Flowable`.

### Single

`Single` - это `Observable`, который генерирует только один элемент или выдаёт ошибку.

При подписке можно реализовать методы:
- `onSuccess()` - возвращает результат.
- `onError()` - возвращает ошибку.

Пример использования:
- одноразовый сетевой запрос.

Преобразование:
- `toObservable()` - преобразовывает в `Observable`.
- `singleOrError()` - преобразовывает из `Observable`. Если в потоке данных более одного элемента, то будет выброшено исключение.

### Maybe

`Maybe` - источник, который либо генерирует один элемент, либо ничего не генерирует. Название говорит само за себе, этот источник как бы ничего вам не обещает: будут данные - передам, не будут - не передам. При этом в случае отсутствия данных, не будет выброшена ошибка (в отличие от `Single`).

При подписке можно реализовать методы:
- `onSuccess()` - возвращает результат.
- `onComplete()` - уведомляет об отсутствии элементов.
- `onError()` - возвращает ошибку.

При этом методы `onSuccess()` и `onComplete()` - взаимоисключающие, т.е. в случае вызова первого можно не ждать вызова второго.

Преобразование:
- `toObservable()` - преобразовывает в `Observable`.
- Нет простого способа преобразовать `Observable` в `Maybe`. Рекомендуют сначала преобразовать в `Single`, а затем в `Maybe` при помощи `toMaybe()`.

### Completable

`Completable` - операция, которая либо может быть выполнена, либо нет. Полезно, когда вас интересует только то, что операция выполняется правильно и вам не нужно отображать результат или какие-либо данные.

`Completable` не может быть создан при помощи метода `just()`.

При подписке можно реализовать методы:
- `onComplete()` - уведомляет о том, что операция прошла успешно.
- `onError()` - уведомляет об ошибке.

Преобразование:
- `toObservable()` - преобразовывает в `Observable`.
- `toCompletable()` - deprecated. Рекомендуют сначала преобразовать в `Single`, а затем использовать `ignoreElement()`.

***

## Subject

`Subject` - это класс, который может быть и источником, и наблюдателем. Это позволяет использовать его, например, в разного рода контроллерах, которые будут отдавать его наружу в виде `Observable` и внутри оповещать как `Observer`.

У этого класса есть несколько реализаций.

### AsyncSubject / AsyncProcessor

<!-- ![Async Subject](/assets/img/posts/reactivex/async-subject.png) -->

`AsyncSubject / AsyncProcessor` - держит последнее событие до корректного завершения потока, после чего отдаёт его подписчикам. При возникновении ошибки никакие события проброшены не будут.

### PublishSubject / PublishProcessor

`PublishSubject / PublishProcessor` - пробрасывает приходящие в него события дальше, пока не поступит терминальный сигнал. После конца потока или ошибки он возвращает соответствующие события.

### BehaviorSubject / BehaviorProcessor

`BehaviorSubject / BehaviorProcessor` - работает аналогично `PublishSubject`, но при подписке возвращает последнее событие, если оно есть и если Subject не перешёл в терминальное состояние.

### ReplaySubject / ReplayProcessor

`ReplaySubject / ReplayProcessor` - возвращает не одно последнее событие, а сколько душе угодно. Если подписаться на завершённый `ReplaySubject`, то будут получены все накопленные данные.

### CompletableSubject, MaybeSubject и SingleSubject

`CompletableSubject`, `MaybeSubject` и `SingleSubject` работают аналогично `PublishSubject`, только рассчитаны на использование с `Completable`, `Maybe` и `Single` соответственно.

### UnicastSubject

`UnicastSubject` - это фактически `ReplaySubject`, который следит, чтобы у него был только один подписчик. Он выбрасывает `IllegalStateException` при попытке повторной подписки.

### MulticastProcessor

`MulticastProcessor` - работает по аналогии с `PublishProcessor`, за исключением одной небольшой особенности. Он умеет обрабатывать **backpressure** для входящего в него потока. `MulticastProcessor` позволяет задать размер буфера, в котором он будет предзапрашивать элементы из upstream для будущих подписчиков.

***

## Операторы

Операторы - это некий промежуточный шаг (после получения данных, но до получения их подписчиками), который может быть использован для трансформации данных. Есть множество стандартных [операторов][operators] (поставляемых вместе с библиотекой), но также при желании и умении можно написать [собственные][your-own-operators].

Операторы позволяют делать с потоком данных всё, что угодно.

Несколько фактов об операторах:
- Операторы, привязанные к `Observable`, будут вести себя как его `Observer`.
- Этот промежуточный `Observer` может работать с каждым элементов из источника.
- Порядок, в котором вызываются операторы, имеет значение.
- Операторы также являются `Observable`, на который можно подписаться.

### Marble диаграммы

Прежде чем изучать операторы следует разобраться с такой штукой как Marble диаграммы, т.к. они будут часто встречаться и в документации Rx, и в различных статьях о нём.

Marble диаграммы - визуально передают то, что происходит с входными данными после прохождения через оператор. То есть по сути они просто визуально передают смысл того или иного оператора.

Структура Marble диаграмм:

![Основная структура marble диаграмм](/assets/img/posts/reactivex/marble-diagrams.png)

Пояснения:
- Ось X - это время.
- Ось Y - слой, который показывает трансформацию входных данных.
- Круглые точки - это элемент из потока данных.
- Треугольные значки - обрабатываемые элементы.
- Если произошла ошибка, то она отмечается крестом - `Х`.
- Если событие завершено успешно, то это отмечается прямой линией - `|`.

### Создание Observable или Observable Factories

Методы, с помощью которых создаются `Observable` также считаются операторами, поскольку они преобразуют входные данные в `Observable`.

- `just()` - наиболее простой оператор, чаще всего встречается в различных примерах по демонстрации Rx. Он обёртывает введённые данные в `Observable` и возвращает их как элементы. Может принимать от 1 до 10 однотипных элементов. Если по какой-то причине во входном выражении произошло исключение, то оно не будет передано в метод `onError()`. Вместо этого вы можете столкнуться с `RuntimeException`.
- `create()` - создаёт `Observable` с нуля, в качестве аргументов принимает методы наблюдателя.
- `start()` - создаёт `Observable`, который который испускает возвращаемое значение функции.
- `from()` - это целая группа операторов, которая преобразовывает какой-либо объект или структуру данных в `Observable` и по очереди рассылает их наблюдателям. Виды:
  - `fromIterable()` - этот оператор полезен в тех случаях, когда на вход передаётся коллекция и при этом нужно обработать каждый элемент этой коллекции.
  - `fromArray()` - похож на `fromIterable()`, но вместо этого принимает переменное количество аргументов. Он используется только при использовании более чем с одним параметром.
  - `fromCallable()` - принимает в качестве входных данных экземпляр `Callable<V>`, который будет вызван только в случае подписки. Любое обнаруженное исключение будет передано в метод `onError()`.
  - `fromFuture()` - принимает в качестве входных данных экземпляр `Future`.
  - `fromPublisher()` - обычно используется для потенциально неограниченного потока данных
- `empty()` - ничего не отправляет, вызывает уведомление о завершении операции.
- `never()` - ничего не отправляет, даже уведомления о завершении операции.
- `error()` - ничего не отправляет, вызывает уведомление об ошибке.
- `range()` - на вход принимает два числа - `start` и `count` - и генерирует последовательность целых чисел (int), начиная со `start` и заканчивая `start + count - 1`. Можно использовать `rangeLong()` для больших чисел.
- `timer()` - позволяет указать время задержки перед отправкой события. Также отправляет один `Long` со значение `0L` перед завершением.
- `interval()` - похож на оператор `timer()`, но с передачей последовательности целых чисел, разделённых заданным временным интервалом.
- `intervalRange()` - сочетает в себе операторы `range()` и `interval()`. С его помощью можно генерировать инкрементные значения в пределах диапазона с заданным временным интервалом.
- `repeat()` - повторяет входные данные заданное количество раз. Если не задать количество повторений, то данные будут повторяться `Long.MAX_VALUE` раз.
- `repeatUntil()` - повторяет входные данные до тех пор, пока не будет выполнено заданное условие.
- `repeatWhen()`
- `defer()` - работает по тому же принципу, что и `fromCallable()`, но более подробный. Не создаёт `Observable` до тех пор, пока не появится наблюдатель. При этом создаёт новый `Observable` для каждого наблюдателя.

### Оператотры трансформации

- `map()` - преобразует один элемент данных в другой (например, с помощью него можно трансформировать строку, добавив к ней какое-либо значение). Также может преобразовать один тип данных в другой (например, `String` в `Int`).
- `flatMap()` - принимает данные от одного `Observable`, применяя к каждому его элементу переданную вами функцию, которая возвращает новый `Observable`. Т.е. подменяет один `Observable` на другой. Новый `Observable` - это то, что в итоге увидит `Observer`. При этом не заботится о том, в каком порядке эти данные придут подписчику (могут прийти в ином порядке, чем при изначальном создании данных).
- `switchMap()` -
- `concatMap()` - поддерживает порядок элементов и ожидает, пока текущий `Observable` завершит свою работу, прежде чем передать следующий. Подходит, если вы хотите сохранить порядок выполнения.
- `buffer()` - периодически собирает элементы `Observable` в `Bundle`, после чего испускает эти `Bundle`'ы, вместо того, чтобы передавать элементы по одному.
- `groupBy()` - группирует `Observable` на набор `Observable`, каждый из которых испускает отдельную группу элементов из исходного `Observable`.
- `scan()` - (пример использования - поиск факториала)

### Операторы фильтрации

- `debounce()` - добавляет задержку перед отправкой данных.
- `distinct()` - убирает дубли.
- `elementAt()` - принимает индекс, возвращает один элемент по заданному индексу.
- `filter()` - позволяет фильтровать данные. Достаточно передать условие и тогда будут передаваться только те данные, которые прошли заданную проверку.
- `first()` - отправляет только первый элемент из `Observable` или первый элемент, который соответствует заданному условию.
- `ignoreElemnts()` - игнорирует все элементы, но отображает уведомление о том, что всё успешно прошло.
- `last()` - отправляет только последний элемент из `Observable`.
- `sample()` - отправляет только последний элемент из `Observable` с заданным временным интервалом.
- `skip()` - с начала потока будет исключено заданное количество элементов.
- `skipLast()` - с конца потока будет исключено заданное количество элементов.
- `take()` - позволяет задать количество элементов, которые будут переданы подписчикам. Если фактически поступило меньшее количество элементов, чем заданное этим оператором, то `take()` просто завершит свою работу раньше.
- `takeLast()` - отправляет подписчикам заданное количество элементов из конца потока.

### Операторы комбинирования

- `combineLatest()` - объединяет последние элементы из нескольких `Observable` и возвращает полученный результат.
- `join()` - объединяет элементы, излучаемые двумя `Observable`, всякий раз, когда элемент из одного `Observable` испускается в течение временного окна, определённого элементом, испускаемым другим `Observable`.
- `merge()` - слияние двух `Observable` в один. Не заботится о том, в каком порядке выдаются данные из этих потоков.
- `startWith()` - испускать указанную последовательность элементов перед тем, как начать испускать элементы из источника `Observable`.
- `switch()` - преобразует `Observable`, который испускает `Observables` в один `Observable`. Он отправляет элементы, испускаемые самым последним из этих `Observable`.
  - `switchOnNext()` - отправляет данные из одного источника данных, пока не появятся данные из более приоритетного источника.
- `zip()` - объединяет элементы нескольких `Observable` вместе и отправляет отдельно каждую получившуюся комбинацию.
- `zipWith()` - работа с двумя запросами как с одним. Если для кого-то пары не нашлось - будет пропущен. Выдаёт результаты по порядку.
- `concat()` - слияние двух потоков, выдача сначала данных из первого потока (по порядку), затем из второго (тоже по порядку).

### Операторы для обработки ошибок

- `catch()` - восстанавливает поток после получения уведомления об ошибке в `onError()`.
- `retry()` - после получения уведомления об ошибке в `onError()` повторно подписывается на поток в надежде, что он завершится без ошибок.

### Вспомогательные операторы

- `delay()` - добавляет задержку перед отправкой данных.
- `do` - это своего рода набор операторов, которые можно использовать для получения уведомлений перед отправкой их в соответствующий метод `Observer`'а.
  - `doOnNext()` - позволяет добавить какое-либо дополнительное действие, которое будет применяться к каждому новому элементу.
- `observeOn()` - указывает на scheduler, в котором `Observer` будет наблюдать за `Observable`.
- `subscribeOn()` - указывает на то, в каком scheduler начнёт работать  `Observable` и операторы. При этом не важно, в каком месте всей цепочки он был вызван.
- `timeInterval()` - испускает то, сколько времени прошло с момента поступления предыдущего элемента.
- `timeout()` - позволяет указать время, за которое должны поступить данные. Если не поступили - выбрасывается `TimeoutException`.
- `timestamp()` - прикрепляет временную метку каждому элементу.
- `using()` - создаёт disposable, срок жизни которого аналогичен `Observable`.

### Условные и логические операторы

- `all()` - позволяет проверить, что все элементы, испускаемые `Observable`, соответствуют заданным критериям.
- `contains()` - позволяет проверить, если ли в `Observable` конкретный элемент.
- `defaultIfEmpty()` - испускает либо элементы из `Observable`, либо элемент по умолчанию, если `Observable` пуст.

### Математические операторы

- `count()` - количество элементов в `Observable`.

### Остальные операторы

[Полный список операторов][operators].

[Реализация собственных операторов][your-own-operators].

***

## Backpressure

**Backpressure** — ситуация, когда новые события поступают существенно быстрее, чем успевают обрабатываться, и поэтому начинают скапливаться в буфере, переполняя его. Это может привести к неприятностям вроде `OutOfMemoryError`. Подробнее можно посмотреть [тут][backpressure-github].

***

## Rx vs Android

При написании приложений под Android могут быть использованы библиотеки:
- `RxJava` - основа.
- `RxAndroid` - поверх RxJava добавлены специфичные для платформы Android классы. Может быть использована совместно с RxJava. Что в нём специфичного:
  - Класс `AndroidSchedulers` - предоставляет готовые Schedulers для потоков, специфичных для Android.
  - Класс `AndroidObservable` - предоставляющий возможности по работе с жизненным циклом некоторых классов из Android SDK. В нём есть операторы:
    - `bindActivity()` и `bindFragment()` - останавливают поток данных, если ваши `Activity` или `Fragment` уничтожаются/уничтожены.
    - `fromBroadcast()` - позволяет создать `Observable`,  который работает как `BroadcastReceiver`.
  - Класс `ViewObservable` - добавляет привязки к `View`. Операторы:
    - `clicks()` - для получения уведомлений всякий раз, когда происходит нажатие на `View`.
    - `text()` - срабатывает всякий раз, когда `TextView` изменяет своё содержимое.
- `RxKotlin` - это обёртка RxJava на языке Kotlin, с более лаконичными решениями, которые позволяют сократить количество кода. Может быть использована самостоятельно, без дополнительных зависимостей в виде RxJava, но RxAndroid при необходимости придётся подключить.
- `RxBinding` - библиотека, которая осуществляет привязку View и тем самым превращает View в источник данных.

***

## Rx vs Retrofit

`Retrofit` поддерживает библиотеку `RxJava`, благодаря чему открываются некоторые возможности::
- Вместо использования `Callback` в ApiInterface можно использовать `Observable`.

  ```
  @GET("pictures")
  fun getPictures(
      @Query("query") query: String,
  ): Observable<PictureResponse>
  ```

  Но в таком случае потребуется адаптировать Rx типы под Retrofit. Осуществляется с помощью специального [адаптера][adapter-rxjava], созданного разработчиками Retrofit. Его необходимо добавить в зависимости, после чего достаточно дописать одну строчку кода при создании API клиента:

  ```
  retrofit = Retrofit.Builder()
    .baseUrl(BASE_URL)
    .addConverterFactory(...)
    .addCallAdapterFactory(RxJava3CallAdapterFactory.create()) // адаптер для Rx
    .build()
  ```

  Так же есть ещё одна [библиотека][adapter-akarnokd] с адаптером, но только для RxJava3. Принцип работы аналогичный.

- Комбинирование нескольких запросов вместе.

***

## Полезные ссылки

**Статьи на русском:**

[Реактивное программирование. Начало](https://medium.com/@oxmap/%D1%80%D0%B5%D0%B0%D0%BA%D1%82%D0%B8%D0%B2%D0%BD%D0%BE%D0%B5-%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%BD%D0%B0%D1%87%D0%B0%D0%BB%D0%BE-4ad548a7d41c) - немного про реактивное программирование в целом.  
[ReactiveX 2.0 с примерами, или грокаем реактивное программирование 2.0. Часть 1: Observable vs Flowable, Backpressure](https://habr.com/ru/post/336268/) - серия статей, написана на примерах из RxJava v.1.x, полезна для общего понимая.  
[Введение в RxJava: Почему Rx?](https://habr.com/ru/post/269417/) - серия статей. Является переводом [туториала][froussios] по RxJava Крисса Фруссиоса.  
[Справочник по источникам событий в Rx](https://habr.com/ru/company/funcorp/blog/459174/) - коротко и понятно об источниках данных.

**Статьи на английском:**

[RxJava Ninja: Introduction to Reactive Programming](https://medium.com/tompee/rxjava-ninja-introduction-to-reactive-programming-4b1e27b20576) - серия статей по RxJava.  

**Видео на русском:**

[RxJava - Observable, Flowable (часть 1)](https://www.youtube.com/watch?v=V-UkPijjJrk)  
[RxJava - Transformation, Filter (часть 2)](https://www.youtube.com/watch?v=Z0vB_TlvJJ4)  
[RxJava - Combination, Utility, Binding (часть 3)](https://www.youtube.com/watch?v=6DOPxgqgzkk)

**Примеры приложений:**

[MVVM with Hilt, RxJava 3, Retrofit, Room, Live Data and View Binding](https://medium.com/swlh/mvvm-with-hilt-rxjava-3-retrofit-room-live-data-and-view-binding-8da9bb1004bf) - пример на Java.  
[How to make complex requests simple with RxJava in Kotlin](https://medium.com/mindorks/how-to-make-complex-requests-simple-with-rxjava-in-kotlin-ccec004c5d10) - пример на Kotlin.






[reactivex]: http://reactivex.io/
[operators]: http://reactivex.io/documentation/operators.html
[your-own-operators]: https://github.com/ReactiveX/RxJava/wiki/Implementing-Your-Own-Operators
[rxjava-github]: https://github.com/ReactiveX/RxJava
[backpressure-github]: https://github.com/ReactiveX/RxJava/wiki/Backpressure-(2.0)
[backpressure-bimlibik]: https://bimlibik.github.io/posts/reactive-programming/#backpressure
[froussios]: https://github.com/Froussios/Intro-To-RxJava
[adapter-rxjava]: https://github.com/square/retrofit/tree/master/retrofit-adapters
[adapter-akarnokd]: https://github.com/akarnokd/RxJavaRetrofitAdapter