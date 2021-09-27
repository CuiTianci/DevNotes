
Android10.0以下，需要获取WRITE_EXTERNAL_STORAGE权限。
```java
 /**
     * 将图片保存至InternalStorage/Pictures目录，自动被扫描到系统相册。
     *
     * @param context  Context for getting a ContentResolver
     * @param bmp      待保存的Bitmap
     * @param fileName 文件存储名称
     * @return 是否成功存储。
     */
    public static boolean saveImageToGallery(Context context, Bitmap bmp, String fileName) {
        // 拿到 MediaStore.Images 表的uri
        Uri tableUri = MediaStore.Images.Media.EXTERNAL_CONTENT_URI;
        // 创建图片索引
        ContentValues values = new ContentValues();
        values.put(MediaStore.Images.Media.DISPLAY_NAME, fileName);
        values.put(MediaStore.Images.Media.MIME_TYPE, "image/jpeg");
        // 在AndroidQ及以上版本，支持在Pictures目录下，创建子目录。
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
            values.put(MediaStore.Images.Media.RELATIVE_PATH, "Pictures/QRScanner");
            values.put(MediaStore.Images.Media.IS_PENDING, 1);
        }
        // 手动设置图片被添加进Pictures目录的时间戳。
        values.put(MediaStore.Images.Media.DATE_ADDED, System.currentTimeMillis());
        /*values.put(MediaStore.Images.Media.DATE_TAKEN, System.currentTimeMillis());
        values.put(MediaStore.Images.Media.DATE_MODIFIED, System.currentTimeMillis());*/
        // 将该索引信息插入数据表，获得图片的Uri
        Uri imageUri = context.getContentResolver().insert(tableUri, values);
        try {
            ContentResolver resolver = context.getContentResolver();
            // 通过图片uri获得输出流
            OutputStream os = resolver.openOutputStream(imageUri);
            // 图片压缩保存，quality为100，无损压缩。
            bmp.compress(Bitmap.CompressFormat.JPEG, 100, os);
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
                values.clear();
                values.put(MediaStore.Video.Media.IS_PENDING, 0);
                resolver.update(imageUri, values, null, null);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            bmp.recycle();
        }
        return false;
    }
```
