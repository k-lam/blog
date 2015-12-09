启用MVP的原因：
之前MVC中C层太薄，所以导致view中太容易包含太多代码，如果界面业务逻辑复杂的（状态或步骤多于2的），用MVP模式。

##[简介](http://baike.baidu.com/view/3456444.htm)
在MVP里，Presenter完全把Model和View进行了分离，主要的程序逻辑在Presenter里实现。而且，Presenter与具体的View是没有直接关联的，而是**通过定义好的接口进行交互**

 如果要实现的UI比较复杂，而且相关的显示逻辑还跟Model有关系，就可以在View和Presenter之间放置一个Adapter。