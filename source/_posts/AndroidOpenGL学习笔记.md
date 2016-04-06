---
title: AndroidOpenGL学习笔记
date: 2016-03-03 14:48:49
tags: AndroidOpenGl
---

### 使用OpenGl流程  

首先需要在配置文件中声明需要使用OpenGl的版本信息，如以下所示：  
```java
<uses-feature android:glEsVersion="0x00020000" android:required="true" />  
```  
如果程序中还使用了`Texture compression`,同样的必须要声明所使用的`Text Compression`版本。确保能安装在正确的Android手机版本上  
```java
<supports-gl-texture android:name="GL_OES_compressed_ETC1_RGB8_texture" />
<supports-gl-texture android:name="GL_OES_compressed_paletted_texture" />   
```  
若程序中使用了OpenGl，必须在`GLSurfaceView`的基础上才能使用，下面的代码示例了如何在`Activity`中使用`OpenGl`  

```java  
public class OpenGLES20Activity extends Activity {

    private GLSurfaceView mGLView;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // Create a GLSurfaceView instance and set it
        // as the ContentView for this Activity.
        mGLView = new MyGLSurfaceView(this);
        setContentView(mGLView);
    	}
	}

```  

创建一个`GLSurfaceView` 对象  

```java  
class MyGLSurfaceView extends GLSurfaceView {

    private final MyGLRenderer mRenderer;

    public MyGLSurfaceView(Context context){
        super(context);

        // Create an OpenGL ES 2.0 context
        setEGLContextClientVersion(2);

        mRenderer = new MyGLRenderer();

        // Set the Renderer for drawing on the GLSurfaceView
        setRenderer(mRenderer);
    }
} 
```  
此外还要设置渲染进程更新的机制  

```java 
// Render the view only when there is a change in the drawing data
setRenderMode(GLSurfaceView.RENDERMODE_WHEN_DIRTY);
```  

创建一个`Renderer`类  

实现了`GlSurfaceView.Renderer`接口，控制了如何绘制SurfacevView，其中必须实现三个方法  

* `onSurfaceCreated()` - Called once to set up the view's OpenGL ES environment.
* `onDrawFrame()` - Called for each redraw of the view.
* `onSurfaceChanged()` - Called if the geometry of the view changes, for example when the device's screen orientation changes.  

下面为一个基本的`Renderer`代码示例  

```java 
public class MyGLRenderer implements GLSurfaceView.Renderer {

    public void onSurfaceCreated(GL10 unused, EGLConfig config) {
        // Set the background frame color
        GLES20.glClearColor(0.0f, 0.0f, 0.0f, 1.0f);
    }

    public void onDrawFrame(GL10 unused) {
        // Redraw background color
        GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT);
    }

    public void onSurfaceChanged(GL10 unused, int width, int height) {
        GLES20.glViewport(0, 0, width, height);
    }
}
```  