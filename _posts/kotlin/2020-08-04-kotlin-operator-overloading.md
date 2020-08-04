---
title: Kotlin. Перегрузка операторов
author: Leslie M.
date: "2020-08-04 17:00:00 +0800"
categories: [Kotlin, Basics]
tags: [kotlin, operator, operator overloading]
---

Когда вы в своём коде используете какой-либо оператор, за кулисами вызывается соответствующая ему функция. При этом каждому оператору соответствует функция со строго определённым именем. Например, оператору `+` соответствует функция `plus`.

Kotlin позволяет перегружать операторы, тем самым становится возможным их использование по своему усмотрению.

Набор операторов, которые могут быть перегружены, ограничен. Каждому такому оператору соответствует имя функции, которую нужно определить в своём классе. Функции, которые перегружают операторы должны быть отмечены модификатором `operator`.

***

## Перегрузка арифметических операторов

### Бинарные операторы

Выражение  |  Имя функции
--|--
a * b  |  times
a / b  |  div
a % b  |  mod - в Kotlin 1.0
a % b  |  rem - с Kotlin 1.1
a + b  |  plus
a - b  |  minus

Допустим у нас есть класс `Tree`.

```
data class Tree(val name: String, val age: Int)
```

Мы можем реализовать сложение возрастов двух деревьев следующим образом:

```
data class Tree(val name: String, val age: Int) {
    operator fun plus(other: Tree): Int {
        return age + other.age
    }
}

...

val pine = Tree("Сосна", 2)
val apple = Tree("Яблоня", 4)

println(pine + apple)  // 6
```

В классе данных мы объявили функцию `plus` и отметили её модификатором `operator`, который сообщает о вашем намерении использовать эту функцию совместно с оператором `+`. Теперь объекты можно складывать с помощью оператора `+`, а за кулисами на место оператора компилятор будет подставлять вызов функции `plus`.

При этом можно определить несколько функций с одним именем, отличающихся только типами параметров.


### Составные операторы присваивания

Выражение  |  Имя функции
--|--
a += b  |  plusAssign
a -= b  |  minusAssign
a *= b  |  timesAssign
a /= b  |  divAssign
a %= b  |  modAssign

Не стоит одновременно перегружать составной оператор присваивания и соответствующий ему бинарный оператор (например, `plusAssign` и `plus`), так как компилятор сообщит об ошибке. Ошибку можно исправить заменив оператор на вызов функции. Либо заменив `var` на `val`, тогда операция `plusAssign` станет недоступной (неизменяемая переменная).

Но лучше при проектировании класса следовать следующему принципу: если класс неизменяемый, то добавлять в него только операции, возвращающие новые значения (бинарные); если класс изменяемый, то добавлять в него только модифицирующие операции (составные операторы присваивания).

Также в таких функциях возвращаемым значением должен быть `Unit`, иначе будет фиксироваться ошибка.


### Унарные операторы

Выражение  |  Имя функции
--|--
+a  |  unaryPlus
-a  |  unaryMinus
!a  |  not

Эти функции не принимают никаких аргументов, а принцип объявления аналогичен предыдущим.

```
data class Tree(val name: String, val age: Int) {
    operator fun unaryMinus(): Int {
        return -age
    }
}

...

val pine = Tree("Pine", 2)
println(-pine)   // -2
```


### Инкремент и декремент

Ввыражение  |  Имя функции
--|--
a++, ++a  |  inc
a--, --a  |  dec

Постфиксная операция `a++` сначала вернёт текущее значение, затем увеличит его. Префиксная операция `++a` сначала увеличит текущее значение, затем вернёт его.

```
data class Tree(val name: String, var age: Int) {
    operator fun inc() = Tree(name, ++age)

}

...

var pine = Tree("Pine", 2)
println(++pine)  // 3
```

Возвращаемым значением должен быть класс, в котором объявлена функция. Если эта функция является функцией-расширением, применимой для приёмника типа `T`, то возвращаемым значением должен быть подтип `T`.

***

## Перегрузка операторов сравнения

### Равенство и неравенство (equals)

Выражение  |  Имя функции
--|--
a == b  |  equals
a != b  |  equals

Оба оператора используют функцию `equals`, но с одним различием: для оператора `!=` результат инвертируется. Делается это автоматически, поэтому дополнительно ничего делать не требуется.

В отличие от остальных операторов, операторы `==` и `!=` можно использовать со значениями равными **null**. Сравнение `a == b` сначала проверит операнды на равенство **null**. Если результат получился отрицательным, то вызовется функция `equals`.

Для классов, отмеченных модификатором `data` ([классы данных][1-bimlibik-data]), `equals` генерируется автоматически. Но его можно определить самостоятельно, в том и числе и в классе данных.

```
data class Tree(val name: String, var age: Int) {
    override fun equals(other: Any?): Boolean {
        if (other === this) return true
        if (other !is Tree) return false
        return other.name == name && other.age == age
    }

    ...

    val pine = Tree("Pine", 2)
    val apple = Tree("Apple", 4)

    println(pine == apple)   // false
}
```

Функция `equals` не отмечается модификатором `operator`. Вместо этого она переопределяется (`override`), потому что реализация этого метода есть в классе `Any` и базовый метод уже отмечен модификатором `operator`.

Также стоит отметить, что оператор строгого равенства, или идентичности (`===`, `!==`), не может быть перегружен.


### Сравнение (compareTo)

Выражение  |  Имя функции
--|--
a > b  |  compareTo
a < b  |  compareTo
a >= b  |  compareTo
a <= b  |  compareTo

Классы в Kotlin могут использовать интерфейс **Comparable**, если нужна возможность сравнения их экземпляров: функция `compareTo` определяет, какой из двух объектов больше. Данная функция может вызываться не только напрямую, но и с использованием операторов сравнения, указанных в таблице выше.

Все операторы транслируются в вызов функции `compareTo`, которая должна возвращать значение типа `Int`.

```
data class Tree(val name: String, var age: Int) : Comparable<Tree> {
    override fun compareTo(other: Tree): Int {
        return compareValuesBy(this, other, Tree::name, Tree::age)
    }

}

...

val pine = Tree("Pine", 2)
    val apple = Tree("Apple", 4)

    println(pine > apple)  // true
```

Функция `compareTo` не отмечается модификатором `operator`, так как он уже применён к функции базового интерфейса. Также здесь используется функция из стандартной библиотеки Kotlin `compareValuesBy`, которая принимает список функций обратного вызова, результаты которых подлежат сравнению. В данном примере функции передаются ссылки на свойства, но ещё можно передавать [лямбда--выражения][2-bimlibik-lambdas].

***

## Перегрузка операторов для коллекций и диапазонов

### Обращение по индексу

Выражение  |  Имя функции
--|--
a[i]  |  get
a[i] = b  |  set

В Kotlin к элементам коллекций можно обращаться с помощью квадратных скобок. Такой же синтаксис можно добавить для использования и в собственный класс.

```
data class Tree(var name: String, var age: Int) {
    operator fun get(index: Int): Any {
        return when(index) {
            0 -> name
            1 -> age
            else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
        }
    }

    operator fun set(index: Int, value: Int) {
        when(index) {
            1 -> age = value
            else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
        }
    }

    operator fun set(index: Int, value: String) {
        when(index) {
            0 -> name = value
            else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
        }
    }

}

...

val pine = Tree("Сосна", 2)
println(pine[0])  // Сосна

pine[0] = "Ель"
println(pine[0])  // Ель
```


### Оператор `in`

Выражение  |  Имя функции
--|--
a in b  |  contains
a !in b  |  contains

Оба оператора `in` и `!in` используются для  проверки того, входит ли объект в коллекцию. Они транслируются в функцию `contains`, но результат для оператора `!in` инвертируется.

```
data class Tree(var name: String, var age: Int) {
    operator fun contains(char: Char): Boolean {
        return char in name
    }
}

...

val pine = Tree("Сосна", 2)
println('с' in pine.name)  // true
```

***

## Мультидекларации

Мультидекларации - это особенность, которая позволяет распаковать объект и использовать его значения для инициализации сразу нескольких переменных.

```
val pine = Tree("Сосна", 2)
val(name, age) = pine
println(name)                  // Сосна
```

Достигнуть такого результата можно объявив в своём классе функции `componentN()`, где **N** - номер позиции переменной в конструкторе.

```
class Tree(var name: String, var age: Int) {
    operator fun component1() = name
    operator fun component2() = age
}
```

О мультидекларациях я упоминала при разборе [классов данных][3-bimlibik-destructuring], так как в них функция `componentN()` генерируется автоматически.

***

## Полезные ссылки
[Operator overloading][1-ref-doc-en] - официальная документация.  
[Перегрузка операторов][2-ref-doc-ru] - перевод на русский язык.  



<!-- Полезные ссылки -->
[1-ref-doc-en]: https://kotlinlang.org/docs/reference/operator-overloading.html "kotlinlang.org"
[2-ref-doc-ru]: https://kotlinlang.ru/docs/reference/operator-overloading.html "kotlinlang.ru"

<!-- Ссылки -->
[1-bimlibik-data]: /posts/kotlin-data-classes/
[2-bimlibik-lambdas]: /posts/kotlin-lambdas-expressions-and-anonymous-functions/
[3-bimlibik-destructuring]: /posts/kotlin-data-classes/#%D0%BC%D1%83%D0%BB%D1%8C%D1%82%D0%B8%D0%B4%D0%B5%D0%BA%D0%BB%D0%B0%D1%80%D0%B0%D1%86%D0%B8%D0%B8
