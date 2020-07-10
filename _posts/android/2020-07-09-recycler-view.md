---
title: RecyclerView
author: Leslie M.
date: "2020-07-09 14:55:00 +0800"
categories: [Android, UI]
tags: [android, ui, recyclerView]
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

## Полезные ссылки

[Create a List with RecyclerView](https://developer.android.com/guide/topics/ui/layout/recyclerview "developer.android.com") - гайд из официальной документации.  
[RecyclerView](https://developer.android.com/reference/kotlin/androidx/recyclerview/widget/package-summary "developer.android.com") - документация по классу (androidx).  
[Using the RecyclerView](https://guides.codepath.com/android/using-the-recyclerview "guides.codepath.com") - гайд от codepath.
