# 留给IPhoneX 的网页设计

> 以下文章关于安全区域的配图于2017年10月31号更新，配合IOS 11.2 beta版更新，虽然是老文章，但是对于我来说，没遇到过的问题都值得做个小总结，测试小哥给我的测试反馈中遇到安全区域吸底的问题，结合网上查到这篇webKit文章能有更清晰的了解。



盒模型以外，Safari 对于新版IPhone X 留下了一个美观的页面展示，页面内容自动的嵌入在安全区域内，而不是模糊的在边缘菱角上展示或者被传感器遮挡住。

插入的区域会被页面中的`background-color` 填充塞满(如 `<body> `或 `<html>` 指定的)并混入页面的剩余部分，对于不少网页来说，已经足够了，如果你的网页纯背景下只是存在部分文字或者图片，那插图确实会很好看。

其他的页面，特别是那些为全面屏贴切导航栏设计的页面，像下面图所示，可以选择性进一步使用更全面新展示的特性。 [IPhone X 人机交互指南](https://developer.apple.com/design/human-interface-guidelines/ios/visual-design/adaptivity-and-layout/) 详细介绍了一些应该记住的常规设计准则， 同时也讨论了拥有特殊机制的原生应用可以采纳[UIKit 文档](https://developer.apple.com/documentation/uikit/uiview/positioning_content_relative_to_the_safe_area) 让应用界面变得更加好看，我们的网站也可以使用一些新的IOS 11 WebKit Api 以获得无缝边缘最佳的展示体验。

![viewport-fit="cover"](https://raw.githubusercontent.com/Trojan0523/weekly_notes/main/image/viewport-fit-cover-the-navbar.png)

## 使用全面屏

第一个新特性就是给已有meta属性`viewport`增加一个拓展属性 -- `viewport-fit` , 此属性能提供一个控制插图的行为，此属性在IOS 11 已经可以使用了。

`viewport-fit`的默认值是 `auto` , 此属性行为为以上图所示，为了禁用此行为带来的布局超出屏幕的问题，可以将`viewport-fit`值设置为`cover` ， `viewport`元标签会变成：

```html
<meta name='viewport' content='initial-scale=1, viewport-fit=cover' >
```

![viewport-fit=cover 填充完整整个视窗](https://raw.githubusercontent.com/Trojan0523/weekly_notes/main/image/viewport-fit-cover-the-navbar.png)

重新加载后，导航栏就会往外贴切了不少，看起来是一个贴边的样子而不是空出来一小部分，不过，我们需要弄清楚为什么遵从系统的安全区域插图十分重要：一些页面内容被设备的传感器外壳遮住了(手机上面会有一些传感器，屏幕一部分的视窗是无法被触摸使用的)，而且底部导航栏难以利用.



## 尊重安全区域

采用`viewport-fit=cover` 选择性将一些包含重要内容填充到节点时，需要进一步的操作让我们的页面保持可用，同时确保他们不会在屏幕边缘变得模糊(obscured)。为了能更充分利用页面上Iphone X以上机型增加出来的屏幕高度，使用过程中能动态调整避免误触边缘角落，传感器外壳或者主屏幕的返回Home键。

![safe-area](https://raw.githubusercontent.com/Trojan0523/weekly_notes/main/image/safe-areas-1.png)

为了达到以上图片的效果，webKit为IOS 11 提供了新式CSS函数 ： env() ，此函数同时拥有 [四个单独环境变量定义](https://github.com/w3c/csswg-drafts/pull/1819) ，有兴趣的同学可以去看看上面链接w3c上面的css草案提议。四个变量分别是： `safe-area-inset-left` `safe-area-inset-right` `safe-area-inset-bottom` `safe-area-inset-top`  ，分别代表左右下上不同方向的安全区域插图属性，当混合样式的时候，这些属性允许样式声明引用于每个方向当前安全区域的尺寸。



> ios11 附带的 env() 方法 函数名为 constant() ，从Safari Technology Preview 41 和 ios11.2 beta版开始，constant() 已被删除并替换为env() ，无论有无必要 请将两个方法一并写上(个人观点) ,不然会导致不同系统不同机型识别不了此css方法。

```css
// 踩坑点 注意：需要同时两个函数一起出现，且顺序不能换！！！
height:  constant(safe-area-inset-bottom - $height) /* 兼容IOS < 11.2 */
height: env(safe-area-inset-bottom - $height) /* 兼容IOS >= 11.2 */
```

env() 适用于任何地方的 var() 常量， 例如 在`padding`属性内部:

```css
.stick-to-bottom-element {
  padding: 12px;
  padding-left: env(safe-area-inset-left);
  padding-right: env(safe-area-inset-right);
}
```

对于不支持env() 的浏览器，这条样式规则会被忽略掉，所以分别使用特殊的指定向下兼容的规则很重要，还是需要尊重安全区域内容插图，以便重要内容可见不会被覆盖，同时安全区域不放置任何文字类或点击内容。

![respect safe area](https://raw.githubusercontent.com/Trojan0523/weekly_notes/main/image/safe-area-constants.png)



## 使用min() 和 max() 做组合

> 以下为 Safari Technology Preview 41 和 IOS 11.2 beta 才开始提供的功能

如果您还是想开发时在安全区插入内容，可能会注意到 指定一个最小的填充padding作为安全区域插图比较困难，以上页面我们是使用了12px 左侧padding替换掉 `env(safe-area-inset-left)` 当手机旋转回纵向的时候，左侧安全区域插入会变为0px, 文本紧贴屏幕边缘。

![屏幕旋转回来后没有替换的margin](https://raw.githubusercontent.com/Trojan0523/weekly_notes/main/image/no-margins.png)



为了解决这个问题，我们要指定填充的padding是一个默认的padding 或者安全区域插图，以偏大的元素决定优先级。这里可以通过实现全新的css功能 min() 和 max() 实现，这会在未来的safari浏览器中提供，（笔者写下这篇文章的时候是2021年9月3日，早在2018年底支持了，可以放心使用 [张鑫旭 min() 、max() 数学函数讲解](https://www.zhangxinxu.com/wordpress/2020/04/css-min-max-clamp/)）这两个函数都可以设置任意个数的参数然后返回一个最大值或者最小值，两个函数也可以在calc() 中使用，又或者互相嵌套，两个函数都支持calc() 在内部计算：



对于上面图的情况，最好使用max() 

```css
// 除IE外浏览器都支持，用于指定依赖浏览器一个或者多个特定css功能的支持声明，也可以叫做 特性查询
@supports (padding: max(0px)) {
  .stick-to-left-elements {
    padding-left: max(12px, env(safe-area-inset-left));
    padding-right: max(12px, env(safe-area-inset-right));
  }
}
```



> 使用@ supports检测最大最小值十分重要，因为并不是所有地方都支持，并且由于CSS对无效变量有处理，所以不要在@supports查询中指定变量



在实例页面中，纵向`env(safe-area-inset-left)` 解析为0px，因此，max() 函数解析为 `0 + 12px = 12px` ， 横向`env(safe-area-inset-left)`由于传感器外壳变大，该max() 功能会改为解析env() 函数的尺寸并使用此尺寸作为填充，保证重要内容永远都是可见的。

![max() 函数解析外边距填充](https://raw.githubusercontent.com/Trojan0523/weekly_notes/main/image/max()%20%E5%87%BD%E6%95%B0%E8%A7%A3%E6%9E%90%E5%A1%AB%E5%85%85.png)



有经验的前端开发可能会遇到一个 "CSS Block"机制，通常将CSS限制在特定范围内的值，所以使用min() 和max() 可以更有效解决此问题，并且用于实现有效的响应式设计十分有帮助。



## 回归实际的应用场景：

M站中同样有存在两处需要吸底触碰IPhone X以上机型 底部安全区域的情景： 一个是add-to-bag加入购物车按钮吸底panel， 一个是优惠券吸底

![有安全区域加车](https://raw.githubusercontent.com/Trojan0523/weekly_notes/main/image/add-to-bag-safe.png)

![导航栏加车](https://raw.githubusercontent.com/Trojan0523/weekly_notes/main/image/add-to-bag-navbar.png)

这里的加车是使用了一个白色的背景区吸底加高了底部的padding边距，加高了一个底部镂空区域，与前文的尊重安全区域不做任何有效内容展示相符，具体的css样式如下：

```css
.add-to-bag-sticky {
  width: 100%;
  position: fixed;
  left: 0;
  bottom: 0;
  z-index: 10;
  text-align: center;
  background-color: $white;
  padding: 5px 0;
  @include clear-float();
}
.iphone-white {
  @include home-indicator-compatible($white);
}
// 公用mixin吸底处理安全区域函数
@mixin home-indicator-compatible ($color: #E7E8E8) {
  margin-bottom: 0;
  margin-bottom: constant(safe-area-inset-bottom); // 下方加上一个安全区域高度， 34px ios < 11.2
  margin-bottom: env(safe-area-inset-bottom); // 同上 34px ios >= 12.2 beta

  &:after {
    content: '';
    width: 100vw;
    height: 0;
    height: constant(safe-area-inset-bottom);
    height: env(safe-area-inset-bottom);
    position: fixed;
    left: 0;
    bottom: 0;
    background-color: $color;
  }
}
```



优惠券吸底情况： 

![有安全区域优惠券](https://raw.githubusercontent.com/Trojan0523/weekly_notes/main/image/coupon-safe.png)

![有导航栏优惠券](https://raw.githubusercontent.com/Trojan0523/weekly_notes/main/image/coupon-navbar.png)

这里的做法有悖于上文尊重安全区域 , 根据需求需要完全吸底，所以没有留出固定高度避免将重要内容展示在安全区域，具体的css样式如下：

```css
// 优惠券整区域吸底
.coupon-wrapper {
    position: fixed;
    bottom: 0;
    left: 0;
    width: 100vw;
    height: 80px;
    background-color: $white;
    z-index: 10;
    border-top: 2PX solid #000000;
    opacity: 0.95;
    // -webkit-overflow-scrolling: touch;
}
// 可视文字内容安全区加高
.coupon-content {
    padding-top: 4px;
    display: flex;
    justify-content: center;
    align-items: center;
    @supports (bottom: constant(safe-area-inset-bottom)) or (bottom: env(safe-area-inset-bottom)) {
        div {
            margin-bottom: constant(safe-area-inset-bottom);
            margin-bottom: env(safe-area-inset-bottom);
        }
    }
}
```







## summary

- 虽然应该尊重Webkit提出的避免在安全区域展示的准则，但是无可避免的会遇到一些需求需要不同的设定，这里只是一个简单的建议，尽量将安全区域块空出不做内容的展示。
- 完

