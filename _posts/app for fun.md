前言：自己做一些喜欢的东西是很有趣的事情，建立模型，建立规则，不断学习。譬如发现，动画其实就是基于时间的函数！齐次坐标为什么要把原点设在中心。这种探索，发现，模拟的过程真的很赞。一开始觉得创作点东西，后来才明白，我们从来没创造过，只是发现了原理，然后compose出来

##App : Snow
###background
Christmas gift

###难点与分解
* 2D的图片，怎么才能有一点立体的效果
* 下雪的动画
* 怎么循环出现，在哪里出现，雪花的分布

####立体的效果
1. 我发现，图片对于Z轴的偏移不能超过30度，这样就会有一点立体的感觉
2. 离屏幕越远，大小和透明度同时变小

####下雪的动画
雪是 **飘** 下来的，而且我想给它一点活跃的感觉

1.定义了6种动画，出现和消失，增加跳跃感。直接上代码吧




    public AnimatorSet pickOne(float fdis,int fdur,int rn,int srn,int sn,int ld,int duration){
	    int n = SnowGenerator.random(0,5);
	       // n = 2;
	    AnimatorSet animatorSet = new AnimatorSet();
	    AnimatorSet.Builder builder = animatorSet.play(getFadeIn());
	    switch (n) {
		    case 0://左下飘
			    builder.with(getFloatDown(fdis, fdur)).with(getFloatLeft(ld).setDuration(duration));
			    break;
		    case 1://旋转下降
			    builder.with(getFloatDown(fdis, duration)).with(getRotation(rn,duration));
			    break;
		    case 2://缩放
			    builder.with(getFloatDown(fdis, fdur)).with(getScale(sn,duration));
			    break;
		    case 3://向左缩放
			    builder.with(getFloatDown(fdis, fdur)).with(getScale(1,duration)).with(getFloatLeft(ld / 2));
			    break;
		    case 4://surround  有点傻逼,不飘落会更漂亮,但必须在屏幕中间区域
			    //builder.with(getFloatDown(fdis, fdur)).with(getSurround(srn));
			    view.setY(view.mfy * 1.5f);
			    builder.with(getSurround(srn).setDuration(duration));
			    break;
		    case 5://左右转  左右切换的时候太生硬了
			    builder.with(getFloatDown(fdis, fdur)).with(getTRotation().setDuration(duration));
			    break;
		    //case  6://alpha 试试和2一起
			    //builder.with(getAlpha(3)).with(getFloatDown(fdis,fdur));
			    //break;
	    }
	       // animatorSet.setDuration(duration);
	    builder.with(getFadeOut(duration - 500));
	       // AnimatorSet animatorSet1 = new AnimatorSet();
	    //animatorSet1.play(animatorSet).before(getFadeOut());
	    
	    return animatorSet;
    }

####怎么循环出现，在哪里出现，雪花的分布
怎么出现？怎么循环？read the fucking source。
雪花的分布，这个可以考究一下。一开始用均匀分布。后来总觉得，应该屏幕中间应该多一点雪花。想到了自然分布（怎么由均匀分布转换成自然分布或其他分布，google找公式吧）。结果不理想，然后就还是用均匀分布了。

####总结：
建模是很好玩的东西，主要是花时间在调整动画的参数上面



##App : Star
###背景
刚认识一个女孩，知道她看电影《小王子》。我去看了后，觉得里面的星星场景好漂亮，于是告诉她，我想做这么一个锁屏（邪恶的笑容）。。。于是，项目开始了
###难点与分解
* 锁屏，这个好解决，网上很多
* 星星闪烁
* 星星在天空中的分布

通过做圣诞节app，我知道，我无法解决素材问题，所以还是上网搜索星星的图标，然后通过控制亮度`ColorMatrix`

闪烁的效果可以接受，但是有一个问题，没有空间感！而且找回来的星星，效果不怎样。两种方案

1. 3D模型，用openGL。但是要自己建3D模型
2. 2.5D，但是坐标系，投影，模拟模型要自己搞

对于建3D模型这件事，我还是放弃了，因为最基本的设计我都不会，还要搞3D的。。。所以选择方案二

2.5D？这显然不是一个编程难题，通过之前做手表试戴的经验。我决定找设计帮忙。在我的诱惑下，设计告诉了我如下几点

* 如果有一颗星星感觉上是近一点的，设计上，他旁边的星星应该饱和度低一点。
* 远一点的星星可以适当调低一点透明度
* 为了逼真一点，要做边缘模糊

于是，我就在设计师的指导下，开始ps之路

![star1](https://raw.githubusercontent.com/k-lam/blog/gh-pages/image/kstar00.png)
![star2](https://raw.githubusercontent.com/k-lam/blog/gh-pages/image/kstar2.png)

这就是我的ps作品

然后我们就开始了2.5D投影模型的建模之路

###投影坐标系
这个我不是很担心，因为openGL的坐标系统很完善，我们模拟就是，实际上，也花了点功夫

* 使用齐次坐标系，所以原点必须在中心
* 通过far，near确定齐次坐标的第三个值

###光强模型
我发现gl的光强模型是要确定三个参数，这三个参数为什么是自己定，而不是应该有物理公式确定的吗？于是我上网搜索了一下，发现物理上根本没有这个概念。然后偶然间，在NASA找到星星闪烁的原因是因为云层经过，遮挡了光线。呃。。。这样说，觉得根本不需要考虑不同距离的光衰减了。

###分布模型
这个我是专门上知乎问了，[传送门，开！](https://www.zhihu.com/question/36974677?from=profile_question_card)

其中

> 旋涡星系的星星分布呈旋涡状，椭圆星系的星星分布呈椭球状，不规则星系的星星分布呈不规则状

这些的确很有趣。

一开始还考虑球体空间均匀分布投影到平面的情况。后来用上面的投影坐标系后，就简化模型了

##没完成
为什么没完成？因为我发现，效果不好看。就简单的是，用小灯泡来闪，比你做这么一个app更加漂亮！