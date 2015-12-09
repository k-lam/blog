##Camera
###PreviewSize

其实就算preview 的 height 和 width的ratio和显示容器的长宽ratio一样，都不能保证不变形！（小米4和三星s5就变形了！）

1. 摄像头传感器可能只返回一种size的preview数据，再经过软缩放，裁剪变成你程序设置的的height和width，这种情况下，摄像头传感器返回的preview size一定是preview size中最大的。这种情况下，由于软缩放，所以还是存在变形的可能
2. 摄像头有硬缩放，裁剪的能力。或真的可以返回不同size 的preview size

所以如果是想通过摄像头preview数据来操作，为了不变形。最安全的方式就是用最大的preview size。

不过这个存在带宽问题，最大的preview，一定占用很大的内存。


###Preview和picture的mapping问题

http://stackoverflow.com/a/18159351/4617198

Intent调用相机


图片Uri存储在goToCamera的Uri中，注意调用相机的intent.putExtra(MediaStore.EXTRA_OUTPUT, uri);
这一句的MediaStore.EXTRA_OUTPUT要求的是

>The name of the Intent-extra used to indicate a content resolver Uri to be used to store the requested image or video.

也就是，必须是content://这种。而且注意，getContentResolver().insert(Media.INTERNAL_CONTENT_URI, cv);是不可以的，不可以用Internal。android没有开放这个权限。所以不能把相片放到app private的存储空间再放到android这个默认的contentPrivider里面。


	/**
     * open camera
     *
     * @return the file to store the picture
     */
    Uri goToCamera() {

        Uri uri = getContentResolverUri();
        if(uri == null){
            Toast.makeText(getActivity(),"无法读取外部存储器",Toast.LENGTH_LONG).show();
            return null;
        }
        Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
        intent.putExtra(MediaStore.EXTRA_OUTPUT, uri);
        startActivityForResult(intent, LazyCamOrAlbumDialog.CAPTURE_IMAGE_ACTIVITY_REQUEST_CODE);
        return uri;
    }

	  Uri getContentResolverUri(){

        SharedPreferences preferences = PreferenceManager
                .getDefaultSharedPreferences(getActivity());
        String uri_string = preferences.getString(Configuration.SharedPreferencesKey.TRYON_CAMERA_CONTENT_URI,"");
        Uri uri = null;
        if(uri_string.length() == 0){
            ContentValues cv = new ContentValues();
            cv.put(Media.TITLE,"dbg");
            try {
                uri = getActivity().getContentResolver().insert(Media.EXTERNAL_CONTENT_URI, cv);
            }catch (Exception ex){
                return null;
            }
            preferences.edit().putString(Configuration.SharedPreferencesKey.TRYON_CAMERA_CONTENT_URI,uri.toString()).commit();
        }else{
            uri = Uri.parse(uri_string);
        }
        return uri;
    }
    
    
    
###[返回的picture orientation问题](http://stackoverflow.com/a/13068627/4617198)
http://stackoverflow.com/questions/13062769/android-front-and-back-camera-captured-picture-orientation-issue-rotated-in-a-w
    
    
###把图片放大gallery

把图片放到gallery，这样其他app就能用到我们app生产的图片了。

	public class GalleryHelper {
    public void savePhoto(Context context,Bitmap bmp)
    {
        File imageFileFolder = new File(Environment.getExternalStorageDirectory(),"大表哥");
        imageFileFolder.mkdir();
        FileOutputStream out = null;
        Calendar c = Calendar.getInstance();
        String date = fromInt(c.get(Calendar.MONTH))
                + fromInt(c.get(Calendar.DAY_OF_MONTH))
                + fromInt(c.get(Calendar.YEAR))
                + fromInt(c.get(Calendar.HOUR_OF_DAY))
                + fromInt(c.get(Calendar.MINUTE))
                + fromInt(c.get(Calendar.SECOND));
        File imageFileName = new File(imageFileFolder, date.toString() + ".jpg");
        try
        {
            out = new FileOutputStream(imageFileName);
            bmp.compress(Bitmap.CompressFormat.JPEG, 100, out);
            out.flush();
            out.close();
            scanPhoto(context,imageFileName.toString());
            out = null;
        } catch (Exception e)
        {
            e.printStackTrace();
        }
    }


    public String fromInt(int val)
    {
        return String.valueOf(val);
    }

    MediaScannerConnection msConn;
    public void scanPhoto(Context context,final String imageFileName)
    {
        msConn = new MediaScannerConnection(context,new MediaScannerConnection.MediaScannerConnectionClient()
        {
            public void onMediaScannerConnected()
            {
                msConn.scanFile(imageFileName, null);
                //Log.i("msClient obj  in Photo Utility", "connection established");
            }
            public void onScanCompleted(String path, Uri uri)
            {
                msConn.disconnect();
                //Log.i("msClient obj in Photo Utility","scan completed");
            }
        });
        msConn.connect();
    }
	}	