---
title: Kotlin. Нюансы при использовании библиотеки Gson
author: Leslie M.
date: "2021-01-15 04:13"
categories: [Kotlin, Libraries]
tags: [kotlin, gson, json, android]
---

В данной статье я собрала некоторые нюансы, которые стоит учитывать, если вы пишите на Kotlin и при этом используете библиотеку [Gson][gson-github]. Лично мной не проверялось, записано на будущее.

***

Gson - это очень популярная библиотека для сериализации и десериализации объектов Java и JSON. Главный нюанс заключается в том, что Gson создан для Java и поэтому не знает об особенностях Kotlin.

Как правило, при работе с JSON мы хотим использовать такие особенности Kotlin как:
- [классы данных][bimlibik-data-classes];
- [null-безопасность][bimlibik-null-safety];
- [аргументы по умолчанию][bimlibik-default-args].

В результате у нас получится примерно такой класс:

```
data class Tree (
  val name: String,
  val description: String,
  val color: String = "Зелёный"
)
```

От такой реализации мы ожидаем одного: если мы получим JSON объект, у которого одно из значений (`name`, `description` иди `color`) равно `null` (или вовсе отсутствует), то вместо `null` будет использовано значение по умолчанию.

По факту значения по умолчанию будут игнорироваться, что рано или поздно приведёт к `NullPointerException` или `TypeCastException`. При этом десериализация будет проходить успешно.

Связано это с тем, что для инициализации полей Gson использует конструктор по умолчанию. Конструктор по умолчанию не может принимать аргументы. Однако, в нашем классе мы определили конструктор, который принимает аргументы и как результат конструктор по умолчанию стал не доступен. Поэтому Gson полностью пропускает инициализацию полей.

В репозитории библиотеки Gson можно найти несколько [сообщений об ошибке][gson-issues] на эту тему. Но в этом нет никакой ошибки, просто библиотека предназначена для Java.

Ниже представлены некоторые варианты, решающие данную проблему.

***

## Nullable значения

Самый простой вариант - это сделать все свойства **nullable**, т.е. с поддержкой null-значений.

```
data class Tree (
  val name: String?,
  val description: String?,
  val color: String?
)
```

Но помните, что объявляя **nullable** свойство вы берёте на себя ответственность по проверке его значения. Иначе компилятор будет запрещать вызов функций для таких значений, ведь это может привести к `NullPointerException`.

***

## Вспомогательные свойства

Данный вариант предполагает использование вспомогательных свойств совместно с аннотацией `@SerializedName()`.

Свойства, значения которых могут быть `null`, объявляем приватными. Таким вспомогательным свойствам принято добавлять нижнее подчёркивание перед именем.

```
data class Tree (
  @SerializedName("name") private val _name: String?,
  @SerializedName("description") private val _description: String?,
  @SerializedName("color") val _color: String?
)
```

В теле класса объявляем свойства для чтения (с геттером). В геттере будет происходить проверка поступившего во вспомогательное свойство значения на `null`. И если значение равно `null`, то вместо `null` должно быть установлено значение по умолчанию.

```
data class Tree (
  @SerializedName("name") private val _name: String?,
  @SerializedName("description") private val _description: String?,
  @SerializedName("color") val _color: String?
) {
  val name
    get() = _name ?: throw IllegalArgumentException("Неизвестное имя дерева")

  val description
    get() = _description ?: "Описание отсутствует"

  val color
    get() = _color ?: "Зелёный"
}
```

Можно проверить все необходимые свойства на `null`, вызвав их в блоке `init`. В этом случае мы будем получать исключение каждый раз, когда значение одного из свойств равно `null`.

```
data class Tree (
  @SerializedName("name") private val _name: String?,
  @SerializedName("description") private val _description: String?
) {
  val name
    get() = _name ?: throw IllegalArgumentException("Неизвестное имя дерева")

  val description
    get() = _description ?: "Описание отсутствует"

  init {
    this.name
    this.description
    this.color
  }
}
```

***

## Значения по умолчанию для всех свойств

Если всем свойствам в классе присвоить значения по умолчанию, то Gson будет использовать их, когда соответствующее поле в объекте JSON отсутствует.

```
data class Tree (
  val name: String = "",
  val description: String = "",
  val color: String? = "Зелёный"
)
```

Это работает, потому что компилятор Kotlin генерирует конструктор по умолчанию, когда всем свойствам присвоено значение по умолчанию. А как упоминалось ранее, Gson использует конструктор по умолчанию для инициализации полей.

Но есть одна оговорка. Это работает только когда поле **отсутствует** в объекте JSON. Если же оно есть и его значение равно `null`, тогда этот вариант не сработает. Gson десериализует нулевое значение даже в ненулевой тип без каких-либо ошибок. Но в дальнейшем это может привести к `TypeCastException`.

```
// поле "color" отсутствует, будет использовано значение по умолчанию
{
  "name": "Сосна",
  "description": "Хвойное дерево с длинными иглами и округлыми шишками."
}

// значение "color" равно null, значение по умолчанию не будет использовано
{
  "name": "Сосна",
  "description": "Хвойное дерево с длинными иглами и округлыми шишками.",
  "color": null
}
```

***

## Реализация TypeAdapterFactory

С помощью библиотеки [Kotlin Reflect][kotlin-reflect] мы можем создать свой собственный [TypeAdapterFactory][gson-type-adapter-factory]. Благодаря этому можно внедрить собственную логику, которая будет использоваться для десериализации каждого значения.

Например, можно создать адаптер, который будет выбрасывать специальное исключение при обнаружении значения равного `null` во время десериализации.

```
import com.google.gson.*
import com.google.gson.reflect.TypeToken
import com.google.gson.stream.JsonReader
import com.google.gson.stream.JsonWriter
import kotlin.jvm.internal.Reflection
import kotlin.reflect.KClass
import kotlin.reflect.full.memberProperties
class NullableTypAdapterFactory : TypeAdapterFactory {

  override fun <T : Any> create(gson: Gson, type: TypeToken<T>): TypeAdapter<T>? {
    val delegate = gson.getDelegateAdapter(this, type)

    // Если класс не является типом Kotlin, то используется адаптер по умолчанию
    if (type.rawType.declaredAnnotations.none {
      it.annotationClass.qualifiedName == "kotlin.Metadata" }) {
        return null
    }

    return object : TypeAdapter<T>() {
      override fun write(out: JsonWriter, value: T?) = delegate.write(out, value)

      override fun read(input: JsonReader): T? {
        val value: T? = delegate.read(input)

        if (value != null) {
          val kotlinClass: KClass<Any> = Reflection.createKotlinClass(type.rawType)

          // Проверка полей на возможность хранить null-значения
          kotlinClass.memberProperties.forEach {
            if (!it.returnType.isMarkedNullable && it.get(value) == null) {
              throw JsonParseException("Value of non-nullable member [${it.name}] cannot be null")
            }
          }

        }
        return value
      }
    }
  }
}
```

Пример использования адаптера:

```
data class Tree (val name: String, val description: String)

---

fun main() {
  val gson = GsonBuilder()
  .registerTypeAdapterFactory(NullableTypAdapterFactory())
  .create()

  val treeJson = """
    {
      "name": null,
      "description": "Хвойное дерево с длинными иглами и округлыми шишками."
    }
  """.trimIndent()

  val tree = gson.fromJson(treeJson, Tree::class.java) // JsonParseException
  println(tree.name.trim())
}
```

Принцип работы адаптера в следующем:
- `NullableTypeAdapterFactory` сначала проверяет, является ли класс (`Tree`) типом Kotlin. В противном случае использует стандартный адаптер.
- Получает `delegateAdapter` - адаптер, который Gson использовал бы, если бы мы ему его не предоставили. С помощью этого адаптера мы десериализуем объект.
- С помощью библиотеки **Kotlin Reflect**, заглядывает в класс объекта. Если одно из полей класса не может хранить `null`, а значение этого поля равно `null` (в объекте JSON), то выбрасывает исключение `JsonParseException`. Если же ничего подобного не было обнаружено, то возвращает десериализованный объект.

У этого решения есть главный недостаток - требуется использовать библиотеку **Kotlin Reflect**, которая немного увеличит размер приложения. Также мы, по сути, снижаем  производительность в обмен на null-безопасность.

Данный вариант взят из статьи [Using GSON with Kotlin’s Non-Null Types][medium-type-adapter].

***

## Библиотеки с поддержкой Kotlin

Библиотека Gson не является единственной для сериализации/десериализации. Существуют библиотеки, которые поддерживают все особенности Kotlin и с которыми вышеописанных проблем не возникнет.

Наиболее популярные:
- [Jackson][github-jackson]
- [Moshi][github-moshi]
- [Kotlin serialization][github-kotlin-serialization]

Также можно заглянуть на сайт [Awesome Kotlin][awesome-kotlin] - это сборник идей, связанных с Kotlin, в том числе различных библиотек. Правда на момент написания статьи в нём есть только вышеперечисленные библиотеки.




<!-- Ссылки -->
[awesome-kotlin]: https://kotlin.link/ "kotlin.link"
[bimlibik-data-classes]: https://bimlibik.github.io/posts/kotlin-data-classes/ "bimlibik.github.io"
[bimlibik-default-args]: https://bimlibik.github.io/posts/kotlin-constructors-and-init-block/#%D0%BE%D1%81%D0%BD%D0%BE%D0%B2%D0%BD%D0%BE%D0%B9-%D0%BA%D0%BE%D0%BD%D1%81%D1%82%D1%80%D1%83%D0%BA%D1%82%D0%BE%D1%80-primary-constructor "bimlibik.github.io"
[bimlibik-null-safety]: https://bimlibik.github.io/posts/kotlin-null-safety/ "bimlibik.github.io"
[github-jackson]: https://github.com/FasterXML/jackson "github.com"
[github-moshi]: https://github.com/square/moshi "github.com"
[github-kotlin-serialization]: https://github.com/Kotlin/kotlinx.serialization "github.com"
[gson-github]: https://github.com/google/gson "github.com"
[gson-issues]: https://github.com/google/gson/issues/1550 "github.com"
[gson-type-adapter-factory]: https://www.javadoc.io/doc/com.google.code.gson/gson/2.6.2/com/google/gson/TypeAdapterFactory.html "javadoc.io"
[kotlin-reflect]: https://mvnrepository.com/artifact/org.jetbrains.kotlin/kotlin-reflect "mvnrepository.com"
[medium-type-adapter]: https://medium.com/swlh/using-gson-with-kotlins-non-null-types-468b1c66bd8b "medium.com"
