### 1. 问题描述（举例）
https://codepen.io/cammiew/pen/ExgwBNY
嵌套flex内的元素溢出父元素，被撑破。

### 1. 原因
2019年1月29日，Chrome72对flex做了一点改变，详细见[规范](https://www.w3.org/TR/css-flexbox-1/#min-size-auto)。也就是说flex主轴上子元素的最小大小是内容的大小。![image.png](./pics/flex.png)
chrome72之前的版本，都是没有按照规范行为而出现的bug，这次调整只是让它符合规范行为。也就是说子元素是flex弹性盒子，应该要和普通盒子的处理方式一样，内容会把外层父元素撑开，因此`min-width`和`min-height`属性的默认值为`auto`。
### 1. 解决方法
在内容的直系父元素上指定一个最小宽度（或高度），去覆盖它的默认行为（即内容的高度），比如`min-width:0`。
    - 这样如果内容有滚动条，就不会出现内容撑破的情况了。
    - 如果内容不需要滚动条，需要设置断行。（QQ浏览器表现不一致，还要限制下内容`min-width`或者加`overflow:hidden`。


