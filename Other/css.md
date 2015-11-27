#高性能css-----性能大作战

<!-- toc -->
[toc]
##一，游览器的工作原理、工作线程


###1.游览器工作原理

>1、首先，浏览器引擎将HTML解析为一颗DOM树（也叫文档对象模型），
>2、然后根据DOM树的CSS几何属性（大小和边距等）构建一棵渲染树。（渲染树不包含隐藏的元素）。
>3、最后根据渲染树节点（称为“帧“或者”盒“）的CSS属性，游览器把html元素 “绘制” 到屏幕上，按照从左上开始到右下的顺序。

![Alt text](./8_1.jpg)

<p>
 <p style="color:red">注：DOM树中每一个节点在渲染树中至少存在一个对应的节点（隐藏的DOM元素在渲染树中没有对应的节点）。
 <p style="color:red">渲染树中的节点被称为“帧“或者”盒“，可以理解为一个具有填充(padding)，边距(marging)，边框(borders)和位置(position)的盒子。
<p style="color:red">一旦DOM和渲染树构建完成，浏览器就开始显示(绘制"paint")页面元素，对渲染树的计算通常只需要遍历一次。但table及其内部元素通常要花3倍于同等元素的时间。




###2.浏览器工作线程
游览器工作线程图示：

![Alt text](./游览器工作线程.png)


游览器排版线程工作原理：
>* 当页面变化时，排版线程会尝试以每秒60帧的速度去重绘页面，即便这时页面还不完整。
>* 当用户滚动页面时，排版线程向主线程请求更新页面 新显示部分 的位图。但是，如果主线程不能迅速响应请求，排版线程会用它目前所拥有的这部分内容去渲染页面，由于对应的内容还没有，所以会以白板的形式渲染出来。
>* 当DOM的变化影响了元素的几何属性(宽和高)，浏览器需要重新计算元素的几何属性，同样其他元素的几何属性和位置也会因此受到影响
 



##二，重绘与回流（repaint | reflow）

###1.什么叫回流与重绘

> 1.回流：当元素的规模尺寸，布局，隐藏等改变而重新构建渲染树，称为回流。
> 
> 2.重绘：当需要更新某些元素属性，这些属性只是影响元素的外观、风格，而不影响布局。就称为重绘。
> 
><p style="color:red"> 注：回流必将引起重绘，而重绘不一定会引起回流。


###2.触发重绘、回流的操作：

>1、调整窗口大小、改变字体、增加或者移除样式表
>
>2、添加、删除元素 、隐藏元素 (display:none )、改变元素尺寸和位置（比如改变 top 、left 、 jquery的animate方法）。内容变化，比如用户在input框中输入文字
>
>3、操作 class 属性、脚本操作 DOM；激活 CSS 伪类，比如 :hover 
>
>4、获取某些属性会强制游览器回流重绘。
* offsetTop, offsetLeft, offsetWidth, offsetHeight    
* scrollTop/Left/Width/Height   
* clientTop/Left/Width/Height   
* 请求了getComputedStyle(), 或者 ie的 currentStyle 计算

<p style="color:red">注：visibility:hidden(只重绘，不回流)

###3.如何搞定回流
1. 很多浏览器都会优化操作，浏览器会维护1个队列，把所有会引起回流、重绘的操作放入这个队列，等队列中的操作到了一定的数量或者到了一定的时间间隔，浏览器就会把flush队列，进行一个批处理。这样就会让多次的回流、重绘变成一次回流重绘。虽然有了浏览器的优化，但有时候我们写的一些代码可能会强制浏览器提前flush队列，这样浏览器的优化可能就起不到作用了。当你请求向浏览器请求一些style信息的时候，就会让浏览器flush队列，比如：
    1. offsetTop, offsetLeft, offsetWidth, offsetHeight
    2. scrollTop/Left/Width/Height
    3. clientTop/Left/Width/Height
    4. width,height
    5. 请求了getComputedStyle(), 或者 ie的 currentStyle
    
    当你请求上面的一些属性的时候，浏览器为了给你最精确的值，需要flush队列，因为队列中可能会有影响到这些值的操作。所以，在多次使用这些值时应进行缓存。 



如果想设定元素的样式，通过改变元素的 class 名 (尽可能在 DOM 树的最里层)（Change classes on the element you wish to style (as low in the dom tree as possible)）
避免设置多项内联样式（Avoid setting multiple inline styles）
应用元素的动画，使用 position 属性的 fixed 值或 absolute 值（Apply animations to elements that are position fixed or absolute）
权衡平滑和速度（Trade smoothness for speed）
避免使用 table 布局（Avoid tables for layout）
避免使用CSS的 JavaScript 表达式 (仅 IE 浏览器)（Avoid JavaScript expressions in the CSS (IE only)）


避免设置多层内联样式
我们都知道与DOM交互很慢。我们尝试在一种无形的DOM树片段组进行更改，然后整个改变应用到DOM上时仅导致了一个回流。同样，通过style属性设置样式导致回流。避免设置多级内联样式，因为每个都会造成回流，样式应该合并在一个外部类，这样当该元素的class属性可被操控时仅会产生一个reflow。
动画效果应用到position属性为absolute或fixed的元素上


如何减少回流、重绘

1. 不要1个1个改变元素的样式属性，最好直接改变className：
// bad
var left = 1;
var top = 1;
el.style.left = left + "px";
el.style.top  = top  + "px";

// good 
el.className += " className1";

// 比较好的写法 
el.style.cssText += "; left: " + left + "px; top: " + top + "px;";

2. 让要操作的元素进行"离线处理"，处理完后一起更新，这里所谓的"离线处理"即让元素不存在于render tree中，比如：
        a) 使用documentFragment或div等元素进行缓存操作，这个主要用于添加元素的时候，大家应该都用过，就是先把所有要添加到元素添加到1个div(这个div也是新加的)，
            最后才把这个div append到body中。
        b) 先display:none 隐藏元素，然后对该元素进行所有的操作，最后再显示该元素。因对display:none的元素进行操作不会引起回流、重绘。所以只要操作只会有2次回流。

   3 不要经常访问会引起浏览器flush队列的属性，如果你确实要访问，就先读取到变量中进行缓存，以后用的时候直接读取变量就可以了，见下面代码：
// 别这样写，大哥
for(循环) {
    el.style.left = el.offsetLeft + 5 + "px";
    el.style.top  = el.offsetTop  + 5 + "px";
}

// 这样写好点
var left = el.offsetLeft,top  = el.offsetTop,s = el.style;
for(循环) {
    left += 10;
    top  += 10;
    s.left = left + "px";
    s.top  = top  + "px";
}


 4. 考虑你的操作会影响到render tree中的多少节点以及影响的方式，影响越多，花费肯定就越多。比如现在很多人使用jquery的animate方法移动元素来展示一些动画效果，想想下面2种移动的方法：
// block1是position:absolute 定位的元素，它移动会影响到它父元素下的所有子元素。
// 因为在它移动过程中，所有子元素需要判断block1的z-index是否在自己的上面，
// 如果是在自己的上面,则需要重绘,这里不会引起回流
$("#block1").animate({left:50});
// block2是相对定位的元素,这个影响的元素与block1一样，但是因为block2非绝对定位
// 而且改变的是marginLeft属性，所以这里每次改变不但会影响重绘，
// 还会引起父元素及其下元素的回流
$("#block2").animate({marginLeft:50});

注：开发中，比较好的实践是尽量减少重排次数和缩小重排的影响范围。例如： 
1. 将多次改变样式属性的操作合并成一次操作。例如： 
合并为： 
.changeDiv { background: #eee;  color: #093;  height: 200px; } 
document.getElementById('changeDiv').className = 'changeDiv'; 

2. 将需要多次重排的元素，position属性设为absolute或fixed，这样此元素就脱离了文档流，它的变化不会影响到其他元素。例如有动画效果的元素就最好设置为绝对定位。
  
3. 在内存中多次操作节点，完成后再添加到文档中去。例如要异步获取表格数据，渲染到页面。可以先取得数据后在内存中构建整个表格的html片段，再一次性添加到文档中去，而不是循环添加每一行。 
  
4. 由于display属性为none的元素不在渲染树中，对隐藏的元素操作不会引发其他元素的重排。如果要对一个元素进行复杂的操作时，可以先隐藏它，操作完成后再显示。这样只在隐藏和显示时触发2次重排。 
  
5. 在需要经常取那些引起浏览器重排的属性值时，要缓存到变量。 
  





##三，选择器的性能

###1.CSS匹配从右到左查找原则：
比如DIV#divBox p span.red{color:red;}：浏览器的查找顺序如下：先查找html中所有class='red'的span元素，找到后，再查找其父辈元素中是否有p元素，再判断p的父元素中是否有id为divBox的div元素，如果都存在则匹配上。

###2.不要在ID选择器前使用标签名
一般写法：DIV#divBox
更好写法：#divBox
解释： 因为ID选择器是唯一的，加上div反而增加不必要的匹配。

###3.使用class减少层级关系

##四，css性能优化原则
###1.优化原则
1. 减少选择器的数量 （包括这样的跟浏览器有关的: .ie7 .foo .bar）
2. 减少 reflow
不要用通配符（包括想这样不规范的类型选择符[type="url"]）
页面的缩放比例是会影响 CSS 性能的（比如： Opera）
窗口大小也会影响 CSS 性能 （比如： Chrome）
页面反复重新加载会降低 CSS 性能 （比如 Opera）
“border-radius” 和 “transform” 是性能最差的 CSS 属性（至少在WebKit & Opera 中是）
基于 WebKit 的浏览器中，Timeline 栏可以算出总的样式 计算/reflow/repaint 时间
WebKit 中的选择器匹配快得多很多

>1、[type=”…”] 比 input[type=”…”] 更加耗时
2、::selection 和 :active 是最为耗时的选择器
3、text-shadow 和 linear-gradient 是最不耗时的.
     Opacity 和透明 rgba() 样色相对好一些。
     box-shadow 带这inset one (0 1px 1px 0) 稍微比(0 2px 3px 0) 好一点. 
     border-radius 更高一些。
减少HTML 中元素的数量。
减少重绘。





 
##CSS 的动画和转换性能



原理

现代浏览器在使用CSS3动画时，以下四种情形绘制的效率较高，分别是： 
* 改变位置 
* 改变大小 
* 旋转 
* 改变透明度


CSS的图层的概念（Chrome浏览器）

浏览器在渲染一个页面时，会将页面分为很多个图层，图层有大有小，每个图层上有一个或多个节点。在渲染DOM的时候，浏览器所做的工作实际上是： 
1. 获取DOM后分割为多个图层 
2. 对每个图层的节点计算样式结果（Recalculate style--样式重计算） 
3. 为每个节点生成图形和位置（Layout--回流和重布局） 
4. 将每个节点绘制填充到图层位图中（Paint Setup和Paint--重绘） 
5. 图层作为纹理上传至GPU 
6. 符合多个图层到页面上生成最终屏幕图像（Composite Layers--图层重组）
Chrome中满足以下任意情况就会创建图层： 
* 3D或透视变换（perspective transform）CSS属性 
* 使用加速视频解码的 <video> 节点 
* 拥有3D（WebGL）上下文或加速的2D上下文的 <canvas> 节点 
* 混合插件（如Flash） 
* 对自己的opacity做CSS动画或使用一个动画webkit变换的元素 
* 拥有加速CSS过滤器的元素 
* 元素有一个包含复合层的后代节点（一个元素拥有一个子元素，该子元素在自己的层里） 
* 元素有一个 z-index 较低且包含一个复合层的兄弟元素（换句话说就是该元素在复合层上面渲染）
需要注意的是，如果图层中某个元素需要重绘，那么整个图层都需要重绘。比如一个图层包含很多节点，其中有个gif图，gif图的每一帧，都会重回整个图层的其他节点，然后生成最终的图层位图。所以这需要通过特殊的方式来强制gif图属于自己一个图层（ translateZ(0) 或者 translate3d(0,0,0) ），CSS3的动画也是一样（好在绝大部分情况浏览器自己会为CSS3动画的节点创建图层）
层和CSS动画

简化一下上述过程，每一帧动画浏览器可能需要做如下工作： 
1. 计算需要被加载到节点上的样式结果（Recalculate style--样式重计算） 
2. 为每个节点生成图形和位置（Layout--回流和重布局） 
3. 将每个节点填充到图层中（Paint Setup和Paint--重绘） 
4. 组合图层到页面上（Composite Layers--图层重组）
如果我们需要使得动画的性能提高，需要做的就是减少浏览器在动画运行时所需要做的工作。最好的情况是，改变的属性仅仅印象图层的组合，变换（ transform ）和透明度（ opacity ）就属于这种情况
现代浏览器如Chrome，Firefox，Safari和Opera都对变换和透明度采用硬件加速，但IE10+不是很确定是否硬件加速
触发重布局的属性

有些节点，当你改变他时，会需要重新布局（这也意味着需要重新计算其他被影响的节点的位置和大小）。这种情况下，被影响的DOM树越大（可见节点），重绘所需要的时间就会越长，而渲染一帧动画的时间也相应变长。所以需要尽力避免这些属性
一些常用的改变时会触发重布局的属性： 
盒子模型相关属性会触发重布局： 
* width 
* height 
* padding 
* margin 
* display 
* border-width 
* border 
* min-height
定位属性及浮动也会触发重布局： 
* top 
* bottom 
* left 
* right 
* position 
* float 
* clear
改变节点内部文字结构也会触发重布局： 
* text-align 
* overflow-y 
* font-weight 
* overflow 
* font-family 
* line-height 
* vertival-align 
* white-space 
* font-size
这么多常用属性都会触发重布局，可以看到，他们的特点就是可能修改整个节点的大小或位置，所以会触发重布局
别使用CSS类名做状态标记

如果在网页中使用CSS的类来对节点做状态标记，当这些节点的状态标记类修改时，将会触发节点的重绘和重布局。所以在节点上使用CSS类来做状态比较是代价很昂贵的
触发重绘的属性

修改时只触发重绘的属性有： 
* color 
* border-style 
* border-radius 
* visibility 
* text-decoration 
* background 
* background-image 
* background-position 
* background-repeat 
* background-size 
* outline-color 
* outline 
* outline-style 
* outline-width 
* box-shadow
这样可以看到，这些属性都不会修改节点的大小和位置，自然不会触发重布局，但是节点内部的渲染效果进行了改变，所以只需要重绘就可以了
手机就算重绘也很慢

在重绘时，这些节点会被加载到GPU中进行重绘，这对移动设备如手机的影响还是很大的。因为CPU不如台式机或笔记本电脑，所以绘画巫妖的时间更长。而且CPU与GPU之间的有较大的带宽限制，所以纹理的上传需要一定时间
触发图层重组的属性

透明度竟然不会触发重绘？

需要注意的是，上面那些触发重绘的属性里面没有 opacity （透明度），很奇怪不是吗？实际上透明度的改变后，GPU在绘画时只是简单的降低之前已经画好的纹理的alpha值来达到效果，并不需要整体的重绘。不过这个前提是这个被修改 opacity本身必须是一个图层，如果图层下还有其他节点，GPU也会将他们透明化
强迫浏览器创建图层

在Blink和WebKit的浏览器中，一当一个节点被设定了透明度的相关过渡效果或动画时，浏览器会将其作为一个单独的图层，但很多开发者使用 translateZ(0) 或者 translate3d(0,0,0) 去使浏览器创建图层。这种方式可以消除在动画开始之前的图层创建时间，使得动画尽快开始（创建图层和绘制图层还是比较慢的），而且不会随着抗锯齿而导出突变。不过这种方法需要节制，否则会因为创建过多的图层导致崩溃
Chrome中的抗锯齿

Chrome中，非根图层以及透明图层使用grayscale antialiasing而不是subpixel antialiasing，如果抗锯齿方法变化，这个效果将会非常显著。如果你打算预处理一个节点而不打算等到动画开始，可以通过这种强迫浏览器创建图层的方式进行
transform变换是你的选择

我们通过节点的 transform 可以修改节点的位置、旋转、大小等。我们平常会使用 left 和 top 属性来修改节点的位置，但正如上面所述， left 和 top 会触发重布局，修改时的代价相当大。取而代之的更好方法是使用 translate ，这个不会触发重布局
JS动画和CSS3动画的比较

我们经常面临一个抉择：是使用JavaScript的动画还是使用CSS的动画，下面将对比一下这两种方式
JS动画

缺点：JavaScript在浏览器的主线程中运行，而其中还有很多其他需要运行的JavaScript、样式计算、布局、绘制等对其干扰。这也就导致了线程可能出现阻塞，从而造成丢帧的情况。
优点：JavaScript的动画与CSS预先定义好的动画不同，可以在其动画过程中对其进行控制：开始、暂停、回放、中止、取消都是可以做到的。而且一些动画效果，比如视差滚动效果，只有JavaScript能够完成
CSS动画

缺点：缺乏强大的控制能力。而且很难以有意义的方式结合到一起，使得动画变得复杂且易于出问题。 
优点：浏览器可以对动画进行优化。它必要时可以创建图层，然后在主线程之外运行。
前瞻

Google目前正在探究通过JS的多线程（Web Workers）来提供更好的动画效果，而不会触发重布局及样式重计算
结论

动画给予了页面丰富的视觉体验。我们应该尽力避免使用会触发重布局和重绘的属性，以免失帧。最好提前申明动画，这样能让浏览器提前对动画进行优化。由于GPU的参与，现在用来做动画的最好属性是如下几个： 
* opacity 
* translate 
* rotate 
* scale
也许会有一些新的方式使得可以使用JavaScript做出更好的没有限制的动画，而且不用担心主线程的阻塞问题。但在那之前，还是好好考虑下如何做出流畅的动画吧





参考资料：

深入浏览器研究 CSS 的动画和转换性能
http://www.oschina.net/translate/css-animations-and-transitions-performance?print

减少重绘
http://book.2cto.com/201406/43611.html

页面重绘与重排版的性能影响
http://blog.csdn.net/hhq163/article/details/6965895
status 

前端性能优化（CSS动画篇）
http://www.tuicool.com/articles/NBbQjy3

高性能WEB开发(8) - 页面呈现、重绘、回流。
http://www.blogjava.net/BearRui/archive/2010/05/10/web_performance_repaint_relow.html

浏览器的工作原理：新式网络浏览器幕后揭秘
http://www.html5rocks.com/zh/tutorials/internals/howbrowserswork/

复杂应用的 CSS 性能分析和优化建议
http://www.orzpoint.com/profiling-css-and-optimization-notes/

<!-- toc -->


