---
title: TextureView
date: 2016-12-23 14:59:27
categories: android
tags: android
description: "A TextureView can be used to display a content stream. Such a content stream can for instance be a video or an OpenGL scene. The content stream can come from the application's process as well as a remote process."
---
A TextureView can be used to display a content stream. Such a content stream can for instance be a video or an OpenGL scene. The content stream can come from the application's process as well as a remote process.
<!-- more -->
## 概述
#### 官网：
A TextureView can be used to display a content stream. Such a content stream can for instance be a video or an OpenGL scene. The content stream can come from the application's process as well as a remote process.

TextureView can only be used in a hardware accelerated window. When rendered in software, TextureView will draw nothing.

Unlike SurfaceView, TextureView does not create a separate window but behaves as a regular View. This key difference allows a TextureView to be moved, transformed, animated, etc. For instance, you can make a TextureView semi-translucent by calling myView.setAlpha(0.5f).

Using a TextureView is simple: all you need to do is get its SurfaceTexture. The SurfaceTexture can then be used to render content. The following example demonstrates how to render the camera preview into a TextureView:
```java
public class LiveCameraActivity extends Activity implements TextureView.SurfaceTextureListener {
      private Camera mCamera;
      private TextureView mTextureView;

      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);

          mTextureView = new TextureView(this);
          mTextureView.setSurfaceTextureListener(this);

          setContentView(mTextureView);
      }

      public void onSurfaceTextureAvailable(SurfaceTexture surface, int width, int height) {
          mCamera = Camera.open();

          try {
              mCamera.setPreviewTexture(surface);
              mCamera.startPreview();
          } catch (IOException ioe) {
              // Something bad happened
          }
      }

      public void onSurfaceTextureSizeChanged(SurfaceTexture surface, int width, int height) {
          // Ignored, Camera does all the work for us
      }

      public boolean onSurfaceTextureDestroyed(SurfaceTexture surface) {
          mCamera.stopPreview();
          mCamera.release();
          return true;
      }

      public void onSurfaceTextureUpdated(SurfaceTexture surface) {
          // Invoked every time there's a new Camera preview frame
      }
  }
```
A TextureView's SurfaceTexture can be obtained either by invoking getSurfaceTexture() or by using a TextureView.SurfaceTextureListener. It is important to know that a SurfaceTexture is available only after the TextureView is attached to a window (and onAttachedToWindow() has been invoked.) It is therefore highly recommended you use a listener to be notified when the SurfaceTexture becomes available.

It is important to note that only one producer can use the TextureView. For instance, if you use a TextureView to display the camera preview, you cannot use lockCanvas() to draw onto the TextureView at the same time.
#### 翻译：
TextureView可以用来显示一个内容流。这个内容流可以是视频或者OpenGL的实例。这个内容流可以来自应用进程也可以来自远端。
TextureView仅在开启了硬件加速后才有效果，否则没有效果。
跟SurfaceView不同的是，TextureView没有创建新的窗口，而是作为一个普通的view存在。这个关键的不同点就赋予了TextureView能像view一样可以放大，缩小，平移，动画等特性。
想要使用TextureView非常简单，你只需要获取他的SurfaceTexture即可。然后SurfaceTexture负责渲染内容。下面的示例就是演示TextureView如何渲染相机的预览图
SurfaceTexture可以通过调用getSurfaceTexture()方法获取，也可以像上面程序那样，设置SurfaceTextureListener获取。在TextureView绑定到window后，也就是调用过onAttachedToWindow()后，我们必须知道SurfaceTexture是否可用，这是非常重要的。因此，强烈推荐你使用监听的方式，以便及时获取到可用的SurfaceTexture。
还有一件重要的事你需要注意。TextureView仅可以被一个生产者使用，例如，如果你用它显示相机预览图的时候，就不能使用lockCanvas()在上面绘画。至于lockCanvas()这个东西，自己google吧（这句是我加的哈）。

以上就是官方API对TextureView的介绍，翻译的很生硬吧，说好听点是为了保证原汁原味，说难听点就是没有文学气息，没有艺术细胞

接下来看下对API的介绍，只挑重要的说，不重要的自己看。比如buildLayer()，解释是：你掉了这个方法也没什么卵用。那我们还有必要拿出来说吗？
##API
getBitmap(int width, int height)
这个方法是获取当前TextureView中的内容，返回一个bitmap，并且可以指定宽高，其他重载方法就不多说了，无非是不设置宽高呗。有一个还是可以提一下的：getBitmap(Bitmap bitmap)，这个是自己定义一个bitmap，然后把TextureView的内容复制上去，再返回

getLayerType()
这个方法是返回view层的类型，此处总是返回LAYER_TYPE_HARDWARE.也就是说TextureView有一个硬件层。至于LAYER_TYPE_HARDWARE这个东西是什么，有什么用，可以去官网查。这里可以简单说：它相较于软件层的优点是，渲染速度快，通过缓存等一些手段，简化复杂的渲染方式

getSurfaceTexture()
这个前面其实已经说了，就是获取一个SurfaceTexture，SurfaceTexture的作用也说了，它才是绘制渲染内容流的东西。

getTransform(Matrix transform)
获取当前TextureView的变换形式，我们可以看到传入的是Matrix类型，返回的其实也是一个Matrix。这样就可以获取TextureView的变换信息了

isOpaque()
判断TextureView是否是不透明的，推荐子类重写

lockCanvas()
锁住画布，开始画画
其他的方法感觉都是View中公用的，可以自己去了解一下。
##应用
看完getBitmap方法，我就想到，这不就可以做个照相机了吗？于是在他的示例代码中加了几行，一个简易的照相机就出来了，当然，非常简易，照相机要做好，还是不简单的
```java
public class TestAty extends AppCompatActivity implements TextureView.SurfaceTextureListener, View.OnClickListener {

    private Camera camera;
    private TextureView textureView;
    private ImageView imageView;
    private Button btn;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_test_aty);
        textureView = (TextureView) findViewById(R.id.texture);
        btn = (Button) findViewById(R.id.btn);
        btn.setOnClickListener(this);
        imageView = (ImageView) findViewById(R.id.image);
        textureView.setSurfaceTextureListener(this);

        textureView.getBitmap();


    }

    @Override
    public void onSurfaceTextureAvailable(SurfaceTexture surface, int width, int height) {
        camera = Camera.open();
        camera.setDisplayOrientation(90);
        try {
            camera.setPreviewTexture(surface);
            camera.startPreview();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    @Override
    public void onSurfaceTextureSizeChanged(SurfaceTexture surface, int width, int height) {

    }

    @Override
    public boolean onSurfaceTextureDestroyed(SurfaceTexture surface) {
        camera.stopPreview();
        camera.release();
        return true;
    }

    @Override
    public void onSurfaceTextureUpdated(SurfaceTexture surface) {

    }

    @Override
    public void onClick(View v) {
        imageView.setImageBitmap(textureView.getBitmap());
    }
```
真的是没加几行代码
##总结
在android7.0说到SurfaceView和TextureView时是这么说的：
Android 7.0 可同步移动到 SurfaceView 类，此类在某些情况下提供的电池性能优于 TextureView：在渲染视频或 3D 内容时，包含滚动和动画视频位置的应用在使用 SurfaceView 时比 TextureView 耗电更少。
SurfaceView 类可减少屏幕合成对电池的消耗，因为它是在专用硬件中合成，与应用窗口内容分隔开。因此，它产生的中间副本少于 TextureView。
现在，SurfaceView 对象的内容位置和包含的应用内容同步更新。这一变化导致的一个结果是，在画面移动时，SurfaceView 中播放的视频的简单的平移或缩放不再在画面侧面产生黑条。
从 Android 7.0 开始，我们强烈建议您使用 SurfaceView 代替 TextureView，以实现省电。
可见，TextureView只是在一些需要view特性的场景中使用，否则的话，还是用SurfaceView吧



