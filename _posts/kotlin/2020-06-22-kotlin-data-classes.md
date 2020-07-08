---
title: Kotlin. Классы данных (Data classes)
author: Leslie M.
date: "2020-06-22 21:30:00 +0800"
categories: [Kotlin, Classes and Objects]
tags: [kotlin, theory, data classes]
---

В процессе разработки нам часто приходится создавать классы, предназначенные исключительно для хранения каких-либо данных. При этом, чтобы такой класс стал максимально удобным переопределяются методы `toString()`, `equels()` и `hashCode()`.

Обычно данные методы имеют одинаковую реализацию и чтобы каждый раз не писать один и тот же код можно просто отметить класс ключевым словом **data** - все необходимые методы будут сгенерированы автоматически. В Kotlin такие классы называются _классами данных_ (data classes).

Не каждый класс можно отметить ключевым словом **data**. Для этого он должен соответствовать определённым требованиям:
- В основном конструкторе должен быть как минимум один параметр.
- Все параметры основного конструктора должны быть отмечены ключевыми слова **val** или **var** (рекомендуется val).
- Классы данных не могут быть отмечены ключевыми словами **abstract**, **open**, **sealed**, **inner**.

***

## Переопределяемые функции

Возможно у кого-то возникнет вопрос: для чего вообще нужны методы `toString()`, `equels()` и `hashCode()`? В данном разделе остановимся на этом подробнее, а также опишем какие ещё функции в классах данных генерируются автоматически.

### `toString()`
Часто, особенно при отладке, возникает необходимость вывести в лог информацию об экземпляре класса. Если метод `toString()` не переопределён, то при обращении к экземпляру в лог будет выведена ссылка на него.

```
class Person(val name: String , val age: Int) {
  ...
}

...

val person = Person("Adam", 27)
println(person)

// Выведется в лог
Person@Se9f23b4
```

Это не очень информативно и вряд ли чем-то поможет. Чтобы исправить ситуацию достаточно переопределить метод `toString()` и указать в нём, что именно нужно выводить в лог при обращении к экземпляру класса.

```
class Person(val name: String , val age: Int) {
  override fun toString() = "Person(name = $name , age = $age )"
}

...

val person = Person("Adam", 27)
println(person)

// Выведется в лог
Person(name = Adam , age = 27)
```

Если же мы отметим класс ключевым словом **data**, метод `toString()` будет переопределён автоматически. При этом в лог будут выводиться все поля, указанные в конструкторе, в порядке их добавления.

```
data class Person(val name: String , val age: Int)

...

val person = Person("Adam", 27)
println(person)

// Выведется в лог
Person(name = Adam , age = 27)
```

### `equels()`
Иногда нам может потребоваться сравнить между собой два объекта таким образом, чтобы они считались равными, если содержат одни и те же данные.

```
val person1 = Person("Adam", 27)
val person2 = Person("Adam", 27)
println(person1 == person2)

// Выведется в лог
false
```

Объекты не равны, потому что по умолчанию сравниваются не данные, которые они хранят, а ссылки на объекты. Чтобы задать свой алгоритм сравнения переопределяется метод `equels()`.

```
class Person(val name: String , val age: Int) {
  override fun toString() = "Person(name = $name , age = $age )"

  override fun equals(other: Any?): Вооlean {
    if (other == null || other !is Person)
      return false
    return name == other.name && age == other.age
  }
}

...

val person1 = Person("Adam", 27)
val person2 = Person("Adam", 27)
println(person1 == person2)

// Выведется в лог
true
```

Если же мы отметим класс ключевым словом **data**, метод `equels()` будет переопределён автоматически. При этом работать будет точно также, как и в примере выше: будет проверять на равенство все значения, указанные в основном конструкторе.

```
data class Person(val name: String , val age: Int)

...

val person1 = Person("Adam", 27)
val person2 = Person("Adam", 27)
println(person1 == person2)

// Выведется в лог
true
```

Так как оператор `==` за кулисами вызывает метод `equels()`, для сравнения ссылок объектов используется оператор `===`.

```
data class Person(val name: String , val age: Int)

...

val person1 = Person("Adam", 27)
val person2 = Person("Adam", 27)
println(person1 === person2)

// Выведется в лог
false
```

### `hashCode()`
Экземпляр класса можно использовать как ключ в структурах данных на основе хэш-функций. Это возможно благодаря тому, что каждому объекту присваивается уникальный хэш-код, даже если значения этих объектов идентичны.

```
val hashSet = hashSetOf(Person("Adam", 27))
println(hashSet.contains(Person("Adam", 27)))

// Выведется в лог
false
```

Но по логике, если два объекта содержат одинаковые значения, значит и хэш-код у них должен быть одинаковым. Для этого и переопределяется метод `hashCode()`.

```
class Person(val name: String , val age: Int) {
  override fun toString() = "Person(name = $name , age = $age )"

  override fun equals(other: Any?): Вооlean {
    if (other == null || other !is Person)
      return false
    return name == other.name && age == other.age
  }

  override fun hashCode(): Int = name.hashCode() * 31 + age
}

...

val hashSet = hashSetOf(Person("Adam", 27))
println(hashSet.contains(Person("Adam", 27)))

// Выведется в лог
true
```

Обратите внимание, что метод `hashCode()` работает совместно с методом `equels()`. Это означает, что если переопределить метод `hashCode()` без переопределения метода `equels()`, каждому объекту будет присвоен уникальный хэш-код, даже если значения этих объектов равны. Связано это с тем, что перед присвоением хэш-кода происходит сравнение объектов. А без метода  `equels()` объекты сравниваются по их ссылкам, а не значениям.

Опять же, чтобы обо всём этом не думать, достаточно отметить класс ключевым словом **data** и метод `hashCode()` (и все остальные) будет переопределён автоматически. При этом работать будет точно также, как и в примере выше: будет возвращать значение, зависящее от хэш-кодов всех свойств, объявленных в основном конструкторе.

```
data class Person(val name: String , val age: Int)

...

val hashSet = hashSetOf(Person("Adam", 27))
println(hashSet.contains(Person("Adam", 27)))

// Выведется в лог
true
```

### `copy()`
Ещё один метод, который генерируется автоматически для всех классов данных. Он позволяет копировать экземпляры класса, изменяя значения некоторых свойств.

Разработчики Kotlin пытаются заложить нам в голову идею о том, что вместо модификации объекта лучше создать новый объект. Поэтому и была добавлена данная функция - упростить создание новых объектов.

На практике это может выглядеть примерно следующим образом:

```
data class Person(val name: String , val age: Int)

...

val person = Person("Adam", 27)
println(person.copy(age = 28))

// Выведется в лог
Person(name=Adam, age=28)
```

***

## Мультидекларации

_Мультидекларации_ (destructuring declarations) - это особенность, характерная для классов данных, которая позволяет распаковать объект и использовать его значения для инициализации сразу нескольких переменных.

```
val person = Person("Adam", 27)
val(name, age) = person

println(name)

// Выведется в лог
Adam
```

Достигается это все благодаря тому, что для каждой переменной, объявленной в основном конструкторе, автоматически генерируются функции componentN(), где N - номер позиции переменной в конструкторе. Что делает данная функция? Возвращает значение переменной. Такую функцию можно создать самому для класса, который не является классом данных.

```
class Person(val name : String , val age : Int) {
  operator fun component1() = name
  operator fun component2() = age
}

...
val person = Person("Adam", 27)
val name = person.component1()
val age = person.component2()
```

***

## Стандартные классы данных

В стандартной библиотеке Kotlin есть два класса данных: `Pair` и `Triple`. Из названий понятно, что они позволяют хранить две и три переменных разного типа одновременно. В данной статье подробно их описывать не буду, добавлено в познавательных целях.

***

## Полезные ссылки

[Data Classes](https://kotlinlang.org/docs/reference/data-classes.html "kotlinlang.org") - официальная документация.  
[Destructuring Declarations](https://kotlinlang.org/docs/reference/multi-declarations.html "kotlinlang.org") - официальная документация.  
[Классы данных](https://kotlinlang.ru/docs/reference/data-classes.html "kotlinlang.ru") - перевод на русский.  
[Мультидекларации](https://kotlinlang.ru/docs/reference/multi-declarations.html "kotlinlang.ru") - перевод на русский.