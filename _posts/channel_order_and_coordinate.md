## android中color的顺序

argb

可以看`android.graphics.Color.argb()`源码

```java
public static int argb(int alpha, int red, int green, int blue) {
        return (alpha << 24) | (red << 16) | (green << 8) | blue;
}
```

int 高位是Alpha，然后rgb

bitmap中也是使用color的顺序



## openCV中color的顺序

Mat表示的是矩阵，也就是，每个channel是什么意思，其实depend on coder.

这里列举经常遇到的：`Utils.bitmapToMat`和`Utils.matToBitmap` 或者mat.put()

```java
Utils.bitmapToMat(bitmap, mat);
double[] result = mat.get(row, col);
//result[0]=R  result[1]=G result[2]=B result[3]=A

//更直观：
mat.put(0,0,new int[]{Color.argb(a,r,g,b)})//假设a,r,g,b是int
result = mat.get(0, 0)
// result[0] = r; result[1] = g; result[2] = b; result[3] = a
  
// Utils.matToBitmap 也会按照上面 0123 -> 3012转
```



注意，

1. bitmap 和 mat互转，Mat只能是8U4C 8U3C 和8U1C
2. 不需要考虑大端小端，无关



## openGL

openGl通过glReadPixels把里面的内容读到一个int 的buffer中，按照abgr存储



## 坐标

1. opengl 的y轴和cv, bitmap是相反的
2. mat.get(row, col) bitmap.get(x, y), 如果mat和bitmap由同一张图构成，mat.get(a, b) 应该和 bitmap.get(b, a) 对应。