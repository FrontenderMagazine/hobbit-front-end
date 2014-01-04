* Три версии одного и того же сайта
* Управление состоянием
* Места съёмки
* Ресурсы
* Анимации
* Выводы

<section class="byline"></section>
В [нашей первой статье][1] о разработке эксперимента для Google Chrome
[A Journey Through Middle-earth][2] мы сконцентрировались на работе с  WebGL
для мобильных устройств. В этой статье мы рассмотрим все задачи, проблемы и их
решения, к которым мы пришли в процессе работы над остальной частью фронтенда
на HTML5.

## Три версии одного и того же сайта {#toc-three-versions}

Давайте начнём с вопроса об адаптации этого сайта-эксперимента для работы и с
настольными компьютерами, и с мобильными устройствами с точки зрения изменения
размеров экрана и возможностей устройств.

Весь проект основан на «кинематографическом» стиле, где мы хотели побольше
использовать ландшафтную (горизонтальную) ориентацию экрана, чтобы сохранить
сходство с фильмом. Исходя из того, что большая часть сайта состоит из
миниигр, то не имеет смысла, чтобы они тоже выходили за эти рамки.

Можно привести посадочную страницу сайта, как пример того, как мы адаптировали
дизайн для разных размеров экрана.

<figure>![][3]
<figcaption>Орлы только что принесли нас на посадочную страницу.
</figcaption></figure>

У сайта есть три разных режима: настольный, планшет и мобильный. Это не только
для того, что бы управлять версткой, но и для того, чтобы управлять временно
загружаемыми ресурсами и разнообразными способами оптимизации
производительности. Для устройств с разрешением экрана большим, чем у 
настольных компьютеров и планшетов, но производительностью хуже, чем у
телефонов было довольно сложно подобрать универсальный набор правил.

Мы используем данные user-agent для определения мобильных устройств и размер
окна для вычленения из них планшетов (645px и выше). В принципе каждый режим
может отображать все разрешения потому, что верстка основана на media queries
или относительное позиционирование (в процентах) с помощью JavaScript.

Так как в нашем случае дизайн не привязан к жёсткой сетке и контент очень
меняется от модуля к модулю, то сценарии и контрольные точки (breakpoints) сильно
зависят от специфического поведения конкретных элементов. Неоднократно случалось
так, что после того, как всё было идеально завёрстано с помощью миксинов sass и
media-queries нам необходимо было добавить эффект, привязанный к положению 
курсора или динамических объектах, и приходилось всё переписывать на JavaScript.

Кроме этого мы добавляем класс с текущим режимом в родительском тэге для
использования в стилях (пример в SCSS):

    .loc-hobbit-logo {
    
        // Значения по умолчанию.
    
        .desktop & {
            // Изменения только для режима «настольный».
        }
    
        .tablet &, .mobile & {
            
            // Обычно здесь другие ресурсы для планшетов и мобильных.
                @media screen and (max-height: 760px), (max-width: 760px) {
                // Специфические стили для этих контрольных точек.
            }
                @media screen and (max-height: 570px), (max-width: 400px) {
                // Специфические стили для этих контрольных точек.
            }
        }
    }
        

Мы поддерживаем все размеры экрана до 360x320, что оказалось довольно непростым
при создании этого сайта. Для компьютеров у нас предусмотрен минимальный размер,
после которого мы начинаем показывать полосы прокрутки, так как хотим, чтобы
по возможности пользователи просматривали сайт в большем разрешении. Для
мобильных разрешена и ландшафтная, и портретная ориентация, кроме интерактивных
мест, где пользователей просят повернуть устройство и просматривать в 
ландшафтной ориентации. Основной причиной этого явилось мнение о том, что в
портретном режиме сайт не вызывает такого сильного эффекта. В любом случае сайт 
хорошо адаптирован под любой размер экрана, поэтому мы оставили эту возможность.


* **События DeviceOrientation** Вёрстка контента управляется контрольными точками
и CSS, но также нам нужно знать, когда происходит изменения ориентации устройства
в JavaScript, чтобы приостанавливать игры и анимации, и сохранять при этом
правильное состояние. Оказывается, что нельзя так уж полагаться на значение
window.orientation, потому что оно не стандартизовано и неодинаково для разных
устройств. Вместо этого лучше следить за событием и смотреть на
`window.innerWidth` и `window.innerHeight`, чтобы определить ориентацию.*

Хочется напомнить, что не стоит путать вёрстку не стоит смешивать с определением
возможностей вроде способа ввода, ориентации устройства, сенсоров и т.д. Всё это
должно быть в разных режимах и работать в них. Например, поддержка и мыши, и 
тач-событий. Также поддержка ретина-дисплеев для улучшения качества (но в
основном, для улучшения производительности — иногда производительность важнее
качества). Так, canvas-элементы генерируются в половину разрешения на
ретина-дисплеях, иначе пришлось бы рендерить в четыре раза больше пикселей.

Можно легко испытать все размеры прямо в браузере [эмулируя][4] устройство в 
Chrome DevTools. При переключении режима между мобильным, планшетом и настольным
компьютером нужно перезагружать сайт для получения правильных зависимостей и 
настроек.

Мы часто пользовались этим инструментом во время разработке, особенно в Chrome
Canary, в котором он предлагает улучшенные возможности и множество предустановок.
Это хороший способ проверить дизайн. При этом мы всё тестировали сайт на реальных
устройствах. В первую очередь потому, что сайт рассчитан на полноэкранный режим.
Страницы с вертикальной прокруткой скрывают браузерный скролл в большистве
случаев (разве что в Safari на iOS7 с этим проблемы сейчас), и надо было всё
уместить независимо от этого. Мы использовали одну из предустановок в эмуляторе
и изменяли размер экрана, чтобы симулировать потерю доступного места.
Тестирование на реальных устройствах также очень важно для отслеживания затрат
памяти и измерения производительности.


## Handling the state {#toc-state}

After the [landing page][2] we land at the map of Middle-earth. Did you notice
the URL changing? The site is a single page application that uses the
[History API][5] to handle [routing][6].

Each section of the site is its own object inheriting a boilerplate of
functionality such as DOM-elements, transitions, loading of assets, disposing 
etc. When you explore different parts of the site, sections are initiated, 
elements are added to and removed from the DOM and assets for the current 
section are loaded.

Since the user can hit the browser’s back button or navigate via the menu at
any time, everything that is created needs to be disposed of at some point. 
Timeouts and animations need to be stopped and discarded or they will cause 
unwanted behaviour, errors, and memory leaks. This is not always an easy task, 
especially when deadlines are approaching and you need to get everything in 
there as fast as possible.

Keep calm and add those event listeners. Make a practice of adding a dispose
function to every object. Watch out for leaving timers and tweens behind. If 
tweening, use the equivalent of`TweenMax.killTweensOf(foo)` or save references
and stop them from triggering callbacks. Remove runtime added DOM elements. Use 
profiling tools regulary to keep an eye on the memory consumption and leaks.

## Showing off the locations {#toc-showing-off} To show off the beautiful
settings and the characters of Middle-earth we built a modular system of image 
and text components that you can drag or swipe horizontally. We haven’t enabled 
a scrollbar here since we want to have different speeds on different ranges, 
like in image sequences where you stop the motion sideways until the clip has 
played out.<figure>

![][7]<figcaption>[Thranduil's Hall][8] timeline</figcaption></figure>
### The timeline When development started we didn’t know the content of the
modules for each location. What we knew was that we wanted a templated way of 
showing different types of media and information in a horizontal timeline that 
would give us the freedom to have six different location presentations without 
having to rebuild everything six times. To manage this we created a timeline 
controller that handle the panning of its modules based on settings and the 
modules’ behaviours.

### Modules and behaviour components The different modules we added support for
are image-sequence, still image, parallax scene, focus-shift scene and text. The
parallax scene module has an opaque background with a custom number of layers 
that listens to the viewport progress for exact positions. The focus-shift scene
is a variant of the parallax bucket, with the addition that we use two images 
for each layer which fades in and out to simulate a focus change. We tried to 
use the blur filter, but it’s still to expensive, so we’ll wait for CSS shaders 
for this. The content in the text module is drag-enabled with the TweenMax 
plugin

[Draggable][9]. You can also use the scrollwheel or two-finger swipe to scroll
vertically. Note the[throw-props-plugin][10] that adds the fling-style physics
when you swipe and release. The modules can also have different behaviours that 
are added as a set of components. They all have their own target selectors and 
settings. Translate to move an element, scale to zoom, hotspots for info overlay,
debug metrics for testing visually, a start-title overlay, a flare layer, and 
some more. These will be appended to the DOM or controlling their target element
inside the module. With this in place we can create the different locations with
just a[config file][11] that defines what assets to load and setup the
different kinds of modules and components.
### Image sequences

The most challenging of the modules from a performance and a download-size
aspect is the image sequence. There’s a bunch to read about this[topic][12]. On
mobile and tablets we replace this with a still image. It’s too much data to 
decode and store in memory if we want decent quality on mobile. We tried 
multiple alternative solutions; using a background image and a spritesheet first,
but it led to memory problems and lag when the GPU needed to swap between 
spritesheets. Then we tried swapping img elements, but it was also too slow. 
Drawing a frame from a spritesheet to a canvas was the most performant, so we 
began optimizing that. To save computation time each frame, the image data to 
write into the canvas is pre-processed via a temporary canvas and saved with 
putImageData() to an array, decoded and ready to use. The original spritesheet 
can then be garbage collected, and we store only the minimum amount of data 
needed in memory. Maybe it’s actually less to store undecoded images, but we get
better performance while scrubbing the sequence this way. The frames are pretty 
small, just 640x400, but those will just be visible during scrubbing. When you 
stop, a high-res image loads and quickly fades in.

        var canvas = document.createElement('canvas');
        canvas.width = imageWidth;
        canvas.height = imageHeight;
        
        var ctx = canvas.getContext('2d');
        ctx.drawImage(sheet, 0, 0);
        
        var tilesX = imageWidth / tileWidth;
        var tilesY = imageHeight / tileHeight;
        
        var canvasPaste = canvas.cloneNode(false);
        canvasPaste.width = tileWidth;
        canvasPaste.height = tileHeight;
        
        var i, j, canvasPasteTemp, imgData, 
        var currentIndex = 0;
        var startIndex = index * 16;
        for (i = 0; i 
        
        The sprite-sheets are generated with [Imagemagick][13]. Here is a simple [example on GitHub][14] that shows how to create a spritesheet of all images inside a folder.
        
        
        
        ### Animating the modules
        
        
        To place the modules on the timeline, a hidden representation of the timeline, displayed offscreen, keeps track on the ‘playhead’ and the width of the timeline. This can be done with just code, but it was good with a visual representation when developing and debugging. When running for real it’s just updated on resize to set dimensions. Some modules fills the viewport and some have their own ratio, so it was a little tricky to scale and position everything in all resolutions so everything is visible and not cropped too much. Each module has two progress indicators, one for the visible position on screen and one for the duration of the module itself. When making parallax movement it’s often hard to calculate start- and end-position of objects to sync with the expected position when it’s in view. It’s good to know exactly when a module enters the view, plays its internal timeline and when it animates out of view again.
        
        <figure></figure>
        
        Each module has a subtle black layer on top that adjusts its opacity so it’s fully transparent when it’s in the center position. This helps you to focus on one module at a time, which enhances the experience.
        
        
        
        ### Page performance
        
        Moving from a functioning prototype to a jank-free release version means going from guessing to knowing of what happens in the browser. This is where Chrome DevTools is your best friend.
        
        We have spent quite a lot of time optimising the site. Forcing hardware-acceleration is one of the most important tools of course to get smooth animations. But also hunting 
        
        [colorful columns][15] and red rectangles in Chrome DevTools. There are many good articles about the topics, and you should read them [all][16]. The reward for removing skipping frames is instant, but so is the frustration when they return again. And they will. It's an ongoing process that needs iterations.
        
        Watch the layers panel (only in Canary) and the “paint rectangles” in Chrome DevTools. If, for example, child elements need to be updated per frame and be painted you should investigate if it’s faster to rearrange the layers to minimize the areas that need to be painted as much as possible.
        
        
        
        I like to use TweenMax from Greensock for tweening properties, transforms and CSS. Think in containers, visualise your structure as you add new layers. Keep in mind that existing transforms can be overwritten by new transforms. The translateZ(0) that forced hardware acceleration in your CSS class is replaced by a 2D matrix if you tween 2D values only. To keep the layer in acceleration mode in those cases, use the property “force3D:true” in the tween to make a 3D matrix instead of a 2D matrix. It’s easy to forget when you combine CSS and JavaScript tweens to set styles.
        
        
        
        Don’t force hardware acceleration where it’s not needed. GPU memory can quickly fill up and cause unwanted results when you want to hardware-accelerate many containers, especially on iOS where memory have more constraints. To load smaller assets and scale them up with css and disable some of the effects in mobile mode made huge improvements.
        
        
        
        [Memory leaks][17] was another field we needed to improve our skills in. When navigating between the different WebGL experiences a lot of objects, materials, textures and geometry are created. If those are not ready for garbage collection when you navigate away and remove the section they will probably cause the device to crash after a while when it runs out of memory.
        
        <figure>
        
        ![][18]<figcaption>Exiting a section with a failing dispose function.</figcaption></figure><figure>![][19]<figcaption>Much better!</figcaption></figure>
        To find the leak it was pretty straight forward workflow in DevTools, recording the timeline and capturing heap snapshots. It’s easier if there are specific objects, like 3D geometry or a specific library, that you can filter out. In the example above it turned out that the 3D scene was still around and also an array that stored geometry was not cleared. If you find it hard to locate where the object lives, there is a nice feature that let you view this called [retaining paths][20]. Just click the object you want to inspect in the heap snapshot and you get the information in a panel below. Using a good structure with smaller objects helps when locating your references.
        
        <figure>
        
        ![][21]<figcaption>The scene was referenced in the EffectComposer.</figcaption></figure>
        In general, it's healthy to think twice before you manipulate the DOM. When you do, think about efficiency. Don't manipulate the DOM inside a game loop if you can help it. Store references in variables for reuse. If you need to search for an element, use the shortest route by storing references to strategic containers and searching inside the nearest ancestor element.
        
        
        
        Delay reading dimensions of newly added elements or when removing/adding classes if you experience layout bugs. Or make sure [Layout is triggered][22]. Sometimes the browser batch changes to styles, and will not update after the next layout trigger. This can really be a big problem sometimes, but it’s there for a reason, so try to learn how it’s working behind the scenes and you will gain a lot.
        
        
        
        ### Fullscreen
        
        
        When available, you have the option to put the site in fullscreen-mode in the menu via the Fullscreen API. But on devices there is also the browsers decision to put it into fullscreen. Safari on iOS had previously a hack to let you control that, but that is not available anymore so you have to prepare your design to work without it when making a non-scrolling page. We can probably expect updates on this in future updates, since it has broke a lot of web-apps.
        
        
        
        ## Assets {#toc-assets}
        
        <figure>
        
        ![][23]<figcaption>Animated instructions for the experiments.</figcaption></figure>
        Throughout the site we have a lot of different types of assets, we use images (PNG and JPEG), SVG (inline and background), spritesheets (PNG), custom icon fonts and Adobe Edge animations. We use PNGs for assets and animations (spritesheets) where the element can't be vector based, otherwise we try to use SVGs as much as possible.
        
        
        
        The vector format means no loss of quality, even if we scale it. 1 file for all devices.
        

*   Small file size.
*   We can animate each part separately (perfect for advanced animations). As
        an example we hide the "subtitle" of the Hobbit logo (the desolation of Smaug) 
        when it's scaled down.
     
*   It can be embedded as an SVG HTML tag or used as a background-image with no
        extra loading (it’s loaded the same time as the html page
        ).

Icon typefaces have the same advantages as SVG when it comes to scalability and
are used instead of SVG for small elements like icons on which we only need to 
be able to change the colour (hover, active, etc.). The icons are also very easy
to reuse, you just need to set the CSS "content" property of an element.

## Animations {#toc-animations}

In some cases animating SVG elements with code can be very time consuming,
especially when the animation needs to be changed a lot during the design 
process. To improve the workflow between designers and developers we use Adobe 
Edge for some animations (the instructions before the games). The animation 
workflow is really close to Flash and that helped the team but there are a few 
drawbacks, especially with integrating the Edge animations in our asset loading 
process since it comes with it’s own loaders and implementation logic.

I still feel we have a long way to go before we have a perfect workflow for
handling assets and handmade animations on the web. We’re looking forward to 
seeing how tools like Edge will evolve. Feel free to add suggestions on other 
animation tools and workflows in the comments.

## Conclusion {#toc-conclusion} Now when all the parts of the project are
released and we look at the final result I must say we are quite impressed with 
the state of modern mobile browsers. When we started off this project we had 
much lower expectations on how seamless, integrated and performant we would be 
able to make it. It's been a great learning experience for us and all the time 
spent iterating and testing (a lot) has improved our understanding of how modern
browsers work. And that's what it will take if we want to shorten the production
time on these types of projects, going from guessing to knowing.

 [1]: http://www.html5rocks.com/en/tutorials/casestudies/hobbit/
 [2]: http://middle-earth.thehobbit.com/
 [3]: img/eagles.png

 [4]: https://developers.google.com/chrome-developer-tools/docs/mobile-emulation
 [5]: http://diveintohtml5.info/history.html
 [6]: http://visionmedia.github.io/page.js/
 [7]: img/thranduils-hall.jpg
 [8]: http://middle-earth.thehobbit.com/thranduils-hall
 [9]: http://www.greensock.com/draggable/
 [10]: http://www.greensock.com/throwprops/
 [11]: https://gist.github.com/inear/7626665
 [12]: http://awardwinningfjords.com/2012/03/08/image-sequences.html
 [13]: http://www.imagemagick.org/script/index.php
 [14]: https://gist.github.com/inear/7616849

 [15]: https://developers.google.com/chrome-developer-tools/docs/tips-and-tricks#timeline-frames-mode
 [16]: http://jankfree.org/
 [17]: http://www.html5rocks.com/en/tutorials/memory/effectivemanagement/
 [18]: img/failing-dispose.png
 [19]: img/much-better.png

 [20]: https://developers.google.com/chrome-developer-tools/docs/heap-profiling?hl=sv&csw=1#views_paths
 [21]: img/effectcompositor.png

 [22]: http://www.html5rocks.com/en/tutorials/speed/high-performance-animations/#toc-animating-layout-properties
 [23]: img/instructions.jpg