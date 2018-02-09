# Front_end_Interview
前端面试汇总

## <a name='table-of-contents'>目录</a>

 1. [HTML 相关](#html-interviews)
 1. [CSS 相关](#css-interviews)
 1. [JS 相关](#js-interviews)
 1. [React 相关](#react-interviews)
 1. [Vue 相关](#vue-interviews)
 1. [移动端 相关](#app-interviews)
 1. [Node 相关](#node-interviews)
 1. [其他 相关](#other-interviews)


---
#### <a name='html-interviews'>HTML 相关：</a>
  
  
<br>[⬆ Back to top](#table-of-contents)

---
#### <a name='css-interviews'>CSS 相关：</a>

* 1、<a name='css_1'>并列布局的方式</a>

        核心思路是怎么让block不自适应平铺为整行。触发bfc就可以了；比如绝对布局，float，inline-block等等


* 2、说一下CSS盒模型

        IE的怪异盒模型和标注浏览器的盒模型，然后可以通过box-sizing属性控制两种盒模型的变换。


* 3、box-sizing的应用场景
* 4、弹性FLEX布局

        各种概念和属性能想到的说了一大堆，也扯到了Grid布局


* 5、一个未知宽高元素怎么上下左右垂直居中

        flex弹性布局的实现，说了一下兼容性，扯到了postcss的一些东西，然后说了一下常规的兼容性比较好的实现。

  
 
 
<br>[⬆ Back to top](#table-of-contents)

---
#### <a name='js-interviews'>JS 相关：</a>

* 1、<a name='js_1'>0.1 + 0.2？</a>
        
        0.1 + 0.2 != 0.3
        
        简单解释：
        首先这个不是JavaScript设计的问题，C、C++、Java、Python等遵循IEEE 754标准都必然会产生这个问题。
        JavaScript中的小数采用的是双精度（64位）表示，由三部分组成：符号(1位) + 指数(11位) + 尾数(52位)，
        浮点数二进制表示, 有以下两个原则:
        
            * 整数部分对 2 取余然后逆序排列
            * 小数部分乘 2 取整数部分, 然后顺序排列
        
        在二进制中0.1表示为0.0001100110011001100110011001100110011001100110011001…..（后面全是 1001 循环）
        双精度浮点数只有52位有效数字，从第53位开始，就舍入了。这样就造成了“浮点数精度损失”问题。
        
        解决方案：
        使用内置的函数toPrecision或toFixed来保留一定的精度。
        
            (0.1 + 0.2).toPrecision(10) == 0.3
            > true
            
            (0.1 + 0.2).toFixed(10) == 0.3
            > true
        
        延伸问题：
        1、单精度（32位）、双精度（64位）三部分组成位数分别是多少？
        2、二进制中整数、实数分别如何存储
        3、0.1 + 0.2 + 1.0 = 1.3，为什么？
        4、toPrecision、toFixed如何使用


* 2、<a name='js_2'>函数与构造函数的区别？</a>

        没啥区别，区别都是new调用做的，改了this的指向而已


* 3、<a name='js_3'>数值怎么存储？64位浮点型；“小数怎么存储？”</a>
* 4、<a name='js_4'>写一个js的通用事件绑定函数</a>
* 5、原型链，对象，构造函数之间的一些联系
* 6、DOM事件的绑定的几种方式
* 6、DOM事件中target和currentTarget的区别
* 6、你平时怎么解决跨域的。以及后续JSONP的原理和实现以及cors怎么设置

* 6、深拷贝的实现原理
        
        这个也还好，就是考虑的细节不是很周全，先是说了一种JSON.stringify和JSON.parse的实现，
        以及这种实现的缺点，主要就是非标准JSOn格式无法拷贝以及兼容性问题，然后问了我有么有用过IE8的一个什么JSON框架，
        我也不记得是什么了，因为我压根没听过，然后说了一下尾递归实现深拷贝的原理，还问了我typeof null是啥，这个当然是Object。。


* 6、什么是函数柯里化？以及说一下JS的API有哪些应用到了函数柯里化的实现？
 
        这个我就说了一下函数柯里化一些了解，以及在函数式编程的应用，最后说了一下JS中bind函数和数组的reduce方法用到了函数柯里化。



<br>[⬆ Back to top](#table-of-contents)

---
#### <a name='react-interviews'>React 相关：</a>

* 1、<a name='react_1'>react、vue原理、基本思想和区别</a>

        于我来说最直观的是写法的区别，jsx与模板；同时debug中也存在差异。再有就是框架实现思想上的区别了，数据绑定与diff


* 2、<a name='react_2'>react怎么优化？</a> 

* 3、<a name='react_3'>react的思想是什么？</a> 

        数据驱动balabala，举了一个之前封装轮播图的例子


* 4、对redux怎么看？

    [从时间旅行的乌托邦，看状态管理的设计误区](https://juejin.im/post/5a37075051882527a13d9418)


* 5、webpack的入口文件怎么配置，多个入口怎么分割？
* 8、有没有自己写过webpack的loader,他的原理以及啥的
* 8、有没有去研究webpack的一些原理和机制，怎么实现的。
* 6、webpack配置用到webpack.optimize.UglifyJsPlugin这个插件，有没有觉得压缩速度很慢，有什么办法提升速度

        一个想到是缓存原理，压缩只重新压缩改变的，还有就是减少冗余的代码，压缩只用于生产阶段，然后面试官问还有呢？
        我就说，还可以从硬件上提升，可以得到质的飞跃，比如换台I9处理器的电脑。。。。


* 7、node写的多入口怎么配置？
* 8、项目用到了Babel的一个插件：transform-runtime以及stage-2，说一下他们的作用

        ES的一些API，比如generator啥的默认不转换，只转换语法，需要这个来转换，然后说profill啥的，扯了一下stage-1，stage-2，stage-3，


* 8、babel把ES6转成ES5或者ES3之类的原理是什么，有没有去研究。

  
<br>[⬆ Back to top](#table-of-contents)

---
#### <a name='vue-interviews'>Vue 相关：</a>

  
<br>[⬆ Back to top](#table-of-contents)

---
#### <a name='app-interviews'>移动端 相关：</a>

* 1、<a name='app_1'>移动端做过什么优化么？</a>
        
        直出、域名收敛
        
        域名收敛？为什么要收敛？”“因为dns解析慢啊？”“那和pc端有什么区别，
        pc端域名不是发散来提高并发数么？”
        
        我心里一想是啊，其实浏览器pc和m没啥区别那为啥一个发散一个收敛，
        或者说发散我们都知道克服pc浏览器的并发限制。那m端？我当时有点迷没说上来就过了，
        回来又百度了一下感觉上其实就是m端网速慢dns太耗时。。我没反应过来还有网速的事情
                                                

* 2、<a name='app_2'>js与native怎么交互？</a>
        
        嗯虽然我没做过，但是我了解过应该是native定义一套协议，js使用该协议发请求，
        native拦截解析并返回js的所需balabala


* 3、<a name='app_3'>缓存策略都有哪些，包括native？</a>
        
        基于node的微小服务——细说缓存与304
        https://github.com/Aaaaaaaty/Blog/issues/8


* 4、m端与pc在html5的新特性上有哪些是不一样的？
        
        基于node的微小服务——细说缓存与304
        https://github.com/Aaaaaaaty/Blog/issues/8


<br>[⬆ Back to top](#table-of-contents)


---
#### <a name='node-interviews'>Node 相关：</a>

  
<br>[⬆ Back to top](#table-of-contents)

---
#### <a name='node-interviews'>其他 相关：</a>

* 1、有没有了解http2.0,websocket,https，说一下你的理解以及你所了解的特性
* 1、了解http协议。说一下200和304的理解和区别
        
        协商缓存和强制缓存的区别，流程，还有一些细节，提到了expires,Cache-Control,If-none-match,Etag,last-Modified的匹配和特征


* 1、git大型项目的团队合作，以及持续集成啥的
        
        这里我就说了一下自己了解的git flow方面的东西，因为没有实战经验，所以我就选择性说明了这一块的不熟练，然后面试官也没细问。


* 2、说一下你项目中用到的技术栈，以及觉得得意和出色的点，以及让你头疼的点，怎么解决的
* 2、说一下项目中觉得可以改进的地方以及做的很优秀的地方？
  
<br>[⬆ Back to top](#table-of-contents)
