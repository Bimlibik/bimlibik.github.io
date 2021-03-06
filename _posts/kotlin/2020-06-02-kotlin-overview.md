---
title: Kotlin. Общий обзор
author: Leslie M.
date: "2020-06-02 01:40:00 +0800"
categories: [Kotlin, Basics]
tags: [kotlin, theory, basics]
---

Начало разработки Kotlin было анонсировано JetBrains в 2011 году.
Планировался он как альтернатива языкам Java и Scala, так как тоже выполняется
под управлением Java Virtual Machine. И спустя 6 лет компания Google
анонсировала начало официальной поддержки Kotlin, как языка для разработки под
операционную систему Android.

Почему именно Kotlin? Потому что Kotlin - это язык со свежим взглядом на
старые вещи. Он дает нам простые и удобные инструменты, которые позволяют
писать лаконичный код.

Какую бы статью про Kotlin я не читала, в них всегда присутствует сравнение
того, как один и тот же код выглядит на Java и Kotlin. После чего следует
вывод, что на Kotlin все кратко и красиво. По факту так и есть. На своем
небольшом опыте работы с обоими языками могу сказать, что как только я начала
использовать Kotlin в своих проектах, то у меня сразу отпало желание
возвращаться к Java. И это не потому что Java какой-то плохой язык. Просто
Kotlin удобнее и к этому быстро привыкаешь.

К тому же Android Studio полностью поддерживает Kotlin, позволяя создавать новые
проекты с файлами Kotlin, добавлять файлы Kotlin в существующий проект и даже
конвертировать Java в Kotlin. Абсолютно все инструменты, доступные в Android
Studio, можно использовать при разработке на Kotlin.

Так как Kotlin полностью совместим с Java, то при работе с Android API можно
заметить, что очень часто код выглядит почти также как и соответствующий код
на Java. С единственной поправкой - вызовы методов можно комбинировать с
особенностями языка Kotlin.

***

## Релизы

На данный момент последняя версия Kotlin вышла 17 августа 2020 года - 1.4.0. [Подробнее](https://blog.jetbrains.com/ru/kotlin/2020/08/kotlin-1-4-released-with-a-focus-on-quality-and-performance/).

Версия 1.3.70, дата выхода - 3 марта 2020 года. [Подробнее](https://blog.jetbrains.com/kotlin/2020/03/kotlin-1-3-70-released/ "blog.jetbrains.com").

***

## Статьи

В данном разделе будут ссылки на мои статьи о Kotlin.

[Основной синтаксис](https://bimlibik.github.io/posts/kotlin-basic-syntax/). <br>
[Null-безопасность. Операторы "?.", "!!.", "?:"](https://bimlibik.github.io/posts/kotlin-null-safety/). <br>
[Модификаторы доступа](https://bimlibik.github.io/posts/kotlin-visibility-modifiers/). <br>
[Модификатор const](https://bimlibik.github.io/posts/kotlin-const-modifier/). <br>
[Отложенная и ленивая инициализация свойств](https://bimlibik.github.io/posts/kotlin-lateinit-and-lazy/). <br>
[Лямбда-выражения и анонимные функции](https://bimlibik.github.io/posts/kotlin-lambdas-expressions-and-anonymous-functions). <br>
[Функции области видимости (Scope Functions)](https://bimlibik.github.io/posts/kotlin-scope-functions/). <br>
[Перегрузка операторов](https://bimlibik.github.io/posts/kotlin-operator-overloading/). <br>


### Классы и объекты

[Ключевое слово open](https://bimlibik.github.io/posts/kotlin-open-keyword/). <br>
[Классы данных (Data classes)](https://bimlibik.github.io/posts/kotlin-data-classes/). <br>
[Вложенные и внутренние классы](https://bimlibik.github.io/posts/kotlin-nested-and-inner-clesses/). <br>
[Классы перечислений (enum)](https://bimlibik.github.io/posts/kotlin-enum-classes/). <br>
[Изолированные классы (sealed classes)](https://bimlibik.github.io/posts/kotlin-sealed-classes/). <br>
[Основной и вторичный конструкторы. Init блок](https://bimlibik.github.io/posts/kotlin-constructors-and-init-block/). <br>
[Абстрактные классы и интерфейсы](https://bimlibik.github.io/posts/kotlin-abstract-classes-and-interfaces/). <br>
[Ключевое слово object](https://bimlibik.github.io/posts/kotlin-object-keyword/). <br>


### Библиотеки

[Нюансы при использовании библиотеки Gson](https://bimlibik.github.io/posts/kotlin-gson/)

***

## Полезные ссылки

[Официальная документация](https://kotlinlang.org/docs/reference/ "kotlinlang.org").

[Неофициальный сайт](https://kotlinlang.ru/ "kotlinlang.ru") с частичным переводом документации на русский язык.

[Kotlin blog](https://blog.jetbrains.com/kotlin/ "blog.jetbrains.com").

[Kotlin Christmas](https://kotlin.christmas/2020 "kotlin.christmas") - ресурс, где вы найдете множество интересных статей по Kotlin, библиотеках, фреймворках и лучших практик.
