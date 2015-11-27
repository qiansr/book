# CSS编码总体原则

* 从外部文件加载CSS，尽可能减少文件数。加载标签放在HEAD 部分。
* 用 LINK 标签加载，永远不要用@import。
* 加载样式表
```
<link rel="stylesheet" type="text/css" href="myStylesheet.css" />
```
* 不要用内联式样式，这样会很难追踪样式规则
```
    <p style="font-size: 12px; color: #FFFFFF">This is poor form, I say</p>
```
* 使用 reset.css，normalize.css等重置游览器默认样式。
* 定义样式的时候，对样式在页面只出现一次的元素用id，其他的用class。
* 理解 层叠和选择器的优先度。
* 避免使用性能开销大的CSS选择器。例如：* 通配符选择器，div#myid，table.results。后两种应该直接用#myid和.results代替。
