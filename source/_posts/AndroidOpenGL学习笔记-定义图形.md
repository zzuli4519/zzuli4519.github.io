---
title: AndroidOpenGL学习笔记(定义图形)
date: 2016-03-03 15:08:33
tags: AndroidOpenGl
---

### Defining Shapes  

`OpenGL-ES`允许在三维坐标系里定义绘制图形，定义一个图形必须首先定义它的坐标，在`AndroidOpenGl`中需要定义顶点坐标数组信息，为了更加高效的使用定点坐标信息，需要将数据保存到`ByteBuffer`中，这些将在`OpenGLEs`绘制的过程中用到。  

```java 
	public class Triangle {

    private FloatBuffer vertexBuffer;

    // number of coordinates per vertex in this array
    static final int COORDS_PER_VERTEX = 3;
    static float triangleCoords[] = {   // in counterclockwise order:
             0.0f,  0.622008459f, 0.0f, // top
            -0.5f, -0.311004243f, 0.0f, // bottom left
             0.5f, -0.311004243f, 0.0f  // bottom right
    };

    // Set color with red, green, blue and alpha (opacity) values
    float color[] = { 0.63671875f, 0.76953125f, 0.22265625f, 1.0f };

    public Triangle() {
        // initialize vertex byte buffer for shape coordinates
        ByteBuffer bb = ByteBuffer.allocateDirect(
                // (number of coordinate values * 4 bytes per float)
                triangleCoords.length * 4);
        // use the device hardware's native byte order
        bb.order(ByteOrder.nativeOrder());

        // create a floating point buffer from the ByteBuffer
        vertexBuffer = bb.asFloatBuffer();
        // add the coordinates to the FloatBuffer
        vertexBuffer.put(triangleCoords);
        // set the buffer to read the first coordinate
        vertexBuffer.position(0);
    }
}
```  
在`OpenGL`绘制的坐标系中，用`[0.0.0](x,y,z)`来制定容器绘制帧布局的中心位置，`[1,1,0]`代表帧布局最右边上角位置，`[-1,-1,0]`代表了帧布局左下角容器位置。详细说明可以参见[OpenGl说明] [OpenGl]  

定义一个复杂了图形，同样也是需要定义图形各个定点的坐标，并且需要提供绘制定点的顺序，如绘制一个矩形，最简单的绘制方式是利用以上的三角形代码来进行绘制，如以下的代码所示：  

```java
	public class Square {

    private FloatBuffer vertexBuffer;
    private ShortBuffer drawListBuffer;

    // number of coordinates per vertex in this array
    static final int COORDS_PER_VERTEX = 3;
    static float squareCoords[] = {
            -0.5f,  0.5f, 0.0f,   // top left
            -0.5f, -0.5f, 0.0f,   // bottom left
             0.5f, -0.5f, 0.0f,   // bottom right
             0.5f,  0.5f, 0.0f }; // top right

    private short drawOrder[] = { 0, 1, 2, 0, 2, 3 }; // order to draw vertices

    public Square() {
        // initialize vertex byte buffer for shape coordinates
        ByteBuffer bb = ByteBuffer.allocateDirect(
        // (# of coordinate values * 4 bytes per float)
                squareCoords.length * 4);
        bb.order(ByteOrder.nativeOrder());
        vertexBuffer = bb.asFloatBuffer();
        vertexBuffer.put(squareCoords);
        vertexBuffer.position(0);

        // initialize byte buffer for the draw list
        ByteBuffer dlb = ByteBuffer.allocateDirect(
        // (# of coordinate values * 2 bytes per short)
                drawOrder.length * 2);
        dlb.order(ByteOrder.nativeOrder());
        drawListBuffer = dlb.asShortBuffer();
        drawListBuffer.put(drawOrder);
        drawListBuffer.position(0);
    }
}
```




[OpenGl]:("http://developer.android.com/guide/topics/graphics/opengl.html#faces-winding")