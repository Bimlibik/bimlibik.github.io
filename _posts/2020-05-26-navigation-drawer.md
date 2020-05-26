---
title: Navigation Drawer
author: Leslie M.
date: "2020-05-26 18:20:00 +0800"
categories: [Android, UI]
tags: [android, ui, navigation]
---

Nawigation drawer - это главное меню приложения, которое выдвигается слева направо
при нажатии пользователем на значок "гамбургера". Либо свайпом слева направо.
Его еще называют "шторкой" и в открытом виде выглядит так:

![navigation drawer open](/assets/img/posts/navigation-drawer/navigation-drawer-open.png)

Когда этот элемент интерфейса только только появился, необходимо было
осуществлять много манипуляций по его добавлению на экран (если не учитывать
наличие специального шаблона). Ради интереса можно ознакомиться со [статьей](http://developer.alexanderklimov.ru/android/navigation_drawer_activity.php "developer.alexanderklimov.ru"), в которой описывается весь этот нелегкий путь.

Но вот на Google I/O 2018 была предложена совершенно новая концепция навигации
по приложению, для которой уже весной 2019 года выпустили стабильную версию.
Первоначально носила название **Navigation Architecture Component**, теперь же
именуется **Jetpack Navigation**.

Главная цель - создание приложений по типу **singleActivity**.

С выходом **Jetpack Navigation** добавление шторки в приложении значительно
упростилось, уменьшилось количество кода и настроек. Ну и, конечно же, был
обновлен шаблон Navigation Drawer Activity.

В примере ниже рассмотрим добавление шторки вручную.

***

## Алгоритм добавления шторки в интерфейс

### Зависимости

```
// Java language implementation
  implementation "androidx.navigation:navigation-fragment:$nav_version"
  implementation "androidx.navigation:navigation-ui:$nav_version"

// Kotlin
  implementation "androidx.navigation:navigation-fragment-ktx:$nav_version"
  implementation "androidx.navigation:navigation-ui-ktx:$nav_version"
```

В этом же файле `build.gradle` должно быть:

```
android {
  ...

  kotlinOptions {
        jvmTarget = "1.8"
  }
}
```

Иначе словите ошибку:
![jvm-error](/assets/img/posts/navigation-drawer/error-jvm.jpg)

### Фрагменты
Так как основная цель **Jetpack Navigation** - создание приложений по типу
**singleActivity**, основной контент будет отображаться во фрагментах.

На этом этапе нужно создать классы фрагментов и макеты к ним. В моем приложении
будет три фрагмента - `MainFragment`, `SettingsFragment`, `AboutFragment`. По
внутреннему содержанию они идентичны (все отображают `TextView`), а значит будет
достаточно создать один файл разметки.

**Макет для MainFragment, SettingsFragment, AboutFragment - `fragment_page.xml`**

```
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <TextView
        android:id="@+id/tv_info"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal|center_vertical"
        android:textSize="30sp" />
</FrameLayout>
```

**Код для класса MainFragment**  
Для остальных фрагментов код идентичен, разница только в отображаемом тексте.
```
import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import androidx.fragment.app.Fragment
import kotlinx.android.synthetic.main.fragment_page.*

class MainFragment : Fragment() {

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? =
        inflater.inflate(R.layout.fragment_page, container, false)

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        // каждому фрагменту установите уникальный текст для наглядности
        tv_info.text = "Main Content"
    }
}
```

### Навигация

Теперь для созданных фрагментов нужно выстроить навигацию.

Добавляем новую директорию в ресурсы через контекстное меню **New ->
Android Resource Directory**. В новом окне полю **Resource type** задать
значение **navigation**:
![navigation directory](/assets/img/posts/navigation-drawer/navigation_directory.jpg)

В созданной папке добавляем новый файл `nav_graph.xml`.

Проще всего спроектировать навигацию через визуальный конструктор. В левом
верхнем углу находится кнопка "New Destination", с ее помощью можно добавить в
граф все вышесозданные фрагменты.

При выделении фрагмента в визуальном редакторе появляется **кружок** в центре
правого края. С помощью него выстраивается цепочка переходов от фрагмента к
фрагменту.

![create new action](/assets/img/posts/navigation-drawer/navigation_graph_actions.jpg)

В результате в `nav_graph.xml` должен получится такой код:

```
<?xml version="1.0" encoding="utf-8"?>
<navigation
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/nav_graph"
    app:startDestination="@id/main_fragment">

    <fragment
        android:id="@+id/main_fragment"
        android:name="com.foxy.navigationdrawer.MainFragment"
        android:label="MainContent" >
        <action
            android:id="@+id/action_main_fragment_to_about_fragment2"
            app:destination="@id/about_fragment" />
        <action
            android:id="@+id/action_main_fragment_to_settings_fragment"
            app:destination="@id/settings_fragment" />
    </fragment>
    <fragment
        android:id="@+id/about_fragment"
        android:name="com.foxy.navigationdrawer.AboutFragment"
        android:label="About" />
    <fragment
        android:id="@+id/settings_fragment"
        android:name="com.foxy.navigationdrawer.SettingsFragment"
        android:label="Settings" />
</navigation>
```

Обратите внимание на атрибут `android:label` - он отвечает за заголовок,
который будет отображаться в тулбаре для каждого фрагмента.

### Меню для шторки

По аналогии с навигацией, нужно добавить папку в ресурсы для меню и файл
`nav_drawer_menu.xml`:

```
<menu
    xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/settings_fragment"
        android:title="Settings" />
    <item
        android:id="@+id/about_fragment"
        android:title="About" />
</menu>
```

Обратите внимание на идентификаторы в меню - для корректной работы они должны
быть идентичны идентификаторам из `nav_graph.xml`.

### Header для шторки

Создается в папке layouts, дизайн на ваше усмотрение. Вот, что получилось у меня
(`nav_header.xml`):

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="192dp"
    android:padding="16dp"
    android:orientation="vertical"
    android:gravity="bottom"
    android:background="@color/colorPrimary">

    <ImageView
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="3"
        android:src="@drawable/ic_launcher_foreground"/>
</LinearLayout>
```

### Объединяем все вместе в макете activity

Корневым элементом макета `activity` обязательно должен быть [DrawerLayout](https://developer.android.com/reference/androidx/drawerlayout/widget/DrawerLayout?hl=ru "developer.android.com"),
так как именно он позволяет шторке выдвигаться из края экрана. Внутри DrawerLayout'а
объявляется основное содержимое экрана: toolbar, контейнер для фрагментов и сама
шторка - NavigationView.

```
<?xml version="1.0" encoding="utf-8"?>
<androidx.drawerlayout.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <androidx.appcompat.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:minHeight="?attr/actionBarSize"
            android:background="@color/colorPrimary"
            app:title="@string/app_name" />

        <fragment
            android:id="@+id/fragment_container"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:name="androidx.navigation.fragment.NavHostFragment"
            app:defaultNavHost="true"
            app:navGraph="@navigation/nav_graph" />
    </LinearLayout>

    <com.google.android.material.navigation.NavigationView
        android:id="@+id/nav_view"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        android:fitsSystemWindows="true"
        app:headerLayout="@layout/nav_header"
        app:menu="@menu/nav_drawer_menu" />
</androidx.drawerlayout.widget.DrawerLayout>
```
Элемент `<fragment>` является контейнером для наших фрагментов. Подробнее
об особо важных атрибутах:
- Значение атрибута `android:name="androidx.navigation.fragment.NavHostFragment"`
говорит о том, что данный элемент в разметке будет являться хостом для фрагментов.
Указывается обязательно в таком виде без изменений, так как хост должен быть
производным от NavHostFragment, который в свою очередь обрабатывает смену
фрагментов местами.
- `app:defaultNavHost="true"` - позволяет перехватывать нажатие на системную
кнопку "Назад", т.е. не нужно ее дополнительно отслеживать и обрабатывать.
- `app:navGraph="@navigation/nav_graph"` - связывает NavHostFragment с
созданным нами графом навигации.

Для NavigationView устанавливаем ранее подготовленные файлы - header и меню, а
также с помощью атрибута `android:layout_gravity` указываем с какой стороны
она будет выезжать.

Так как в макете мы объявили реализацию `toolbar`, требуется отключить
стандартный `actionBar`. Для этого заходим в `values/styles.xml` и меняем
`DarkActionBar` на `NoActionBar`. Должно получится так:

```
<style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
```

### Подключение в классе MainActivity

```
import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import androidx.navigation.findNavController
import androidx.navigation.ui.*
import kotlinx.android.synthetic.main.activity_main.*

class MainActivity : AppCompatActivity() {

    lateinit var appBarConfig: AppBarConfiguration

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // устанавливаем свою реализацию тулбара
        setSupportActionBar(toolbar)

        val navController = findNavController(R.id.fragment_container)

        // конфигурация тулбара
        appBarConfig = AppBarConfiguration(navController.graph, drawer_layout)
        setupActionBarWithNavController(navController, appBarConfig)

        // метод, который связывает шторку с навигацией
        nav_view.setupWithNavController(navController)
    }

    // отслеживает клик по иконке гамбургера и стрелке UP
    override fun onSupportNavigateUp(): Boolean {
        val navController = findNavController(R.id.fragment_container)
        return navController.navigateUp(appBarConfig) || super.onSupportNavigateUp()
    }
}
```

`AppBarConfiguration` - устанавливает в тулбаре иконку "гамбургера" и меняет
ее на стрелку **UP**, если мы находимся не на главном экране приложения. Вместо
графа в качестве первого параметра можно указать список фрагментов, у которых
**не будет** происходить смена иконки "гамбургера" на стрелку **UP**:

```
appBarConfig = AppBarConfiguration(setOf(R.id.main_fragment), drawer_layout)
```

Запускаем приложение и наслаждаемся результатом.

***

## Цвет иконки

Добавляем в `values/styles.xml` следующий код:

```
<style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
    <!-- Customize your theme here. -->
    <item name="colorPrimary">@color/colorPrimary</item>
    <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
    <item name="colorAccent">@color/colorAccent</item>
    <item name="drawerArrowStyle">@style/DrawerIconStyle</item>
</style>

<style name="DrawerIconStyle" parent="Widget.AppCompat.DrawerArrowToggle">
    <item name="spinBars">true</item>
    <item name="color">@color/colorWhite</item>
</style>
```

Значение `colorWhite` задано в файле `colors.xml`. Соответственно можно указать
любой цвет по вкусу.

## Полезные ссылки

Документация - [Update UI components with NavigationUI](https://developer.android.com/guide/navigation/navigation-ui?hl=ru#add_a_navigation_drawer "developer.android.com").

StartAndroid - [Navigation. NavigationUI](https://startandroid.ru/ru/courses/architecture-components/27-course/architecture-components/560-urok-27-navigation-navigationui.html "startandroid.ru").

Полный код текущего проекта [здесь](https://github.com/Bimlibik/Examples/tree/master/NavigationDrawer "github.com").
