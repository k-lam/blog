### 卷积

[知乎：怎样通俗易懂地解释卷积？](https://www.zhihu.com/question/22298352)

等车无聊答了玩，班门弄斧求轻喷...我先后一共在四个场景下遇到过卷积：1. 微分方程，2. 傅立叶变换及其应用，3. 概率论，4. 卷积神经网。在不同场景的应用前面的人已经答的很好了，我先不展开了，讲一下我印象最深刻的一个简单理解，是微分方程公开课上prof举的一个栗子，和上面打人的栗子其实差不多，不过这个更现实，故事细节有点模糊，记不清的地方只能凭脑补了。。大概就是他的女儿是做环保的，有一次她接到一个项目，评估一个地区工厂化学药剂的污染（工厂会排放化学物质，化学物质又会挥发散去），然后建模狮告诉她药剂的残余量是个卷积。她不懂就去问她爸爸，prof就给她解释了。假设t时刻工厂化学药剂的排放量是f(t) mg，被排放的药物在排放后Δt时刻的残留比率是g(Δt) mg/mg；那么在u时刻，对于t时刻排放出来的药物，它们对应的Δt=u-t，于是u时刻化学药剂的总残余量就是∫f(t)g(u-t)dt，这就是卷积了。

作者：Nirvana AC

链接：https://www.zhihu.com/question/22298352/answer/43685716

来源：知乎

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



[图像卷积与滤波的一些知识点](http://blog.csdn.net/zouxy09/article/details/49080029)



### 傅里叶

[如果看了此文你还不懂傅里叶变换，那就过来掐死我吧【完整版】](http://blog.jobbole.com/70549/)



### [欧几里得距离](https://zh.wikipedia.org/wiki/%E6%AC%A7%E5%87%A0%E9%87%8C%E5%BE%97%E8%B7%9D%E7%A6%BB)

![\|{\vec  {x}}\|_{2}={\sqrt  {|x_{1}|^{2}+\cdots +|x_{n}|^{2}}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/56a66e2fe1f9b5e41cd5cb7d7e0fd549705f9d99)





### [范数 norm](https://zh.wikipedia.org/wiki/%E8%8C%83%E6%95%B0#.E6.AC.A7.E5.87.A0.E9.87.8C.E5.BE.97.E8.8C.83.E6.95.B0)





### [牛顿迭代法](https://baike.baidu.com/item/%E7%89%9B%E9%A1%BF%E8%BF%AD%E4%BB%A3%E6%B3%95)

[雅可比矩阵](https://en.m.wikipedia.org/wiki/Jacobian_matrix_and_determinant)

[泰勒展开](https://zh.wikipedia.org/wiki/%E6%B3%B0%E5%8B%92%E5%85%AC%E5%BC%8F)

[如何通俗易懂地讲解牛顿迭代法求开方？](https://www.zhihu.com/question/20690553/answer/146104283)

[Gauss–Newton algorithm](https://en.m.wikipedia.org/wiki/Gauss%E2%80%93Newton_algorithm)

[Gauss-Newton算法学习](http://blog.csdn.net/jinshengtao/article/details/51615162)

[Newton's method in optimization](https://en.m.wikipedia.org/wiki/Newton%27s_method_in_optimization)  牛顿法的推导过程主要看这个



## 插值

[插值-interpolate](http://hyry.dip.jp/tech/book/page.html/scipynew/scipy-610-interpolate.html)