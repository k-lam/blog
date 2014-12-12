#[Canvcs](http://developer.android.com/reference/android/graphics/Canvas.html)

###clipXXX

裁剪画布，注意，裁剪后，只有在裁剪区域里面的，才会被显示
save/restore可以恢复

注意，多个clipXXX之间是相交关系和Modify关系

另外，Path.addRect这些方法  如果连个区域以上，不同手机的表现会出现不一致的情况