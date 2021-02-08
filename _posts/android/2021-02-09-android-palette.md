---
title: Определение доминирующих цветов с помощью Palette
author: Leslie M.
date: "2021-02-09 03:24"
categories: [Android, UI]
tags: [android, palette, glide, colors, dominant color, ui]
---

Библиотека [Palette][palette-doc] определяет основные цвета изображения, что в свою очередь позволяет динамически создавать и изменять цветовую схему приложения на основе заданного изображения.

## Подключение библиотеки

Достаточно добавить следующую зависимость в файл `build.gradle` уровня приложения:
```
implementation 'androidx.palette:palette-ktx:1.0.0'
```


## Создание палитры

Для создания экземпляра `Palette` потребуется изображение в формате `Bitmap`, которое необходимо передать методу палитры `from(Bitmap bitmap)`. Затем палитра может быть сгенерирована синхронно или асинхронно.

```
// синхронно
fun createPalette(bitmap: Bitmap): Palette = Palette.from(bitmap).generate()

// асинхронно
fun createPalette(bitmap: Bitmap) {
    Palette.from(bitmap).generate { palette ->
        ...
    }
}
```

Если вы планируете постоянно или достаточно часто создавать экземпляры `Palette`, то возможно стоит рассмотреть вариант с кэшированием созданных экземпляров. Это позволит реализовать более отзывчивый интерфейс.

Также не рекомендуется создавать экземпляры `Palette` в основном потоке.


## Настройка палитры

При помощи [`Palette.Builder`][palette-builder-doc] можно настроить экземпляр палитры. Например, выбрать количество цветов в палитре, какая область изображения будет использоваться для создания палитры, какие цвета разрешены в палитре.

Методы билдера для настройки:
- `addFilter()` - добавляет фильтр, который указывает на то, какие цвета разрешены в итоговой палитре. Методу необходимо передать свой [`Palette.Filter`][palette-filter-doc], у которого требуется изменить метод `isAllowed()` - он определяет какие цвета допустимы/недопустимы в палитре.
- `maximumColorCount()` - устанавливает максимальное количество цветов в палитре. Значение по умолчанию - 16, а оптимальное значение зависит от исходного изображения. Для пейзажей оптимальные значения находятся в диапазоне от 8 до 16, в то время как изображения с лицами обычно имеют значения от 24 до 32. Чем больше количество цветов, тем дольше будет создаваться палитра.
- `setRegion()` - позволяет указать то, какую область изображения следует использовать при создании палитры.
- `addTarget()` - позволяет выполнить собственное сопоставление цветов, добавив [`Target`][palette-target] - специальный класс, который даёт возможность настраивать выбор цветов при создании палитры.


## Использование цветов

Библиотека пытается извлекать шесть основных цветовых профилей:
- Light Vibrant
- Vibrant
- Dark Vibrant
- Light Muted
- Muted
- Dark Muted

Чтобы получить цвет, связанный с каким-либо из этих профилей, используется метод `get<Profile>Color()`, где `<Profile>` заменяется именем одного из профилей. Например, для получения цветового профиля **Dark Vibrant** используется метод `getDarkVibrantColor()`. При использовании этих методов им необходимо предоставлять значения цветов по умолчанию.

Второй способ получения цвета одного из этих профилей - с помощью объекта [`Palette.Swatch`][palette-swatch]. Он генерируется классом `Palette` для каждого цветового профиля. Для получения цвета используется метод `get<Profile>Swatch()`. В отличие от первого варианта, этому методу не требуется значение по умолчанию и он может вернуть `null`, если выбранный цветовой профиль отсутствует в изображении.

```
// цвет профиля Vibrant
val vibrantColor = palette.vibrantSwatch?.rgb
```

Также, объект `Palette.Swatch` на основе цвета профиля создаёт рекомендуемый цвет для заголовка и основного текста, которые гарантированно будут иметь достаточный контраст.

Помимо вышеперечисленных цветов генерируется **доминирующий** цвет изображения. Скорее всего это единственный цвет, профиль которого гарантированно будет сгенерирован. Сколько я не экспериментировала, но мне ни разу не возвращалось значение `null`. Получить доминирующий цвет можно при помощи аналогичных методов: `getDominantColor()` или `getDominantSwatch()`.

С помощью метода `getSwatches()` можно получить доступ ко всем цветам в палитре.


## Glide

`Palette` можно объединить с библиотекой **Glide** и тогда определение доминирующих цветов будет происходить при загрузке изображения.

Для инициализации `Palette` необходимо получить изображение в формате bitmap. Ход действий будет отличаться в зависимости от того, какую версию **Glide** вы используете - 3 или 4.

### Glide v3

Добавьте библиотеку в файл `build.gradle` уровня приложения:

```
implementation 'com.github.bumptech.glide:glide:3.8.0'
```

Для работы с `Palette` нам потребуется создать дополнительные классы. Инструкция на Java есть в [официальной вики][glide-wiki] на GitHub. Адаптируем под Kotlin.

Класс `PaletteBitmap`:

```
data class PaletteBitmap(val palette: Palette, val bitmap: Bitmap)
```

Класс `PaletteBitmapResource`:

```
class PaletteBitmapResource(
    private val paletteBitmap: PaletteBitmap,
    private val bitmapPool: BitmapPool
) : Resource<PaletteBitmap> {

    override fun get() = paletteBitmap

    override fun getResourceClass() = PaletteBitmap::class.java

    override fun getSize() = Util.getBitmapByteSize(paletteBitmap.bitmap)

    override fun recycle() = bitmapPool.put(paletteBitmap.bitmap)
}
```

Класс `PaletteBitmapTranscoder`:

```
class PaletteBitmapTranscoder(
    private val context: Context
) : ResourceTranscoder<Bitmap, PaletteBitmap> {

    private val bitmapPool: BitmapPool = Glide.get(context).bitmapPool

    override fun transcode(
      toTranscode: Resource<Bitmap>,
      options: Options
    ): Resource<PaletteBitmap> {
        val bitmap = toTranscode.get()
        val palette = Palette.from(bitmap).generate()
        val result = PaletteBitmap(palette, bitmap)
        return PaletteBitmapResource(result, bitmapPool)
    }
}
```

Мы создали свой транскодер, который будет преобразовывать **Bitmap** в другой тип ресурса - `PaletteBitmapResource`.

Реализация будет выглядеть следующим образом:

```
Glide.with(context)
    .load(url)
    .asBitmap()
    .transcode(PaletteBitmapTranscoder(context), PaletteBitmap::class.java)
    .into(new ImageViewTarget<PaletteBitmap>(imageView) {
        @Override
        protected void setResource(PaletteBitmap resource) {
            if(resource != null) {
                super.view.setImageBitmap(resource.bitmap)
                val palette = resource.palette
                toolbarColor = palette.getMutedColor(ContextCompat.getColor(context, R.color.default))
            }          
        }
    })
```


### Glide v4

Добавьте библиотеку в файл `build.gradle` уровня приложения. Актуальную версию проверяйте на [GitHub][glide].

```
implementation 'com.github.bumptech.glide:glide:4.12.0'
kapt 'com.github.bumptech.glide:compiler:4.12.0'
```

Здесь же проверьте, что у вас включен плагин `kotlin-kapt`:

```
apply plugin: 'kotlin-kapt'
```

Создаём класс, расширяющий `AppGlideModule`:

```
@GlideModule
class PaletteGlideModule : AppGlideModule()
```

Класс может быть пустым, главное добавить аннотацию и унаследоваться от `AppGlideModule`. Если всё сделано верно, то модуль будет обнаружен и мы сможем запускать загрузку изображений с `GlideApp.with()` вместо `Glide.with()`.

Это всё, что нужно сделать. Далее просто реализуем:

```
GlideApp.with(context)
    .asBitmap()
    .load(url)
    .listener(object : RequestListener<Bitmap> {
        override fun onLoadFailed(
            e: GlideException?,
            model: Any?,
            target: Target<Bitmap>?,
            isFirstResource: Boolean
        ): Boolean {
            Log.i(TAG, "Error while loading picture: $e")
            return false
        }

        override fun onResourceReady(
            resource: Bitmap?,
            model: Any?,
            target: Target<Bitmap>?,
            dataSource: DataSource?,
            isFirstResource: Boolean
        ): Boolean {
            if (resource != null) {
                val palette: Palette = Palette.from(resource).generate()
                color = palette.darkMutedSwatch?.rgb ?: R.color.default
            }
            return false
        }
}).into(imageView)
```


## Полезные ссылки

[Selecting Colors with the Palette API][palette-doc] - официальная документация.  



[glide-wiki]: https://github.com/bumptech/glide/wiki/Custom-targets#palette-example
[glide]: https://github.com/bumptech/glide
[palette-doc]: https://developer.android.com/training/material/palette-colors
[palette-builder-doc]: https://developer.android.com/reference/androidx/palette/graphics/Palette.Builder#addFilter(android.support.v7.graphics.Palette.Filter)
[palette-filter-doc]: https://developer.android.com/reference/androidx/palette/graphics/Palette.Filter
[palette-target]: https://developer.android.com/reference/androidx/palette/graphics/Target
[palette-swatch]: https://developer.android.com/reference/androidx/palette/graphics/Palette.Swatch#getTitleTextColor()
