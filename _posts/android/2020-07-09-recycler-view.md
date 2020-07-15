---
title: RecyclerView
author: Leslie M.
date: "2020-07-09 14:55:00 +0800"
categories: [Android, UI]
tags: [android, ui, recyclerView, layoutManager, snapping, viewType, mergeAdapter, concatAdapter]
---

`RecyclerView` - компонент для отображения элементов списка, который является более продвинутой и гибкой версией `ListView`, но не является его родственником, а относится к семейству `ViewGroup`.

***

## Принцип работы

Для отображения данных `RecyclerView` использует несколько компонентов:
- Объект `RecyclerView`, который нужно добавить в макет. Он заполняется элементами списка в зависимости от того, какой был установлен `LayoutManager`. Существуют стандартные `LayoutManager`'ы, например, `LinearLayoutManager` отображает элементы в виде списка, а `GridLayoutManager` -  в виде сетки. Но можно создать и свой собственный `LayoutManager`.
- Элементы списка представлены в виде объектов `viewHolder`. Например, если список состоит из различных видов деревьев, то `viewHolder` - это конкретный вид дерева - сосна, яблоня, берёза и т.д. `RecyclerView` создает столько объектов `viewHolder`, сколько требуется для отображения на экране устройства и несколько про запас. Когда пользователь начинает прокручивать список, `RecyclerView` берёт те объекты `viewHolder`, которые ушли за пределы экрана и "привязывает" к ним новые данные.
- Объекты `viewHolder` управляются **адаптером**. Он создаёт объекты `viewHolder` и привязывает к ним информацию.

***

## Добавление RecyclerView в проект

### Библиотека

По умолчанию (при создании нового проекта) функциональность `RecyclerView` не доступна. Поэтому для начала нужно добавить соответствующую библиотеку. Для этого в файл `build.gradle`, который находится в папке `app`, добавьте одну из библиотек:

```
dependencies {
    // если приложение работает с библиотекой поддержки (support library)
    implementation 'com.android.support:recyclerview-v7:28.0.0'

    // если приложение работает с androidx
    implementation 'androidx.recyclerview:recyclerview:1.1.0'
}
```

Версия support library уже вряд ли когда-нибудь изменится, так как её поддержку остановили. А вот за версией androidx нужно следить.

После того, как библиотека добавлена, обязательно нажмите на кнопку **Sync Now**, чтобы изменения вступили в силу.


### Добавление в макет

Библиотека добавлена, а значит теперь мы можем обращаться к `RecyclerView`. Первым делом следует добавить его в макет. Он может быть добавлен как дочерний элемент другого компонента:

```
// res/layout/fragment_trees_list

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    ... >

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recycler_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

</LinearLayout>
```

Либо может быть единственным (корневым) компонентом макета:

```
// res/layout/fragment_trees_list

<?xml version="1.0" encoding="utf-8"?>
<androidx.recyclerview.widget.RecyclerView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/recycler_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

Далее для корректной работы `RecyclerView` требуется установить `LayoutManager` и адаптер. `LayoutManager` может быть установлен двумя способами. В макете:

```
// res/layout/fragment_trees_list

<?xml version="1.0" encoding="utf-8"?>
<androidx.recyclerview.widget.RecyclerView
    ...
    app:layoutManager="LinearLayoutManager" />
```

Либо вместе с адаптером в классе фрагмента или активити:

```
recycler_view.apply {
  layoutManager = LinearLayoutManager(context)
  adapter = SimpleTreesAdapter(trees)
}
```

При этом начиная с версии Android Studio 3.6 необязательно вызывать метод findViewById(), а можно напрямую обратится к компоненту из макета по его идентификатору (как в примере выше).


### Макет элемента списка

Для элемента списка можно создать собственный макет. Например, у меня каждый элемент состоит из названия дерева и его краткого описания. Поэтому макет для элемента включает в себя два компонента `TextView`:

```
// res/layout/item_tree.xml

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal">

    <TextView
        android:id="@+id/name"
        android:layout_width="0dp"
        android:layout_weight="1"
        android:layout_height="wrap_content"
        android:layout_margin="16dp" />

    <TextView
        android:id="@+id/description"
        android:layout_width="0dp"
        android:layout_weight="3"
        android:layout_height="wrap_content"
        android:layout_margin="16dp" />
</LinearLayout>
```

А можно воспользоваться стандартными макетами, к которым можно обращаться через `android.R.layout.НАЗВАНИЕ_ИЗ_СПИСКА`. Но на мой взгляд это вариант для ленивых или для любопытных.


### Адаптер и `viewHolder`

Адаптер - это класс, который занимается передачей данных в список, созданием объектов `viewHolder` и их обновлением. Адаптер должен наследоваться от класса `RecyclerView.Adapter`.

`ViewHolder` - это тоже класс, объекты которого адаптер использует для хранения и визуализации элементов списка. `ViewHolder` должен наследоваться от класса `RecyclerView.ViewHolder`. Как правило этот класс располагают внутри адаптера.

```
class SimpleTreesAdapter(
    private val trees: ArrayList<Tree>
) : RecyclerView.Adapter<SimpleTreesAdapter.SimpleTreesViewHolder>() {

    ...

    class SimpleTreesViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
      ...
    }

}
```

В классе адаптера нужно обязательно переопределить 3 метода:
- `onCreateViewHolder()` - данный метод вызывается `LayoutManager`'ом, чтобы создать объекты `viewHolder` и передать им макет, по которому будут отображаться элементы списка.
- `onBindViewHolder()` - данный метод вызывается `LayoutManager`'ом, чтобы привязать к объекту `viewHolder` данные, которые он должен отображать.
- `getItemCount()` - возвращает общее количество элементов в списке.

А в классе `ViewHolder` требуется указать используемые компоненты разметки.

```
class SimpleTreesAdapter(
    private val trees: ArrayList<Tree>
) : RecyclerView.Adapter<SimpleTreesAdapter.SimpleTreesViewHolder>() {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): SimpleTreesViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_tree, parent, false)
        return SimpleTreesViewHolder(view)
    }

    override fun onBindViewHolder(holder: SimpleTreesViewHolder, position: Int) {
        val tree = trees[position]
        holder.names.text = tree.name
        holder.description.text = tree.description
    }

    override fun getItemCount(): Int = trees.size


    class SimpleTreesViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val names: TextView = itemView.findViewById(R.id.name)
        val description: TextView = itemView.findViewById(R.id.description)
    }

}
```


### Последние штрихи

Теперь адаптер настроен и готов к использованию. Осталось только его подключить. Делается это в классе фрагмента или активити (в моём примере используется фрагмент).

```
class SimpleTreesFragment : Fragment() {

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        val view = inflater.inflate(R.layout.fragment_trees_list, container, false)

        if(view is RecyclerView) {
            with(view) {
                layoutManager = LinearLayoutManager(context)
                adapter = SimpleTreesAdapter(createData())
            }
        }
        return view
    }

    // функция заполняет массив информацией, которую берёт из ресурсов, и возвращает его
    private fun createData(): ArrayList<Tree> {
        val names = resources.getStringArray(R.array.names)
        val descriptions = resources.getStringArray(R.array.descriptions)
        val trees = mutableListOf<Tree>()

        for (i in names.indices) {
            trees.add(Tree(name = names[i], description = descriptions[i]))
        }
        return trees as ArrayList<Tree>
    }

    // для создания фрагмента
    companion object {
        fun newInstance() = SimpleTreesFragment()
    }
}


// класс Tree
data class Tree(
    val id: String = UUID.randomUUID().toString(),
    val name: String,
    val description: String)
```

Если какой-либо элемент списка изменился, то следует вызвать метод адаптера `notifyItemChanged()` и передать ему позицию элемента, которую требуется обновить. Вместо него можно использовать метод `notifyDataSetChanged()`, который будет обновлять полностью весь список, но из-за этого он является ресурсозатратным.


### Создание `RecyclerView` с помощью шаблона

Есть возможность пропустить все вышеописанные шаги и воспользоваться стандартным шаблоном. Этот шаблон автоматически добавит в проект новый фрагмент с поддержкой `RecyclerView`. Это означает, что студия за вас создаст не только фрагмент, но и адаптер, `ViewHolder`, макет для элемента списка, а также код, который всё это подключает.

Кликните по любому файлу правой кнопкой мыши и в появившемся контекстном меню выберите: **New > Fragment > Fragment (List)**.


![1](/assets/img/posts/recycler-view/fragment-list.jpg)

***

## Стандартные LayoutManager'ы

`RecyclerView` использует `LayoutManager` для того, чтобы расположить элементы списка на экране. При этом каждый `LayoutManager` позволяет расположить их по-своему.


### Виды

Существует три стандартных `LayoutManager`'а:
- `LinearLayoutManager` - упорядочивает элементы в виде обычного вертикального или горизонтального списка.
- `GridLayoutManager` - размещает элементы в виде сетки одинакового размера.
- `StaggeredGridLayoutManager` - размещает элементы в виде неравномерной сетки: каждый столбец будет слегка смещён по сравнению с предыдущим.

Как правило этих вариантов достаточно для большинства ситуаций. Но если это не ваш случай, то можно создать свой собственный `LayoutManager`, расширив класс `RecyclerView.LayoutManager`.


### LinearLayoutManager

По умолчанию `LinearLayoutManager` упорядочивает элементы  в виде вертикального списка.

```
// вертикальный список
layoutManager = LinearLayoutManager(context)
```

У данного класса есть другой конструктор, который позволяет явно задать ориентацию списка. Помимо контекста, ему требуется два параметра:
- Ориентация - задаётся с помощью констант `HORIZONTAL` и `VERTICAL` класса `LinearLayoutManager`.
- Булево значение: если передать `true` - список будет установлен в обратном порядке (начало будет в конце).

```
// вертикальная ориентация
layoutManager = LinearLayoutManager(context, LinearLayoutManager.VERTICAL, false)

// горизонтальная ориентация
layoutManager = LinearLayoutManager(context, LinearLayoutManager.HORIZONTAL, false)
```

Эти параметры можно устанавливать с помощью специальных методов:

```
val linearManager = LinearLayoutManager(context)
linearManager.apply {
  orientation = LinearLayoutManager.HORIZONTAL
  reverseLayout = false
}
```

Либо с помощью специальных атрибутов в XML:
```
<androidx.recyclerview.widget.RecyclerView
    ...
    app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager"
    android:orientation="horizontal"
    app:reverseLayout="false"/>
```


### GridLayoutManager

Размещает элементы списка в виде сетки одинакового размера.

У класса `GridLayoutManager` есть два конструктора. Для использования первого конструктора необходимы два параметра: контекст и количество столбцов в сетке.

```
layoutManager = GridLayoutManager(context, 3)
```

Для второго конструктора - четыре параметра:
- контекст;
- количество столбцов в сетке;
- ориентация списка - задаётся с помощью констант `HORIZONTAL` и `VERTICAL` класса `LinearLayoutManager`;
- булево значение - если передать `true` - список будет установлен в обратном порядке (начало будет в конце).

Если задать горизонтальную ориентацию, то в списке будет столько рядов, сколько было задано вторым параметром (в данном примере = 3), а листаться, само собой, будет в бок.

```
layoutManager = GridLayoutManager(context, 3, LinearLayoutManager.HORIZONTAL, false)
```

То же самое можно задать с помощью XML атрибутов:

```
<androidx.recyclerview.widget.RecyclerView
    ...
    app:layoutManager="androidx.recyclerview.widget.GridLayoutManager"
    app:spanCount="3"
    android:orientation="horizontal"
    app:reverseLayout="false"/>
```


### StaggeredGridLayoutManager

Размещает элементы в виде неравномерной сетки.

У класса `StaggeredGridLayoutManager` всего один конструктор с двумя параметрами:
- количество столбцов в сетке;
- ориентация списка - задаётся с помощью констант `HORIZONTAL` и `VERTICAL` класса `StaggeredGridLayoutManager`.

Если задать горизонтальную ориентацию, то в списке будет столько рядов, сколько было задано первым параметром (в данном примере = 3), а листаться, само собой, будет в бок.

```
layoutManager = StaggeredGridLayoutManager(3, StaggeredGridLayoutManager.VERTICAL)
```

То же самое можно задать с помощью XML атрибутов:

```
<androidx.recyclerview.widget.RecyclerView
    ...
    app:layoutManager="androidx.recyclerview.widget.StaggeredGridLayoutManager"
    app:spanCount="3"
    android:orientation="vertical" />
```

### Динамическое переключение

Переключаться между `LayoutManager`'ами можно динамически. Например, при нажатии на кнопку:

```
btn_linear.setOnClickListener {
  recycler_view.apply { layoutManager = LinearLayoutManager(requireContext()) }
}

btn_grid.setOnClickListener {
  recycler_view.apply { layoutManager = GridLayoutManager(requireContext(), 3) }
}

btn_staggered.setOnClickListener {
  recycler_view.apply { layoutManager =
    StaggeredGridLayoutManager(3, StaggeredGridLayoutManager.VERTICAL)
  }
}
```

***

## SnapHelper

`SnapHelper` позволяет настроить "прилипание" элементов к определённой позиции в `RecyclerView`. Например, при пролистывании можно настроить прилипание таким образом, что первый видимый элемент будет сам прилипать к краю экрана или ближайший к центру элемент будет автоматически вставать в центр экрана.

Существует два стандартных класса для работы с прилипанием элементов: `LinearSnapHelper` и `PagerSnapHelper`.

`LinearSnapHelper` застовляет ближайший к центру элемент вставать в центр экрана. Допустим вы листаете список и в какой-то момент убрали пальцы от экрана. Список без вашего участия автоматически прокрутится и установит в центр экрана ближайший элемент.

`PagerSnapHelper` предназначен для полноэкранных элементов и ведёт себя как `ViewPager`.

Добавить себе в проект просто:

```
val snapHelper: SnapHelper = LinearSnapHelper()  // или PagerSnapHelper()
snapHelper.attachToRecyclerView(recyclerView)

// более короткий вариант
LinearSnapHelper().attachToRecyclerView(recyclerView)
PagerSnapHelper().attachToRecyclerView(recyclerView)
```

Если ни один вариант вас не устраивает, то создайте свою собственную реализацию этих классов и опишите в нёй необходимое поведение элементов при пролистывании списка.

***

## Использование нескольких макетов для элементов RecyclerView

При отображении списка все его элементы выглядят одинаково и в большинстве случаев это оправдано и разработчика вполне устраивает. Тем не менее возникают ситуации, когда нужно один или целый ряд элементов выделять из остальных. Например:
- требуется добавить header или footer;
- отображение двух списков в одном `RecyclerView`;
- выделение определённого элемента в списке.


### ViewType

Одним из способов, который используется для выделения элементов в списке, является присвоение `viewType` каждому объекту `viewHolder`. `ViewType` - это произвольное цифровое значение от 0 и выше, которое необходимо для того, чтобы различать объекты `viewHolder` между собой. Например, если вам требуется добавить header и footer, при этом элементы списка должны выглядеть идентично, то у вас будет три `viewType`: для header'а, footer'а и элемента списка.

Для начала добавьте в папку **res/layout** три макета: для header'а, footer'а и элемента списка.

```
// header.xml
<?xml version="1.0" encoding="utf-8"?>
<TextView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/header"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:textSize="30sp"
    android:textColor="#00695C"
    android:textStyle="bold"
    android:textAllCaps="true"
    android:text="Header"
    android:gravity="center"/>

// footer.xml
<?xml version="1.0" encoding="utf-8"?>
<TextView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/footer"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:textSize="30sp"
    android:textColor="#C62828"
    android:textStyle="bold"
    android:textAllCaps="true"
    android:text="Footer"
    android:gravity="center"/>

// item_tree_simple.xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal">

    <TextView
        android:id="@+id/name"
        android:layout_width="0dp"
        android:layout_weight="1"
        android:layout_height="wrap_content"
        android:layout_margin="16dp" />

    <TextView
        android:id="@+id/description"
        android:layout_width="0dp"
        android:layout_weight="3"
        android:layout_height="wrap_content"
        android:layout_margin="16dp" />
</LinearLayout>
```

В классе адаптера создадим константы, которые будут хранить значения `viewType`.

```
class HeaderAndFooterAdapter {
  ...

  companion object {
    const val HEADER_VIEW = 1
    const val LIST_ITEM_VIEW = 2
    const val FOOTER_VIEW = 3
  }
}
```

Для удобства создадим базовый класс `GenericViewHolder`.

```
class HeaderAndFooterAdapter {
  ...

  abstract class GenericViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {

    abstract fun bindView(position: Int)

  }
}
```

От него будут наследоваться три класса `ViewHolder`. Каждый из них отвечает за свой макет и привязку к нему данных.

```
private inner class ListItemViewHolder(itemView: View) : GenericViewHolder(itemView) {
    val name: TextView = itemView.findViewById(R.id.name)
    val description: TextView = itemView.findViewById(R.id.description)

    override fun bindView(position: Int) {
      name.text = trees[position - 1].name
      description.text = trees[position - 1].description
    }
}

private class HeaderViewHolder(itemView: View) : GenericViewHolder(itemView) {
    val header: TextView = itemView.findViewById(R.id.header)

    override fun bindView(position: Int) {
      header.text = "I'm a header"
    }
}

private class FooterViewHolder(itemView: View) : GenericViewHolder(itemView) {
    val footer: TextView = itemView.findViewById(R.id.footer)

    override fun bindView(position: Int) {
      footer.text = "I'm a footer"
    }
}
```

Обратите внимание на класс `ListItemViewHolder`. В отличии от остальных он является [внутренним](https://bimlibik.github.io/posts/kotlin-nested-and-inner-clesses/) (модификатор `inner`), так как ему для привязки данных требуется обращаться к свойству `trees` своего внешнего класса. Из поступившего номера позиции вычитается единица, так как нулевая позиция занята header'ом и не будет сюда поступать.

Теперь возьмёмся за код самого адаптера. С помощью метода `getItemViewType()` зададим `viewType` каждому объекту `viewHolder` в зависимости от его позиции в списке. Первый и последний элемент списка - это header и footer. Если в списке 15 элементов, то позиция для footer'а будет **15 + 1**, так как header всегда находится в нулевой позиции.

```
override fun getItemViewType(position: Int): Int {
    return when (position) {
        0 -> HEADER_VIEW
        trees.size + 1 -> FOOTER_VIEW
        else -> LIST_ITEM_VIEW
    }
}
```

В методе `onCreateViewHolder()` создаём объект `viewHolder` в зависимости от `viewType`.

```
override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): GenericViewHolder {
  val view: View
  return when(viewType) {
    HEADER_VIEW -> {
      view = LayoutInflater.from(parent.context).inflate(R.layout.header, parent, false)
      HeaderViewHolder(view)
    }
    FOOTER_VIEW -> {
      view = LayoutInflater.from(parent.context).inflate(R.layout.footer, parent, false)
      FooterViewHolder(view)
    }
    else -> {
      // LIST_ITEM_VIEW
      view = LayoutInflater.from(parent.context).inflate(R.layout.item_tree_simple, parent, false)
      ListItemViewHolder(view)
    }
  }
}
```

В методе `onBindViewHolder()` вызываем метод привязки данных `bindView()`, который переопределён во всех наших классах `ViewHolder`.

```
override fun onBindViewHolder(holder: GenericViewHolder, position: Int) {
  holder.bindView(position)
}
```

Метод `getItemCount()` должен возвращать количество элементов в `RecyclerView`. Поэтому следует учесть наличие header'а и footer'а.

```
override fun getItemCount(): Int = trees.size + 2
```

Адаптер готов к использованию. Результат будет примерно таким:

<p class="post-few-img">
  <img src="/assets/img/posts/android-recycler-view/header.png" alt="demo header" height="550"/>
  <img src="/assets/img/posts/android-recycler-view/footer.png" alt="demo footer" height="550"/>
</p>


### Несколько списков в одном RecyclerView

У меня как-то возникала необходимость отображения двух списков на одном экране друг за другом. При этом при клике по элементу из первого списка он должен был переместиться во второй список и наоборот.

Первая появившаяся мысль - добавить на экран два `RecyclerView`. И это вполне себе работает. Но возникает ряд неудобств, одно из них - некорректная работа **overScroll**. Эффект **overScroll** визуально показывает, что вы дошли до конца или начала списка.

<img src="/assets/img/posts/android-recycler-view/over-scroll.png" alt="demo over scroll" width="350"/>

И если на экране два `RecyclerView`, то эффект **overScroll** появляется для каждого из них. Выглядит не очень красиво. Конечно можно **overScroll** эффект отключить, но тогда появляется неуверенность: "А дошел ли я до конца списка?".

В общем нашлось решение, при котором один `RecyclerView` работает с двумя списками и завязано это всё на использовании  `viewType` из примера выше.

Создаём два макета - один для элементов первого списка, второй - для элементов второго списка. У меня они отличаются только цветом фона - у элементов первого списка он белый, у второго - красный.

```
// item_tree_multiple_1.xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    android:background="#FFFFFF">

    <TextView
        android:id="@+id/name"
        android:layout_width="0dp"
        android:layout_weight="1"
        android:layout_height="wrap_content"
        android:layout_margin="16dp" />

    <TextView
        android:id="@+id/description"
        android:layout_width="0dp"
        android:layout_weight="3"
        android:layout_height="wrap_content"
        android:layout_margin="16dp" />
</LinearLayout>


// item_tree_multiple_2.xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    android:background="#F3BBBB">

    <TextView
        android:id="@+id/name"
        android:layout_width="0dp"
        android:layout_weight="1"
        android:layout_height="wrap_content"
        android:layout_margin="16dp" />

    <TextView
        android:id="@+id/description"
        android:layout_width="0dp"
        android:layout_weight="3"
        android:layout_height="wrap_content"
        android:layout_margin="16dp" />
</LinearLayout>
```

Константы для хранения значений `viewType`:

```
companion object {
  const val FIRST_LIST_ITEM_VIEW = 1
  const val SECOND_LIST_ITEM_VIEW = 2
}
```

Также используем для удобства базовый класс `GenericViewHolder`.

```
abstract class GenericViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {

  abstract fun bindView(position: Int)
}
```

От него будут наследоваться два класса `ViewHolder`: один для элементов первого списка, второй для элементов второго списка.

```
private inner class FirstListItemViewHolder(itemView: View) : GenericViewHolder(itemView) {
    val name: TextView = itemView.findViewById(R.id.name)
    val description: TextView = itemView.findViewById(R.id.description)

    override fun bindView(position: Int) {
        ...
    }
}

private inner class SecondListItemViewHolder(itemView: View) : GenericViewHolder(itemView) {
    val name: TextView = itemView.findViewById(R.id.name)
    val description: TextView = itemView.findViewById(R.id.description)

    override fun bindView(position: Int) {
        ...
    }
}
```

Каждый из них будет по своему реализовывать метод `bindView()`. Для элементов первого списка ничего рассчитывать не требуется, нужно просто привязать к объектам `viewHolder` данные по порядку.

```
private inner class FirstListItemViewHolder(itemView: View) : GenericViewHolder(itemView) {
  val name: TextView = itemView.findViewById(R.id.name)
  val description: TextView = itemView.findViewById(R.id.description)

  override fun bindView(position: Int) {
    name.text = list1[position].name
    description.text = list1[position].description
  }
}
```

Второй список должен отображаться сразу после первого. Так как номер позиции может быть любым числом от 0 до `list1.size + list2.size`, в методе `bindView()` класса `SecondListItemViewHolder` потребуется произвести расчеты.

```
private inner class SecondListItemViewHolder(itemView: View) : GenericViewHolder(itemView) {
  val name: TextView = itemView.findViewById(R.id.name)
  val description: TextView = itemView.findViewById(R.id.description)

  override fun bindView(position: Int) {
    val i = if (list1.size > 0) {
      position - list1.size
    } else {
      position
    }
    name.text = list2[i].name
    description.text = list2[i].description
  }
}
```

Также обратите внимание что оба класса являются [внутренними](https://bimlibik.github.io/posts/kotlin-nested-and-inner-clesses/) (модификатор `inner`), так как им для привязки данных требуется обращаться к компонентам адаптера `list1` и `list2`.

Переходим к коду адаптера. В методе `getItemViewType()` нужно предусмотреть все возможные сценарии: когда в обоих списках есть элементы и когда в одном из списков нет элементов.

```
override fun getItemViewType(position: Int): Int {
  when {
    list1.size > 0 && list2.size > 0 -> {
      return if (position >= list1.size) SECOND_LIST_ITEM_VIEW
      else FIRST_LIST_ITEM_VIEW
    }
    list1.size == 0 && list2.size > 0 -> return SECOND_LIST_ITEM_VIEW
    list1.size > 0 && list2.size == 0 -> return FIRST_LIST_ITEM_VIEW
  }
  return super.getItemViewType(position)
}
```

В методе `onCreateViewHolder()` создаём объект `viewHolder` в зависимости от `viewType`.

```
override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): GenericViewHolder {
  val view: View
  return when(viewType) {
    FIRST_LIST_ITEM_VIEW -> {
      view = LayoutInflater.from(parent.context).inflate(R.layout.item_tree_multiple_1, parent, false)
      FirstListItemViewHolder(view)
    }
    else -> {
      // SECOND_LIST_ITEM_VIEW
      view = LayoutInflater.from(parent.context).inflate(R.layout.item_tree_multiple_2, parent, false)
      SecondListItemViewHolder(view)
    }
  }

}
```

В методе `onBindViewHolder()` вызываем метод привязки данных `bindView()`, который переопределён во всех наших классах `ViewHolder`, а также вешаем слушателя.

```
override fun onBindViewHolder(holder: GenericViewHolder, position: Int) {
  holder.bindView(position)
  holder.itemView.setOnClickListener { updateUi(holder.adapterPosition, holder.itemView.context) }
}
```

Метод `getItemCount()` должен возвращать количество элементов в `RecyclerView`. Поэтому следует учесть наличие двух списков.

```
override fun getItemCount(): Int = list1.size + list2.size
```

При клике по элементу из первого или второго списка будет вызван метод `updateUi()`, который отмечает, что по элементу кликнули и переносит его в другой список.

```
private fun updateUi(position: Int, context: Context) {
  if (position >= list1.size) {
    list2[position - list1.size].clicked = !list2[position - list1.size].clicked
  } else {
    list1[position].clicked = !list1[position].clicked
  }
  sortTrees()
  Toast.makeText(context, "$position", Toast.LENGTH_SHORT).show()
}

private fun sortTrees() {
  list1.clear()
  list2.clear()
  for (tree in trees) {
    if (tree.clicked) {
      list2.add(tree)
    } else {
      list1.add(tree)
    }
  }
notifyDataSetChanged()
}
```

Адаптер готов к использованию. Результат будет примерно таким:

<img src="/assets/img/posts/android-recycler-view/multiple-lists.gif" alt="demo multiple lists" width="350"/>

Необязательно делать списки динамическими, таким образом можно отображать и статические списки. И даже комбинировать с предыдущим примером - добавлять header (один или для всех списков) и footer.


## ConcatAdapter

Несмотря на то, что все примеры, описанные в предыдущем разделе, вполне себе рабочие, в плане кода выглядят не очень хорошо. В основном из-за того, что в одном адаптере скапливается множество реализаций класса `ViewHolder`, а также логика их отображения. Если нам понадобится добавить или удалить какой-либо `ViewHolder`, то придётся переписывать класс адаптера и заново его тестировать.

По этой причине в `recyclerview:1.2.0-alpha02` был добавлен новый класс `MergeAdapter`, который в версии `recyclerview:1.2.0-alpha04` переименовали в [`ConcatAdapter`](https://developer.android.com/reference/androidx/recyclerview/widget/ConcatAdapter "developer.android.com").

`ConcatAdapter` позволяет отображать содержимое нескольких адаптеров в одном `RecyclerView`. То есть вместо накапливания множества реализаций класса `ViewHolder` в одном адаптере, мы можем создать для каждого `ViewHolder`'а свой адаптер, а потом объединить их все при помощи `ConcatAdapter`. Таким образом код станет более понятным и переиспользуемым, а если потребуется добавить в `RecyclerView` что-то новое - просто создадим новый адаптер.


### Использование `ConcatAdapter`. Обзор некоторых методов класса

Передайте в конструктор `ConcatAdapter` все ваши адаптеры, которые нужно объединить, чтобы отображать их в одном `RecyclerView`.

```
val firstAdapter = FistAdapter()
val secondAdapter = SecondAdapter()
val thirdAdapter = ThirdAdapter()

val concatAdapter = ConcatAdapter(firstAdapter, secondAdapter, thirdAdapter)
recyclerView.adapter = concatAdapter
```

Адаптеры будут отображаться на экране в том порядке, в котором были переданы в конструктор класса `ConcatAdapter`.

***

Если один из адаптеров должен несколько раз отображаться на экране, то создайте несколько объектов этого адаптера и передайте их все в конструктор класса `ConcatAdapter`.

```
// Первый адаптер используется дважды
val firstAdapter = FistAdapter()
val secondAdapter = SecondAdapter()
val firstAdapter = FistAdapter()

val concatAdapter = ConcatAdapter(firstAdapter, secondAdapter, firstAdapter)
recyclerView.adapter = concatAdapter
```

***

Когда мы вызываем метод `notifyDataSetChanged()` в любом из адаптеров, `ConcatAdapter` тоже его вызывает.

***

У класса `ConcatAdapter` есть конструктор, который позволяет передавать список из адаптеров. На экране они будут отображаться в том порядке, в котором были добавлены в список.

```
val listOfAdapters = listOf(firstAdapter, secondAdapter, thirdAdapter)
val concatAdapter = ConcatAdapter(listOfAdapters)
recyclerView.adapter = concatAdapter
```

***

Если вам нужно добавить один из адаптеров не сразу, а позже, то используйте метод `addAdapter()`. Этот метод добавляет адаптер в последнюю позицию, т.е. отображаться он будет после всех остальных.

```
val concatAdapter = ConcatAdapter(firstAdapter, secondAdapter)
...
concatAdapter.addAdapter(thirdAdapter)
```

Если же требуется добавить адаптер не последним, а в определённую позицию, то в метод `addAdapter()` передайте номер позиции и сам адаптер. Метод добавит адаптер в указанную позицию, а все остальные адаптеры сместятся.

```
val concatAdapter = ConcatAdapter(firstAdapter, secondAdapter)
...
concatAdapter.addAdapter(0, thirdAdapter)
```

Обратите внимание, что номер позиции не может быть больше количества адаптеров (отсчёт начинается с нуля). В примере у нас три адаптера, каждому из которых может быть присвоена позиция **0, 1 или 2**. Если указать число выше, то вылетит ошибка.

***

Для удаления адаптера используется метод `removeAdapter()`.

```
concatAdapter.removeAdapter(firstAdapter)
```

***

Чтобы узнать сколько элементов объединил в себе `ConcatAdapter` вызовите метод `itemCount`. Количество элементов суммируется со всех добавленных адаптеров.

```
concatAdapter.itemCount
```

***

Можно получить список всех адаптеров, добавленных в `ConcatAdapter`. Для этого вызовите `adapters`, который возвращает `MutableList` со всеми адаптерами.

```
concatAdapter.adapters
```

***

Обычно если в адаптере нам надо обратиться к какой-либо позиции, мы используем метод `getAdapterPosition()` класса `ViewHolder`. При работе с `ConcatAdapter` вместо `getAdapterPosition()` следует использовать `getBindingAdapterPosition()`.


### Пример с header'ом и footer'ом

Возьмём пример, который был в разделе **ViewType**: требуется отобразить header, footer и список между ними. В таком случае у нас будет три адаптера: для header'а, footer'а и элементов списка. Можно использовать и два адаптера, если логика и внешний вид header'а и footer'а идентичны. Но для наглядности в своём примере я буду использовать три.

Для начала убедитесь, что в **build.gradle** добавлена нужная версия библиотеки **recyclerView**:

```
implementation 'androidx.recyclerview:recyclerview:1.2.0-alpha04'
```

Можно использовать и версию `1.2.0-alpha02`, но учтите, что в этой версии `ConcatAdapter` ещё носит название `MergeAdapter`.

Создадим [классы данных](https://bimlibik.github.io/posts/kotlin-data-classes/) для header'а, footer'а и элементов списка (деревья).

```
data class Header(
    val title: String,
    val color: String,
    val textSize: Float
)

data class Footer(
    val title: String,
    val color: String,
    val textSize: Float
)

data class Tree(
    val name: String,
    val description: String
)
```

Добавим макет для каждого компонента.

```
// header.xml
<?xml version="1.0" encoding="utf-8"?>
<TextView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/header"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:textStyle="bold"
    android:textAllCaps="true"
    android:text="Header"
    android:gravity="center"/>

// footer.xml
<?xml version="1.0" encoding="utf-8"?>
<TextView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/footer"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:textStyle="bold"
    android:textAllCaps="true"
    android:text="Footer"
    android:gravity="center"/>

// item_tree_simple.xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal">

    <TextView
        android:id="@+id/name"
        android:layout_width="0dp"
        android:layout_weight="1"
        android:layout_height="wrap_content"
        android:layout_margin="16dp" />

    <TextView
        android:id="@+id/description"
        android:layout_width="0dp"
        android:layout_weight="3"
        android:layout_height="wrap_content"
        android:layout_margin="16dp" />
</LinearLayout>
```

За отображение header'а будет отвечать `HeaderAdapter`.

```
class HeaderAdapter(
    private val header: Header
) : RecyclerView.Adapter<HeaderAdapter.HeaderViewHolder>() {


  override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): HeaderViewHolder {
    val view: View = LayoutInflater.from(parent.context).inflate(R.layout.header, parent, false)
    return HeaderViewHolder(view)
  }

  override fun onBindViewHolder(holder: HeaderViewHolder, position: Int) {
    holder.bindView(header)
  }

  override fun getItemCount(): Int = 1

  // ViewHolder
  class HeaderViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView)  {
    private val headerView: TextView = itemView.findViewById(R.id.header)

    fun bindView(header: Header) {
      headerView.text = header.title
      headerView.setTextColor(Color.parseColor(header.color))
      headerView.textSize = header.textSize
    }
  }

}
```

Для отображения элементов списка создадим `ListItemAdapter`.

```
class ListItemAdapter(
    private val trees: ArrayList<Tree>
) : RecyclerView.Adapter<ListItemAdapter.ListItemViewHolder>() {

  override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ListItemViewHolder {
    val view: View = LayoutInflater.from(parent.context).inflate(R.layout.item_tree_simple, parent, false)
    return ListItemViewHolder(view)
  }

  override fun onBindViewHolder(holder: ListItemViewHolder, position: Int) {
    val tree = trees[position]
    holder.bindView(tree)
  }

  override fun getItemCount(): Int = trees.size

  // ViewHolder
  class ListItemViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView)  {
    private val name: TextView = itemView.findViewById(R.id.name)
    private val description: TextView = itemView.findViewById(R.id.description)

    fun bindView(tree: Tree) {
      name.text = tree.name
      description.text = tree.description
    }
  }

}
```

Ну и наконец адаптер для отображения footer'а.

```
class FooterAdapter(
    private val footer: Footer
) : RecyclerView.Adapter<FooterAdapter.FooterViewHolder>() {

  override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): FooterViewHolder {
    val view: View = LayoutInflater.from(parent.context).inflate(R.layout.footer, parent, false)
    return FooterViewHolder(view)
  }

  override fun onBindViewHolder(holder: FooterViewHolder, position: Int) {
    holder.bindView(footer)
  }

  override fun getItemCount(): Int = 1

  // ViewHolder
  class FooterViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView)  {
    private val footerView: TextView = itemView.findViewById(R.id.footer)

    fun bindView(footer: Footer) {
      footerView.text = footer.title
      footerView.setTextColor(Color.parseColor(footer.color))
      footerView.textSize = footer.textSize
    }
  }

}
```

Теперь осталось лишь объединить всё вместе в методе `onCreate()` - для активити или в методе `onViewCreated()` - для фрагмента. Для этого создадим по одному объекту каждого из адаптеров и передадим их классу `ConcatAdapter()` в том порядке, в котором они должны быть отражены на экране.

```
val headerAdapter = HeaderAdapter(Header("Я - header!", "#283593", 25F))
val treesAdapter = ListItemAdapter(createData())
val footerAdapter = FooterAdapter(Footer("Я - footer!", "#6A1B9A", 25F))

val concatAdapter = ConcatAdapter(headerAdapter, treesAdapter, footerAdapter)
recycler_view.adapter = concatAdapter
```

Результат:

<img src="/assets/img/posts/android-recycler-view/concat-adapter-header-item-footer.png" alt="demo concat adapter" height="550"/>

Если же в `ConcatAdapter()` передать footer сразу после header'а

```
val concatAdapter = ConcatAdapter(headerAdapter, footerAdapter, treesAdapter)
recycler_view.adapter = concatAdapter
```

то результат будет таким:

<img src="/assets/img/posts/android-recycler-view/concat-adapter-header-footer-item.png" alt="demo concat adapter" height="550"/>


***

## Полезные ссылки

**Общие ссылки по теме:**  
[Create a List with RecyclerView](https://developer.android.com/guide/topics/ui/layout/recyclerview "developer.android.com") - гайд из официальной документации.  
[RecyclerView](https://developer.android.com/reference/kotlin/androidx/recyclerview/widget/package-summary "developer.android.com") - документация по классу (androidx).  
[Recyclerview - Release Notes](https://developer.android.com/jetpack/androidx/releases/recyclerview "developer.android.com") - информация о выходе новых версий.  
[Using the RecyclerView](https://guides.codepath.com/android/using-the-recyclerview "guides.codepath.com") - гайд от codepath.

**Кастомизация:**  
[Having multiple lists in a single RecyclerView](https://github.com/masudias/dynamic-recyclerview/wiki "github.com") - гайд по использованию нескольких списков в одном `RecyclerView`.

**Адаптеры:**  
[ConcatAdapter](https://developer.android.com/reference/kotlin/androidx/recyclerview/widget/ConcatAdapter "developer.android.com") - официальная документация.  

**Код:**  
[RecyclerView](https://github.com/Bimlibik/Examples/tree/master/RecyclerView "github.com") - полный код всех примеров из данной статьи.  
[MergeAdapter-sample](https://github.com/Kotlin-Android-Open-Source/MergeAdapter-sample/tree/master/app/src/main/java/com/hoc/mergeadapter_sample "github.com") - пример реализации `ConcatAdapter` от **Kotlin Android Open Source**.  
[Concat Adapter Android Example](https://github.com/MindorksOpenSource/ConcatAdapter-Android-Example "github.com") - пример реализации `ConcatAdapter` от **Mindorks**.  
