---
title: backdrop-filter
date: 2017-07-10 00:22:48
tags:
      -css
---
# css新属性backdrop-filter

------

在iOS系统上常常能看到高斯模糊(Gaussian Blur)效果，而这种效果早期使用CSS来实现是较为痛苦的一件事情。其实，早上2011年，浏览器就开始对CSS filters规范有所支持。也就是说在2011年浏览器就可以实现Filters效果。但这种效果基本上都只运用在SVG上(只有SVG支持Filter效果)，而且只有Firefox浏览器支持，并且只能运用在HTML上。这也造就CSS要实现Filters效果是非常蛋疼的一件事情。比如说下图的效果：

![fosted-glass][1]


### 浏览器兼容性

![此处输入图片的描述][2]


  [1]: http://wx4.sinaimg.cn/mw1024/a359ab18gy1fhdwokjuh8j20c405xwfk.jpg
  [2]: http://wx4.sinaimg.cn/mw690/a359ab18gy1fhdwrsqo5lj20pp0863z2.jpg
  
  到目前为止，也仅有Safari 9支持
  
## backdrop-filter
backdrop-filter是在Filter Level2提出来的。其取值和filter Level1中filter属性的属性值一样，包括：
1. blur()
2. brightness()
3. contrast()
4. drop-shadow()
5. grayscale()
6. hue-rotate()
7. invert()
8. opacity()
9. saturate()
10. sepia()

这个属性现在仅仅适用于ios端，用来实现ios端毛玻璃效果
<font color='blue'>参考资料(http://www.w3cplus.com/css3/advanced-css-filters.html)</font>

--- 

## 纯css实现毛玻璃另外一种方法：

先搞一个div作为容器层，用来放置风景背景图。内部放一个div，作为毛玻璃的主体。再放一个img，显示图标。
容器层：大小是图片大小，把风景图作为背景显示，no-repeat。这里用到一个小技巧，将background-attachment设成fixed，不随元素滚动，让子元素继承了本层的背景后，子元素就变成了一个viewport，移到哪儿就看到背景的哪儿

    .container{
        width: 287px;height: 285px;
        background-image: url(background.png);    
        background-repeat: no-repeat;
        background-attachment: fixed;
        overflow: hidden;
    }

毛玻璃层：这里的关键技巧就是background:inherit，直接使用了父元素的背景，和父级的background-attachment:fixed可完成从相机看世界的各种牛逼效果。本文的的毛玻璃是全景，当然可以上半部或者下半部，或者其他位置，这就看出inherit和fixed牛逼的地方了。        

    .frosted-glass{
        width: 287px;
        height: 285px;
        background: inherit;
        -webkit-filter: blur(5px);
        -moz-filter: blur(5px);
        -ms-filter: blur(5px);
        -o-filter: blur(5px);
        filter: blur(5px);
        filter: progid:DXImageTransform.Microsoft.Blur(PixelRadius=4, MakeShadow=false);
    }
    
测试了只能对图片进行模糊，对dom节点这些页面展示内容无法进行模糊。

要实现对页面内容模糊，方法是：
1、需要能将图片模糊的滤镜。
2、需要捕获到毛玻璃浮层下面所有内容的图像信息。就好像截屏。
3、需要让图片运动能与浏览器滚动同步。主要难点在于实时同步，尤其是处理滚动的惯性。
<font color='blue'>参考资料(https://www.zhihu.com/question/23533918)
如何通过 HTML5 实现 iOS 7 的实时毛玻璃模糊效果？  ---by 知乎</font>


