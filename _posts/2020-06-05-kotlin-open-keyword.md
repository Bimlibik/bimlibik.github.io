---
title: Kotlin. Ключевое слово open. Наследование
author: Leslie M.
date: "2020-06-05 04:20"
categories: [Kotlin, Theory]
tags: [kotlin, theory, basics, keyword_open]
---

По умолчанию в Kotlin нельзя унаследовать один класс от другого. Связано это с
тем, что всем классам при создании неявно задаётся статус **final**, который и
блокирует возможность наследования. Если мы всё же попытаемся унаследоваться от
такого класса, то получим ошибку: "_This type is final, so it cannot be inherited from_".

Чтобы этого избежать, нужно сделать класс наследуемым. В этом и поможет ключевое
слово **open**. Отмечаем им нужный класс, после чего он может стать родительским.

```
open class Fraction {
  ...
}
```

Не только классы, но и функции в Kotlin по умолчанию имеют статус **final**.
Поэтому те функции, которые находятся в родительском классе и которые вы хотите
переопределить в дочерних классах, также должны быть отмечены ключевым словом
**open**.

```
open class Fraction {

  open fun toAttack() {
    ...
  }

}
```

Неожиданно, но и свойства класса по умолчанию являются **final**. Для возможности  
переопределения таких свойств в дочерних классах, не забудьте и их отметить
ключевым словом **open**.

```
open class Fraction {

  open val name: String = "default"

  open fun toAttack() {
    ...
  }

}
```

При этом, если в открытом классе будут присутствовать функции и свойства,
которые не отмечены словом **open**, то переопределяться они не будут. Но
дочерний класс сможет к ним обращаться.

```
open class Fraction {

  open val name: String = "default"

  fun toAttack() {
    ...
  }

}

class Horde : Fraction() {
  override val name = "Horde"
}

class SomeClass() {
  val horde = Horde()
  horde.toAttack()
}
```

***

## Полезные ссылки

Официальная документация - [наследование](https://kotlinlang.org/docs/reference/classes.html "kotlinlang.org"). <br>
Неофициальный перевод документации на русский язык - [наследование](https://kotlinlang.ru/docs/reference/classes.html "kotlinlang.ru").
