---
title: android调用系统相机
date: 2018-02-07 15:31:04
categories: android
tags: android
---
android调用系统相机的两种方法，一种是直接调用获取bitmap，另一种是指定文件，直接获取此文件。
<!-- more -->
## 方法一
```java
Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
startActivityForResult(intent, 101);
``` 
这样就完成了系统相机的调用，当然，在这之前需要获取相机权限，注意6.0以上的动态权限获取，网上很多，这里不再展示。那么拍完照怎么接受回调呢？

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if(resultCode == RESULT_OK && requestCode == 101){
        Bundle bundle = data.getExtras();
        Bitmap bitmap = (Bitmap) bundle.get("data");
        imageView.setImageBitmap(bitmap);
    }
}
```
这样就可以用拿到拍照的数据了，很简单，就这几行代码就结束了。
这种方式只需要android.permission.CAMERA这一种权限。可能有些同学不满足，希望不仅得到照片，还想再图库中显示，也有办法，奉上API
```java
MediaStore.Images.Media.insertImage(this.getContentResolver(),file.getAbsolutePath(), fileName, description);
```
这个api的作用是把一个文件插入到图库中。需要注意的是，是一个文件，不是一个bitmap，所以，后面该怎么做不用多说了吧，就是把刚才得到的bitmap写入一个文件，然后插入图库。还有一点需要注意，这里的fileName并不是插入图库后的文件名，系统用了一个时间戳代替。至此就完成了插入图库，但是你翻到图库，可能并找不到这张图，那是因为图库没有更新。
```java
Intent intent = new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE);
Uri contentUri = FileProvider.getUriForFile(this, FILE_PROVIDER, file);
intent.setData(contentUri);
sendBroadcast(intent);
```
这样就会通知相册，去更新资源，加载进来我们insert的图片。到这里从拍照，获取结果，在系统相册展示就结束了。需要注意的是，后我们有对相册的插入操作，还是个文件，所以，需要app获取读写权限。
## 方法二
```java
Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
File file = new File(getFilePath("test.jpg"));
filePath = file.getAbsolutePath();
Uri contentUri = FileProvider.getUriForFile(this, FILE_PROVIDER, file);
intent.putExtra(MediaStore.EXTRA_OUTPUT,contentUri);
startActivityForResult(intent,5);
```
这个方法跟前一个最大的区别是，我们指定了一个文件给系统相机。
第二行代码有个getFilePath()方法
```java
private String getFilePath(String fileName){
    String parent = getExternalFilesDir(Environment.DIRECTORY_PICTURES).getPath();
    return parent+ "/"+fileName;
}
```
这个就是获取app专属的SD卡目录，不需要获取权限的哦！说道这里，就多说几句。App的专属缓存目录分为SD卡和内存
#### SD卡缓存目录（App专用）
```java
// /storage/emulated/0/Android/data/app_package_name/files/Pictures
Content.getExternalFilesDir(Environment.DIRECTORY_PICTURES); 
// /storage/emulated/0/Android/data/app_package_name/cache
Content.getExternalCacheDir(); 
```
#### 内存缓存目录（App专用）
```java
Content. getCacheDir(); //  /data/data/app_package_name/cache
Content. getFilesDir(); //  /data/data/app_package_name/files
```
这几个目录的特点是，读写不需要获取权限，因为是在自己app领土范围内，当然可以为所欲为。第二是这么目录的内容，都会随着app的卸载二自动删除。

#### 外部存储路径（非专用）
```java
///storage/emulated/0
Environment.getExternalStorageDirectory().getPath();
```
这个就是所有app，只要获取了读写权限都能访问的地方了。但是不得不说，这个地方，还是少用为妙。一般来讲，app专用空间已经足够使用了，守住自己的一亩三分地就行了，app之所以沙箱化，就是不为了好管理内存。况且用这个地方，还要申请权限，有些用户会很敏感。我更是碰到了个奇葩问题。我们的app竟然获取的就是这里的目录，所以拍照的时候需要读写和拍照权限，在小米5动态获取权限的时候，我是一次性申请了拍照和读写。但是拍完照死活拿不到结果。各种尝试都不行，最后灵机一动，把权限顺序变一下，先读写再拍照，就行了。。。。泪奔。所以说，没有特殊需求，不用用这里的空间，出事了自己兜着。

回到拍照，这次拿回调就简单了，我们只需要在resultCode == RESULT_OK 的时候判断我们刚那个文件是否存在就行了。然后后面保存到相册之类的，就同第一种方法了。不过我们不需要bitmap转文件这一步了，因为我们这次获取到的本就是文件，推荐第二种方法。
最后提一下，我上面用到了FileProvider，这个是google为加强文件安全做的，是一个contentProvider的子类，具体配置google一下，很多的。
