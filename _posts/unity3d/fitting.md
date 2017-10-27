Skin动画 Physique动画 **Bone** animations

**CharacterCustomization**

SkinnedMeshRenderer

> Unity uses the **Skinned Mesh Renderer** component to render **Bone** animations, where the shape of the Mesh is deformed by predefined animation sequences. This technique is useful for characters and other objects whose joints bend (as opposed to a machine where joints are more like hinges).

mesh合并，材质合并，骨骼筛选

HumanIK（3D骨骼）



[關於角色的頭髮跟衣服](https://forum.gamer.com.tw/Co.php?bsn=60602&sn=917)



[Merge skinned and non-skinned meshes](https://www.assetstore.unity3d.com/en/#!/content/25366)

## 方法一：

[Unity教程之-Unity3d换装之模型动画分离](http://www.unity.5helpyou.com/2706.html)

[Unity教程之-Unity3d人物换装之Mesh合并(材质合并)](http://www.unity.5helpyou.com/2708.html)

上面两篇主要介绍把人物分割成头 上身 下身。然后美工设计骨骼动画，我们分割再合成。另外包括mesh合并，减少DrawCall。但是没有说怎样换衣服，衣服怎么穿上



动画：美工设计骨骼动画， 程序员把骨骼动画分割  再合成





## 衣服模型

[衣服模型](https://www.assetstore.unity3d.com/en/#!/content/78106)

[布料](https://www.assetstore.unity3d.com/en/#!/content/81333)



unity做衣服的方法是通过一个cloth的组件，但是没有Obi Cloth好，建议下一步对比用。因为现在具体怎么做  都还是不知道。另外

[知乎](https://www.zhihu.com/question/30015408)

> 不知道是哪里看到的冷知识

> GTA V里面主角衣服的皱褶不是用物理引擎即时运算的动态凹凸纹理，而是两张交错的凹凸纹理，一张表示向左一张表示向右。随着主角躯干的转动，改变两边的透明度，躯干向左的时候把向右的纹理透明度变高，向左的变低，反之亦然。这样就用简简单单两个透明度参数体现了非常复杂的计算才能实现的视觉效果。

例图是躯干前部，但是原理一样。

作者：柴健翌

链接：https://www.zhihu.com/question/30015408/answer/46533945

来源：知乎

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

