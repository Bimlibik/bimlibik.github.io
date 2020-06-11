---
title: Kotlin. Отложенная и ленивая инициализация свойств
author: Leslie M.
date: "2020-06-09 15:40:00 +0800"
categories: [Kotlin, Basics]
tags: [kotlin, theory, basics, lateinit, lazy]
---

Разработчики Kotlin крайне серьёзно относятся к проверкам на **null**. Поэтому,
как правило, свойства, которые по логике программы должны хранить ненулевые
значения инициализируются в конструкторе.

Тем не менее бывают ситуации, когда такой подход не особо удобен. Например,
если вы хотите инициализировать свойства через внедрение зависимостей. Kotlin
предусматривает такую возможность и предлагает использовать отложенную (позднюю)
инициализацию. Осуществляется это с помощью модификатора **lateinit**.

```
class Person {
  lateinit var work: Work

  fun main(args: Array<String>) {
    work = Work()
  }

}
```

Модификатор **lateinit** говорит о том, что данная переменная будет
инициализирована позже. При этом инициализировать свойство можно из любого места,
откуда она видна.

Правила использования модификатора **lateinit**:
- используется **только** совместно с ключевым словом **var**;
- свойство может быть объявлено только внутри тела класса (не в основном конструкторе);
- тип свойства не может быть нулевым и примитивным;
- у свойства не должно быть пользовательских геттеров и сеттеров;
- с версии Kotlin 1.2 можно применять к свойствам верхнего уровня и локальным переменным.

Если обратиться к свойству с модификатором **lateinit** до того, как оно будет
проинициализировано, то получите ошибку, которая явно указывает, что свойство не
было определено:

```
lateinit property has not been initialized
```

В версии Kotlin 1.2 модификатор был улучшен. Теперь перед обращением к переменной
можно проверить была ли она инициализирована. Осуществляется это с помощью метода
`.isInitialized`. Данная функция вернет **true**, если переменная
инициализирована и **false**, если нет.

```
class Person {
  lateinit var work: Work

  fun main(args: Array<String>) {
    println(this::work.isInitialized)  // вернет false
    work = Work()
    println(this::work.isInitialized)  // вернет true
  }

}
```

***

Помимо отложенной инициализации в Kotlin существует ленивая инициализация
свойств. Такая инициализация осуществляется с помощью функции `lazy()`, которая
принимает лямбду, а возвращает экземпляр класса `Lazy<T>`. Данный объект
реализует **ленивое** вычисление значения свойства: при первом обращении к
свойству метод `get()` запускает лямбда-выражение (переданное `lazy()` в качестве
аргумента) и запоминает полученное значение, а последующие вызовы просто
возвращают запомненное значение.

```
class Person {
  val work: String by lazy {
    println("Start")
    "End"
  }

  fun main(args: Array<String>) {
    println(work)
    println(work)
  }

  // Код выведет:
  // Start
  // End
  // End
}
```

Ленивая инициализация может быть использована **только** совместно с ключевым
словом **val**.

Свойство, инициализированное подобным образом, называется _делегированным
свойством_. Потому что мы делегировали вычисление значения классу-делегату
`Lazy<T>`. Данный класс является частью стандартной библиотеки Kotlin и именно в
нем реализован get-метод вычисляющий и возвращающий значение.

По умолчанию вычисление ленивых свойств **синхронизировано**: значение
вычисляется только в одном потоке, а все остальные потоки могут видеть одно и то
же значение. Однако способом вычисления можно управлять. Для этого функции
`lazy()` нужно передать один из параметров:
- `LazyThreadSafetyMode.SYNCHRONIZED` - режим по умолчанию, потокобезопасный.
- `LazyThreadSafetyMode.PUBLICATION` - вычисление будет происходить в нескольких
  потоках, но вернётся то значение, которое будет вычислено первым.
- `LazyThreadSafetyMode.NONE` - использовать с осторожностью, не является
  потокобезопасным. Нужно быть уверенным, что вычисление будет происходить в
  одном потоке.