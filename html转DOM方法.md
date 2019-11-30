## 方法
1.  创建一个元素 `div=document.createElement('div');div.innerHTML=html`
2. DOMParser `domParser=new DOMParser();domParser.parseFromString.(html,'text/html')`
3. 新建个Document对象 `document.implementation.createHTMLelement`

**注意**：方法①相当于在当前`window.document`上`createElement`，创建的div是要被浏览器遍历的，而方法②③是新创建一个document与浏览器里的不是一个，和浏览器没关系，浏览器也管不到了
## 比较
##### 1. 方法①
- 优点：处理快速，代码简单 
- 缺点：浏览器会先遍历执行一遍。因为是在浏览器的dom对象里生成的，因此浏览器会遍历一遍，里面的图片都会预加载（即使当前页无用）或者script标签会执行。（当然如果所加载的图片都是当前页需要的，比如项目里的全部数据加载。那由于图片预加载一遍之后路径相同，就不会再加载了）

**这里插入一个关于图片预加载问题**

【1】问题：设置`display：none`的 img还会加载吗？会的
-  img图片预加载触发点：在img的src赋值时浏览器就会开始预加载了
![image](https://github.com/jiameiw/share/blob/master/pics/20191129/1.png)

- 图片在浏览器中的加载时机：在解析html构建dom树时，遇到img就开始加载

而`display：none`只会影响渲染树的隐藏和显示，因此设置了`display：none`也依然会加载。
**总结**：`<img>`标签只要存在在dom里就会加载，阻止他被浏览器遍历从而加载的方式就是不让他出现在`window.document`里。

【2】问题：样式表中的背景图片设置了`display：none`还会加载吗？会的
- 样式表中的背景图片加载时机：构建渲染树时
遍历到了，就会加载了

【3】问题：样式表中的背景图片的父元素设置了`display：none`还会加载吗？不会

构建渲染树时父元素设为none，则默认子元素也不渲染，也就遍历不到了，因此不会渲染。

##### 2. 使用`domparser api/document.implementation.createHTMLelement`
   好处：
   - 防止html里的img图片自动加载
   - 防止`<script>`标签的执行，避免xss攻击
 
缺点：domparser api的性能最差 可参考：[性能对比](https://jsperf.com/domparser-vs-createelement-innerhtml/3) 

所以有快速生成的需求时，尽量不要用DOMParser
##### 3. 试验（项目里打`console.time`和`console.timeEnd`）
```
//1.DOMParser----->用时大概75ms
const parser = new DOMParser()
const dom = parser.parseFromString(html, 'text/html')
return dom && dom.documentElement ? dom.documentElement.innerHTML.replace(/<br>|<hr>/g, '') : ''
//2.createHTMLDocument----->用时大概60ms
const dom = document.implementation.createHTMLDocument(html)
dom.documentElement.innerHTML = html
return dom && dom.documentElement ? dom.documentElement.innerHTML.replace(/<br>|<hr>/g, '') : ''
//3.new div----->用时大概36ms
const htmlWrapDiv = document.createElement('div')
htmlWrapDiv.innerHTML = html
return htmlWrapDiv.innerHTML.replace(/<br>|<hr>/g, '')
```


##应用
去除html里多余的无用的标签或者注释等，省去用正则表达式，因为可能存在`<br/><hr/>`等不好筛选
```
const parser = new DOMParser()
const dom = parser.parseFromString(html, 'text/html')
return dom && dom.documentElement ? dom.documentElement.innerHTML.replace(/<br>|<hr>/g, '') : ''
```


##### 参考

[DOMParser WEB API](https://developer.mozilla.org/zh-CN/docs/Web/API/DOMParser)

 [web图片资源加载与渲染时机](https://segmentfault.com/a/1190000010032501)
