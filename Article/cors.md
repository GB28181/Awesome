# 背景

浏览器出于安全问题考虑，是禁止不同域名下的js页面相互访问的，也就是平时说的“同源策略”。但是，有时候我们又想a.html和b.html两个页面的JS相互通信，那么怎么去解决这个跨域问题呢？从前端着手的解决方式有以下：
* [window.postMessage](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/postMessage)方法
* 设置document.domain属性
下面，我们来介绍下设置document.domain这一种方法

## `document.domain`实现（同一一级域名下的）跨域问题

## 场景1：b.html想访问a.html里的js对象window.apiObj
要实现这样一种前端跨域通信，如何做呢？
* 首先，必要满足的条件是两个页面的一级域名是一样的
  * a.html页面所在域名：a.example.com
  * b.html页面所在域名：b.example.com
* 在a.html页面中设置：`<script>document.domain = 'example.com'</script>`
* 在b.html页面中设置：`<script>document.domain = 'example.com'</script>`

注意：两个页面都需要设置相同的一级域名`document.domain = 'example.com'`才能生效！

## 场景2：a.html里的iframe页面f.html想想问a.html里的的js对象window.apiObj
其实，这个实现的逻辑也是一样的，满足的条件跟上面的一致，按同样的方式操作即可。

## 参考资料
[postMessage的浏览器兼容性](https://caniuse.com/#search=postMessage)

---

文章来源： [前端跨域CORS](https://github.com/ivonzhang/my-fed-essay/wiki/%E5%89%8D%E7%AB%AF%E8%A7%A3%E5%86%B3js%E8%B7%A8%E5%9F%9F%E9%97%AE%E9%A2%98) 编辑：[Edward](https://github.com/crazybber) 
