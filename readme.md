UI绘制是一个比较耗时的操作，特别当页面相对比较复杂或者有一些很大的资源的时候，比如大图什么的。所以我们希望可以把一些相对不重要的部分延迟显示，不要阻塞了用户的操作，或者阻塞了关键UI的显然导致掉帧卡顿。实现思路，我们借助SurfaceView的特性来实现UI绘制和显示的分离从而避免卡顿现象。


### SurfaceView和View对比

* View适用于主动更新的情况，而SurfaceView主要适用于被动更新，例如频繁的刷新。
* View在主线程中对画面进行刷新，而SurfaceView通常通过一个子线程来进行页面的刷新。
* View在绘制时没有使用双缓冲机制，而SurfaceView在底层实现机制中就已经实现了双缓冲机制
* SurfaceView允许你在非ui线程中去绘制。
* SurfaceView的帧率可以操作60FPS
* 在要求实时性比较高的游戏开发中，显然，view的ondraw是满足不了你的，这时候只能是用SurfaceView。
* SurfaceView的刷新处于主动，有利于频繁的更新画面。
* SurfaceView的绘制在子线程进行，避免了UI线程的阻塞。
* SurfaceView在底层实现了一个双缓冲机制，效率大大提升

### 双缓冲机制
双缓冲可以理解为有两个线程轮番去解析视频流的帧图像，当一个线程解析完帧图像后，把图像渲染到界面中，同时另一线程开始解析下一帧图像，使得两个线程轮番配合去解析视频流，以达到流畅播放的效果。

### SurfaceView使用

其实SurfaceView也是继承自View,只不过实现了双缓冲的机制。

* 1、首先这个自定义的SurfaceView类必须继承SurfaceView实现SurfaceHolder.Callback接口。
* 2、实现SurfaceHolder.Callback中的三个SurfaceView生命周期，分别为：

```java
void surfaceDestroyed(SurfaceHolder holder)：当SurfaceHolder被销毁的时候回调。
void surfaceCreated(SurfaceHolder holder)：当SurfaceHolder被创建的时候回调。
void surfaceChange(SurfaceHolder holder)：当SurfaceHolder的尺寸发生变化的时候被回调。
```
* 3、在surfaceCreated方法里开启一个子线程。
* 4、在这个子线程中开启一个由Flag控制的While循环，用于不断地绘制。
* 5、在循环中通过SurfaceHolder对象的lockCanvas方法获得一个Canvas对象用于绘制。
* 6、每次绘制完成通过SurfaceHolder对象unlockCanvasAndPost方法传入Canvas对象完成更新。
* 7、最后要在surfaceDestroyed方法中去改变while循环的Flag为false，结束子线程的绘制。

### 特性
SurfaceView内部实现了双缓冲的机制，但是实现这个功能是非常消耗系统内存的。因为移动设备的局限性，Android在设计的时候规定，SurfaceView如果为用户可见的时候，创建SurfaceView的SurfaceHolder用于显示视频流解析的帧图片，如果发现SurfaceView变为用户不可见的时候，则立即销毁SurfaceView的SurfaceHolder，以达到节约系统资源的目的。

* 不适合频繁显示隐藏的控件。

### 使用场景
* 一些频繁刷新的view。
* 一些复杂的，绘制一次很耗时并且会造成卡顿的view。
* 大的背景图，广告图，gif播放。。。
* 对于一些游戏画面，或者摄像头预览、视频播放


