 
##android.graphics.Matrix

android中画图涉及图形的变换，经常要用矩阵相乘的方法。而这里如果把向量写成竖向量的形式
（android就是可恶的用竖向量的写法，用getValue看看就知道了），应该左乘变换矩阵，而且后面的变换应该放到越左的位置。
如：先平移变换A，再缩放变换B，应该是B(AX)=BAX,android用可恶的pre和post来区分左乘右乘
还有一点要注意的就是缩放变换，不是根据中心来进行缩放的，缩放的意义就在于每个点乘以缩放矩阵，缩放的位置是会变换的

注意doc解释：

>public boolean postSkew (float kx, float ky)
>
>Postconcats the matrix with the specified skew. M' = K(kx, ky) * M

>public boolean postTranslate (float dx, float dy)

>Postconcats the matrix with the specified translation. M' = T(dx, dy) * M

现在我有一个(1,1)点，希望先xy都放大两倍，再向右上平移一个单位，应该是下面这样写

	float[] point = new float[]{1,1};
	Matrix matrix = new Matrix();
	matrix.postScale(2, 2,0,0);
	matrix.postTranslate(1, 1);
	matrix.mapPoints(point);

或者

	float[] point = new float[]{1,1};
	Matrix matrix = new Matrix();
	matrix.preTranslate(1, 1);
	matrix.preScale(2, 2,0,0);
	matrix.mapPoints(point);

Matrix的理解：

可以作一个简单的理解：Matrix修改的不是点，而是坐标系！在经过Matrix修改后的坐标上绘点，会在原坐标上什么位置出现。

	Matrix m = new Matrix();
	m.setValues(new float[]{1,2,3,4,5,6,7,8,9});

	调试结果：
	Matrix{[1.0, 2.0, 3.0][4.0, 5.0, 6.0][7.0, 8.0, 9.0]}

####Translate：
正方向为正，负方向为负，和是哪个坐标系无关

####scale
scale就是缩放因子

####rotate：
这里就恶心了，如果坐标是y轴向下

rotate(degree)代表顺时针(clockwise)转degree

如果是y轴向上，代表逆时针转degree！

	Matrix m = new Matrix();
	m.postRotate(90, 0, 0);
	float[] point = new float[]{1,1};
	m.mapPoints(point);
	//结果point={-1,1}

rotate(-degree)就相当于向放方向转了

注意rotate后，用mapRect
rect的

		Matrix m = new Matrix();
		RectF rect = new RectF(0, 0, 1, 1);
		m.postRotate(90, 0, 0);
		m.mapRect(rect);
		//结果：b:1 l:-1 r:0 t:0