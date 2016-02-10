###GestureDetector.OnGestureListener

#####onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY)

distance很奇怪，是初位置减去末位置，也就是说
向左(X) 上(Y)正，向右 下负，


##`public boolean onTouch(View v, MotionEvent event)`

这里event的坐标是指当前View内部的坐标，无论View是怎样Rotate，Translate,scale，左上角是0，0，坐标指内部坐标，也就是说，event的getX()getY()不受Rotate，Translate,scale操作影响

转换成父控件的坐标：
getMatrix().mapPoints()

而getHintRect这个方法是回根据Rotate，Translete，Scale这些方法去变化Hint的位置大小的，而这个HintRect的意义在于父View用来判断是否把一个touch事件传给这个view，在这个rect中才给他。（dispatchTouch方法中）


###关于android的Layout和Touch

android的位置是通过onLayout()方法决定的，onLayout方法通过设置(mLeft,mTop,mRight,mBottom)来决定他的layout位置，但是View是可以通过translate，rotate，scale这些变化位置，形状，而这些变化不会影响(mLeft,mTop,mRight,mBottom)。而会把这些操作记录在一个matrix中，可以通过getMatrix()方法获取到这个matrix，也可以通过getRotation,getX,getTranslateX,getScale这些方法分开获取。虽然这些操作不会影响(mLeft,mTop,mRight,mBottom)，但会影响事件接收的Rect，如上所述的getHintRect()

###获取绝对坐标
View.getLocationOnScreen() and/or getLocationInWindow()

左上角是view的坐标  

http://androidbin.iteye.com/blog/1633966