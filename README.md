# Front_end_Interview
前端面试汇总

## <a name='table-of-contents'>目录</a>

 #### [HTML 相关](#html-interviews)
 #### [HTML5 相关](#html5-interviews)
 #### [CSS 相关](#css-interviews)
 #### [CSS3 相关](#css3-interviews)
 #### [JS 相关](#js-interviews)
 #### [React 相关](#react-interviews)
 
 <details>
    <summary>View contents</summary>
    
    * [`call`](#call)
    * ['react、vue原理，两者的区别还有基本的思想'](#react、vue原理，两者的区别还有基本的思想)
    * [react怎么优化](#react_1)
    * [react的思想是什么](#react_3)
    * [`yesNo`](#yesno)
    
 </details>
 
 
 #### [移动端 相关](#app-interviews)
 #### [Redux 相关](#redux-interviews)
 #### [Vue 相关](#vue-interviews)
 #### [Node 相关](#node-interviews)

---
#### <a name='html-interviews'>HTML 相关：</a>
  
  
<br>[⬆ Back to top](#table-of-contents)

---
#### <a name='html5-interviews'>HTML5 相关：</a>
  
  
<br>[⬆ Back to top](#table-of-contents)

---
#### <a name='css-interviews'>CSS 相关：</a>

* 1、<a name='css_1'>并列布局的方式</a>
```
核心思路是怎么让block不自适应平铺为整行。触发bfc就可以了；比如绝对布局，float，inline-block等等
```

  
<br>[⬆ Back to top](#table-of-contents)

---
#### <a name='css3-interviews'>CSS3 相关：</a>

  
<br>[⬆ Back to top](#table-of-contents)

---
#### <a name='js-interviews'>JS 相关：</a>

* 1、<a name='js_1'>0.1 + 0.2？</a>
* 2、<a name='js_2'>函数与构造函数的区别？</a>
```
没啥区别，区别都是new调用做的，改了this的指向而已
```

* 3、<a name='js_3'>数值怎么存储？64位浮点型；“小数怎么存储？”</a>
* 4、<a name='js_4'>写一个js的通用事件绑定函数</a>


  
<br>[⬆ Back to top](#table-of-contents)

---
#### <a name='react-interviews'>React 相关：</a>

##### 1、<a name='react_1'>react、vue原理，两者的区别还有基本的思想</a>
* react、vue原理，两者的区别还有基本的思想
```
于我来说最直观的是写法的区别，jsx与模板；同时debug中也存在差异。
再有就是框架实现思想上的区别了，数据绑定与diff
```

* 2、<a name='react_2'>react怎么优化？</a> 

* 3、<a name='react_3'>react的思想是什么？</a> 
```
数据驱动balabala，举了一个之前封装轮播图的例子
```

  
<br>[⬆ Back to top](#table-of-contents)

---
#### <a name='app-interviews'>移动端 相关：</a>

* 1、<a name='app_1'>移动端做过什么优化么？</a>
```
直出、域名收敛

域名收敛？为什么要收敛？”“因为dns解析慢啊？”“那和pc端有什么区别，
pc端域名不是发散来提高并发数么？”

 我心里一想是啊，其实浏览器pc和m没啥区别那为啥一个发散一个收敛，
 或者说发散我们都知道克服pc浏览器的并发限制。那m端？我当时有点迷没说上来就过了，
 回来又百度了一下感觉上其实就是m端网速慢dns太耗时。。我没反应过来还有网速的事情
```

* 2、<a name='app_2'>js与native怎么交互？</a>
```
嗯虽然我没做过，但是我了解过应该是native定义一套协议，js使用该协议发请求，
native拦截解析并返回js的所需balabala
```

* 3、<a name='app_3'>缓存策略都有哪些，包括native？</a>
```
基于node的微小服务——细说缓存与304
https://github.com/Aaaaaaaty/Blog/issues/8
```

* 4、<a name='app_4'>m端与pc在html5的新特性上有哪些是不一样的？</a>
```
基于node的微小服务——细说缓存与304
https://github.com/Aaaaaaaty/Blog/issues/8
```


  
<br>[⬆ Back to top](#table-of-contents)

---
#### <a name='redux-interviews'>Redux 相关：</a>

* 1、<a name='redux_1'>对redux怎么看？</a>
```angular2html
从时间旅行的乌托邦，看状态管理的设计误区
https://juejin.im/post/5a37075051882527a13d9418
```
  
<br>[⬆ Back to top](#table-of-contents)

---
#### <a name='vue-interviews'>Vue 相关：</a>

  
<br>[⬆ Back to top](#table-of-contents)

---

### call

Given a key and a set of arguments, call them when given a context. Primarily useful in composition.