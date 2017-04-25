# Shader

`Android` `Shader`

---

## 1. 简介

[官方定义](https://developer.android.google.cn/reference/android/graphics/Shader.html)：
```xml
Shader is the based class for objects that return horizontal spans of colors during drawing. A subclass of Shader is installed in a Paint calling
paint.setShader(shader). After that any object (other than a bitmap) that is drawn with that paint will get its color(s) from the shader.
```

Android 中对 Shader 是这样解释的：
```
Shader 是一种基类对象，它在图形绘制过程中返回一段段颜色值，通过调用 Paint.setShader() 方法，可以
将它的子类安装进画笔，这样 Paint 对象在绘制过程中所获取的颜色就是来自 Shader 对象。
```

上面提到了Shader的子类，Shader 有5个子类 `BitmapShader`, `ComposeShader`, `LinearGradient`, `RadialGradient`,`SweepGradient`。 本文的目的也是分别讲它的各个子类。

## 2. 图片渲染器 BitmapShader

**BitmapShader** 将一张图片当作纹理来绘制(在 `OpenGL` 中，纹理就是贴图的意思，可以理解为一个没有颜色的正方形被贴上了一张图片，这样视觉效果就是一张正方形的图片)。而这张图片可以通过设置 `BitmapShader` 的 `tiling mode` 来达到镜面和重复的效果。
```
BitmapShader(Bitmap bitmap, Shader.TileMode tileX, Shader.TileMode tileY)
```
上面是 `BitmapShader` 的构造方法。

* bitmap 是指纹理图片
* tileX 是指在X方向轴的 `tiling mode`
* tileY 是指在Y方向轴的 `tiling mode`

很多人可能有疑问，这个 `TileMode` 是什么？

### 1. 神秘莫测的TileMode

什么是 TileMode 呢？ 事实上它只是一个枚举而已。**它只有三个值**。

* **Shader.TileMode CLAMP**
* **Shader.TileMode MIRROR**
* **Shader.TileMode REPEAT**

#### 1. CLAMP

它的意思当要绘制的区间大于图片纹理本身的区间时，多出来的空间位置将被纹理图片的边缘颜色填充

#### 2. MIRROR

这个模式能够让纹理以镜像的方式在X和Y方向复制

#### 3. REPEAT

它的作用是将图片纹理沿XY轴进行复制。


---
[参考文献]

* [绘图Canvas十八般武器之Shader详解及实战](http://mp.weixin.qq.com/s/3SX79GGZFoIPoxIRugAkoQ)
* [Android绘图Canvas十八般武器之Shader详解及实战篇(下)](http://blog.csdn.net/briblue/article/details/53694042)
