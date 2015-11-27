#Chrome控制台
[toc]
##控制台指令
|指令|作用|
|:-|:-|
|$_|表示最近一次表达式结果|
|$0~4|表示最近在DOM结点树上点选的5个DOM节点|
|$(selector)|document.querySelector() 的封装|
|$$(selector)|document.querySelectorAll() 的封装|
|monitor(fn)|监听一个函数，输出函数名与执行时传入的参数|
|unmonitor(fn)|停止监听一个函数|
|keys(object)|返回所有属性名组成的数组|
|values(object)|返回所有属性值组成的数组|

##console指令
|console指令|作用|
|:-|:-|:--|
|console.log|输出普通信息|
|console.info|输出提示性信息|
|console.error|输出错误信息|
|console.warn|输出警示信息|
|console.debug|输出调试信息|
|console.dir|将DOM以树结构输出|
|console.dirxml|用来显示网页的某个node所包含的html代码|
|console.group|输出一组信息的开头|
| console.groupEnd|输出一组信息的结束|
|console.assert|断言，只有表达式为false才输出|
|console.count|执行计数器，输出执行次数|
|console.profile配合console.profileEnd!|配合使用来查看CPU使用相关信息，在Profiles面板里面查看就可以看到cpu相关使用信息!|
|console.timeLine和console.timeLineEnd|配合一起记录一段时间轴|
|console.trace|堆栈跟踪相关的调试|
|查看上述具体API|https://developer.chrome.com/devtools/docs/console-api|



##console.log() 样式



|格式化符号| 实现的功能
|:-|:-|:--|
|%s|    格式化成字符串输出
|%d or %i   |格式化成数值输出
|%f |格式化成浮点数输出
|%o |转化成展开的DOM元素输出
|%O |转化成JavaScript对象输出
|%c |把字符串按照你提供的样式格式化后输入
添加格式化符号，后面跟css样式。
例子，输入：
>console.log('%cRainbow Text ', 'background-image:-webkit-gradient( linear, left top, right top, color-stop(0, #f22), color-stop(0.15, #f2f), color-stop(0.3, #22f), color-stop(0.45, #2ff), color-stop(0.6, #2f2),color-stop(0.75, #2f2), color-stop(0.9, #ff2), color-stop(1, #f22) );color:transparent;-webkit-background-clip: text;font-size:5em;');

效果：

![Alt text](../images/eAfAbqU.gif)


![Alt text](./1446621656777.png)







