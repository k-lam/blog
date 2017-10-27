### sum(axis=n)  min(axis=n)

By default, these operations apply to the array as though it were a list of numbers, regardless of its shape. However, by specifying the axis parameter you can apply an operation along the specified axis of an array

这是官网的解释，什么叫做along the specified axis?

来看下面的：

```
>>>a = np.arange(4).reshape(2,2)
>>>a
array([[0, 1],
       [2, 3]])
>>>a.sum(axis=0)
array([2, 4])
>>>a.sum(axis=1)
array([1, 5])
```

axis = n,就是固定住其他维不变，n维变，譬如一个shape=(2,2) array(其实就是rows = 2 cols = 2的矩阵)的 a.sum(axis=0)结果会是，result = [X,Y] 。 X = a(0,0) + a(1,0)。Y= a(0,1) + a(1,1)这里的a(x,y)的(x,y)是索引。也就是along the specified axis，这个specified axis作为最后一个for循环。

再看

```
>>> a = np.arange(27).reshape(3,3,3)
>>> a
array([[[ 0,  1,  2],
        [ 3,  4,  5],
        [ 6,  7,  8]],

       [[ 9, 10, 11],
        [12, 13, 14],
        [15, 16, 17]],

       [[18, 19, 20],
        [21, 22, 23],
        [24, 25, 26]]])
>>> a.sum(axis=0)
array([[27, 30, 33],
       [36, 39, 42],
       [45, 48, 51]])
```

按照之前的。axis=0是指固定住其他维度，0维变。	看看为什么a.sum(axis=0)第一个数是27。假设结果是R，R一定是R的维数一定是a的维数-1（因为用了其中一个维度做聚合运算啊）。R可以设为R=R(x,y)

R(0,0) = a(0,0,0) + a(1,0,0) + a(2,0,0)

推广到一般情况：

R(x,y) = a(0,x,y) + a(1,x,y) + a(2, x ,y )

推广到更一般的情况

聚合预算符 f

数组a,而且 a.ndim = n而 a.shape=(s0,s1,…sn-1)

求R = a.f(axis=I)，0<=I<n

我们假设一个get()函数

可以知，R.ndim = n - 1，所以设R的元素是

R(i0,i1,…in-2) = f()



## [Indexing, Slicing and Iterating](https://docs.scipy.org/doc/numpy-dev/user/quickstart.html#indexing-slicing-and-iterating)

```
>>> c = np.arange(10)
>>> c[8]=-10
>>> d = np.ones(5,dtype=np.int32)
>>> d[1] = 8
>>> c
array([  0,   1,   2,   3,   4,   5,   6,   7, -10,   9])
>>> d
array([1, 8, 1, 1, 1], dtype=int32)
>>> c[d]
array([  1, -10,   1,   1,   1])
```

c[d]： d是一个长度比c小的ndarray，c[d]的意思是，用d中的element作为c的索引取值，组成一个大小是d.length但数据来自c的ndarray.