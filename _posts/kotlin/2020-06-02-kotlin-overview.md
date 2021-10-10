---
title: Kotlin. Общий обзор
author: Bimlibik
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

На данный момент последняя версия Kotlin вышла 24 августа 2021 года - 1.5.30. [Подробнее][kotlin-1.5.30].

Версия 1.5.20, дата выхода - 24 июня 2021 года. [Подробнее][kotlin-1.5.20]

Версия 1.5.0, дата выхода - 5 мая 2021 года. [Подробнее][kotlin-1.5.0]

Версия 1.4.30, дата выхода - 3 февраля 2021 года. [Подробнее][kotlin-1.4.30]

Версия 1.4.20, дата выхода - 23 ноября 2020 года. [Подробнее][kotlin-1.4.20]

Версия 1.4.0, дата выхода - 17 августа 2020 года. [Подробнее][kotlin-1.4.0]

Версия 1.3.70, дата выхода - 3 марта 2020 года. [Подробнее][kotlin-1.3.70-blog].

***

## Статьи

В данном разделе будут ссылки на мои статьи о Kotlin.

[Основной синтаксис][bimlibik-basic-syntax].  
[Null-безопасность. Операторы "?.", "!!.", "?:"][bimlibik-null-safety].  
[Модификаторы доступа][bimlibik-visibility-modifiers].  
[Модификатор const][bimlibik-const-modifier].  
[Отложенная и ленивая инициализация свойств][bimlibik-lateinit-and-lazy].  
[Лямбда-выражения и анонимные функции][bimlibik-lambdas-expressions-and-anonymous-functions].  
[Функции области видимости (Scope Functions)][bimlibik-scope-functions].  
[Перегрузка операторов][bimlibik-operator-overloading].  
[Коллекции][bimlibik-collections].


### Классы и объекты

[Ключевое слово open][bimlibik-open-keyword].  
[Классы данных (Data classes)][bimlibik-data-classes].  
[Вложенные и внутренние классы][bimlibik-nested-and-inner-clesses].  
[Классы перечислений (enum)][bimlibik-enum-classes].  
[Изолированные классы (sealed classes)][bimlibik-sealed-classes].  
[Основной и вторичный конструкторы. Init блок][bimlibik-constructors-and-init-block].  
[Абстрактные классы и интерфейсы][bimlibik-abstract-classes-and-interfaces].  
[Ключевое слово object][bimlibik-object-keyword].  


### Библиотеки

[Нюансы при использовании библиотеки Gson][bimlibik-gson].

***

## Полезные ссылки

[Официальная документация][doc-kotlin-official-eng] - на английском языке.  
[Learn Kotlin by Example][doc-learn-by-example-eng] - изучай Kotlin на примерах. Понятно и коротко объясняются основные фичи Котлина с такими же доступными примерами, которые можно здесь же запустить и посмотреть результат выполнения.  
[Неофициальный сайт][doc-kotlin-ru] с частичным переводом документации на русский язык.  
[Kotlin blog][kotlin-blog-official-eng].  
[Kotlin Christmas][kotlin-christmas] - ресурс, где вы найдете множество интересных статей по Kotlin, библиотеках, фреймворках и лучших практик.  


<!-- Ссылки на сторонние ресурсы -->
[doc-kotlin-official-eng]: https://kotlinlang.org/docs/reference/ "kotlinlang.org"
[doc-learn-by-example-eng]: https://play.kotlinlang.org/byExample/overview "play.kotlinlang.org"
[doc-kotlin-ru]: https://kotlinlang.ru/ "kotlinlang.ru"
[kotlin-blog-official-eng]: https://blog.jetbrains.com/kotlin/ "blog.jetbrains.com"
[kotlin-christmas]: https://kotlin.christmas/2020 "kotlin.christmas"

<!-- Ссылки на версии Kotlin -->
[kotlin-1.3.70-blog]: https://blog.jetbrains.com/kotlin/2020/03/kotlin-1-3-70-released/ "blog.jetbrains.com"
[kotlin-1.4.0]: https://kotlinlang.org/docs/whatsnew14.html "kotlinlang.org"
[kotlin-1.4.20]: https://kotlinlang.org/docs/whatsnew1420.html "kotlinlang.org"
[kotlin-1.4.30]: https://kotlinlang.org/docs/whatsnew1430.html "kotlinlang.org"
[kotlin-1.5.0]: https://kotlinlang.org/docs/whatsnew15.html "kotlinlang.org"
[kotlin-1.5.20]: https://kotlinlang.org/docs/whatsnew1520.html "kotlinlang.org"
[kotlin-1.5.30]: https://kotlinlang.org/docs/whatsnew1530.html "kotlinlang.org"

<!-- Внутренние ссылки -->
[bimlibik-basic-syntax]: /posts/kotlin-basic-syntax/ "bimlibik.github.io"
[bimlibik-null-safety]: /posts/kotlin-null-safety/ "bimlibik.github.io"
[bimlibik-visibility-modifiers]: /posts/kotlin-visibility-modifiers/ "bimlibik.github.io"
[bimlibik-const-modifier]: /posts/kotlin-const-modifier/ "bimlibik.github.io"
[bimlibik-lateinit-and-lazy]: /posts/kotlin-lateinit-and-lazy/ "bimlibik.github.io"
[bimlibik-lambdas-expressions-and-anonymous-functions]: /posts/kotlin-lambdas-expressions-and-anonymous-functions/ "bimlibik.github.io"
[bimlibik-scope-functions]: /posts/kotlin-scope-functions/ "bimlibik.github.io"
[bimlibik-operator-overloading]: /posts/kotlin-operator-overloading/ "bimlibik.github.io"
[bimlibik-collections]: /posts/kotlin-collections/ "bimlibik.github.io"

[bimlibik-open-keyword]: /posts/kotlin-open-keyword/ "bimlibik.github.io"
[bimlibik-data-classes]: /posts/kotlin-data-classes/ "bimlibik.github.io"
[bimlibik-nested-and-inner-clesses]: /posts/kotlin-nested-and-inner-clesses/ "bimlibik.github.io"
[bimlibik-enum-classes]: /posts/kotlin-enum-classes/ "bimlibik.github.io"
[bimlibik-sealed-classes]: /posts/kotlin-sealed-classes/ "bimlibik.github.io"
[bimlibik-constructors-and-init-block]: /posts/kotlin-constructors-and-init-block/ "bimlibik.github.io"
[bimlibik-abstract-classes-and-interfaces]: /posts/kotlin-abstract-classes-and-interfaces/ "bimlibik.github.io"
[bimlibik-object-keyword]: /posts/kotlin-object-keyword/ "bimlibik.github.io"

[bimlibik-gson]: /posts/kotlin-gson/ "bimlibik.github.io"
