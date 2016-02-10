###draw views
* Draw之前必须先layout，确定位置
* Drawing begins with the root node of the layout.
* layout（ViewGroup）负责调用他的child view，按顺序draw（dispatchDraw），而这些view自己负责draw自己。layout会比child先draw
* ViewGroup中通过`protected boolean drawChild(Canvas canvas, View child, long drawingTime)`来调用 child的draw
* ViewGroup 和他的所有child view 共用同一块Canvas。如果自定义一个view，且要自己onDraw，那就用onDraw传入的canvas就好了

###draw sth on canvas
*	在canvas，可以draw的2D东西包括

1.	2D几何图形，如直线，正方形，多边形，椭圆等，通过canvas.drawXXX，或者通过Path，    canvas.drawPath这些方法。
2.	Bitmap

*	怎样draw？

	借鉴openGL。openGL中有好多坐标系，首先是Object Space，这个坐标系是用来构建一个Object的。然后是World Space，作用是把之前构造出来的物体，放到世界中，在这里就是放到canvas中。

	所以，我们在canvas画图的做法也分成两步：
	1.	构造Object（上面的2D图像或bitmap）
	2.	对canvas进行矩阵变换，把Object画到canvas上面。这里为什么是对canvas进行变换，是因为android提供了对canvas变换的api，而对Object变换的api相对缺少，需要自己进行建模



Paint.Style

	STROKE 描边 
	FILL 和 FILL_AND_STROKE 都是填充