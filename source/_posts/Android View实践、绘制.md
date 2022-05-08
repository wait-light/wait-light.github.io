# 位置参数
View的left、top等参数都是根据父容器的位置来确定的，是相对于于父容器的差值。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1584075/1650433156851-2714ed75-a7b3-4973-b043-f6982d433f5b.png#clientId=u930a9d11-d55d-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=661&id=ue3af232a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=661&originWidth=1019&originalType=binary&ratio=1&rotation=0&showTitle=true&size=44643&status=done&style=none&taskId=udf6871ff-77f2-4f08-9239-a1895ee9168&title=View%E7%9A%84%E4%BD%8D%E7%BD%AE%E5%85%B3%E7%B3%BB&width=1019 "View的位置关系")
# 事件机制

（1）同一个事件序列是指从手指接触屏幕的那一刻起，到手指离开屏幕的那一刻结束，在这个过程 中所产生的一系列事件，这个事件序列以down事件开始，中间含有数量不定的move事件，最终以up事件结束。 

（2）正常情况下，一个事件序列只能被一个View拦截且消耗。这一条的原因可以参考。因为一 旦一个元素拦截了某此事件，那么同一个事件序列内的所有事件都会直接交给它处理，因此同一个事件序列中的事件不能分别由两个View同时处理，但是通过特殊手段可以做到，比如一个View将本该自己处理的事件通过onTouchEvent强行传递给其他View处理。

（3）某个View一旦决定拦截，那么这一个事件序列都只能由它来处理（如果事件序列能够传递给它 的话），并且它的onInterceptTouchEvent不会再被调用。这条也很好理解，就是说当一个View决定拦截一 个事件后，那么系统会把同一个事件序列内的其他方法都直接交给它来处理，因此就不用再调用这个 View 的onInterceptTouchEvent去询问它是否要拦截了。

（4）某个View一旦开始处理事件，如果它不消耗ACTION_DOWN事件（onTouchEvent返回了 false），那么同一事件序列中的其他事件都不会再交给它来处理，并且事件将重新交由它的父元素去处 理，即父元素的onTouchEvent会被调用。意思就是事件一旦交给一个View处理，那么它就必须消耗掉，否 则同一事件序列中剩下的事件就不再交给它来处理了，这就好比上级交给程序员一件事，如果这件事没有 处理好，短期内上级就不敢再把事情交给这个程序员做了，二者是类似的道理。 

（5）如果View不消耗除ACTION_DOWN以外的其他事件，那么这个点击事件会消失，此时父元素的 onTouchEvent并不会被调用，并且当前View可以持续收到后续的事件，最终这些消失的点击事件会传递给 Activity处理。 

（6）ViewGroup默认不拦截任何事件。Android源码中ViewGroup的onInterceptTouch-Event方法默认返回false。 

（7）View没有onInterceptTouchEvent方法，一旦有点击事件传递给它，那么它的onTouchEvent方法就会被调用。 

（8）View的onTouchEvent默认都会消耗事件（返回true），除非它是不可点击的（clickable 和 longClickable同时为false）。View的longClickable属性默认都为false，clickable属性要分情况，比如Button 的clickable属性默认为true，而TextView的clickable属性默认为false。 

（9）View的enable属性不影响onTouchEvent的默认返回值。哪怕一个View是disable状态的，只要它的 clickable 或者 longClickable 有一个为 true ，那么它的onTouchEvent就返回true。 

（10）onClick会发生的前提是当前View是可点击的，并且它收到了down和up的事件。 

（11）事件传递过程是由外向内的，即事件总是先传递给父元素，然后再由父元素分发给子View，通 过 requestDisallowInterceptTouchEvent 方法可以在子元素中干预父元素的事件分发 过程， 但是 ACTION_DOWN 事件除外。  

**事件传递顺序**
Activity -> Window -> View，即事件总是先传递给Activity，Activity再传递给Window，最后Window再传递给顶级View。顶级View接收到事件后，就会按照事件分发机制去分发事件。  

**事件分发机制伪代码**
```java
public boolean dispatchTouchEvent(MotionEvent ev) {
     boolean consume = false;
     if (onInterceptTouchEvent(ev)) {
         consume = onTouchEvent(ev);
     } else {
         consume = child.dispatchTouchEvent(ev);
     }
     return consume;
 }
```
通过上面的伪代码，我们也可以大致了解点击事件的传递规则：对于一个根ViewGroup来说，点击事件产生后，首先会传递给它，这时它的dispatchTouchEvent 就会被调用，如果这个ViewGroup的onInterceptTouchEvent方法返回true就表示它要拦截当前事件，接着事件就会交给这个ViewGroup 处理 ， 即它的 onTouchEvent 方法就会被调用； 如果这个 ViewGroup 的 onInterceptTouchEvent方法返回false就表示它不拦截当前事件，这时当前事件就会继续传递给它的子元素，接着子元素的dispatchTouchEvent方法就会被调用，如此反复直到事件被最终处理。  
# 绘制
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1584075/1650506726357-fe31b69d-ccb6-4dcb-a707-82d34b93cf74.png#clientId=u445b3926-f7d8-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=628&id=u32c160c1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=628&originWidth=866&originalType=binary&ratio=1&rotation=0&showTitle=true&size=195275&status=done&style=none&taskId=ucb20688b-cd2d-451d-bafd-b179d283503&title=performTraversals%E7%9A%84%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B%E5%9B%BE&width=866 "performTraversals的工作流程图")
通常情况下：measure决定View的宽和高，layout决定View的四个顶点坐标和实际的宽和高，draw进行实际的绘制。

在有标题栏的主题下，顶级View的结构如下。通常我们定义的内容被填充至content中。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1584075/1650507101599-0ba8e102-fe36-456f-94ca-c63e8d42e4b1.png#clientId=u445b3926-f7d8-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=524&id=udf438ca1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=524&originWidth=430&originalType=binary&ratio=1&rotation=0&showTitle=true&size=48841&status=done&style=none&taskId=u04206682-a971-4eda-aa85-a54b8de3d36&title=%E9%A1%B6%E7%BA%A7View%EF%BC%9ADecorView%E7%9A%84%E7%BB%93%E6%9E%84&width=430 "顶级View：DecorView的结构")
**空间模式**

1. UNSPECIFIED

父容器不对View有任何限制，要多大给多大，这种情况一般用于系统内部，表示一种测量的状态。

2. EXACTLY 父容器已经检测出View所需要的精确大小，这个时候View的最终大小就是SpecSize所指定的值。它对应于LayoutParams中的match_parent和具体的数值这两种模式。 

3. AT_MOST

父容器指定了一个可用大小即SpecSize，View的大小不能大于这个值，具体是什么值要看不同View的具体实现。它对应于LayoutParams中的wrap_content。  

![image.png](https://cdn.nlark.com/yuque/0/2022/png/1584075/1650507995039-0da47bc5-38c0-4aba-b17a-a53305bfbe99.png#clientId=u445b3926-f7d8-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=318&id=ua8ece41a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=318&originWidth=1121&originalType=binary&ratio=1&rotation=0&showTitle=true&size=104257&status=done&style=none&taskId=u44af998d-3615-47b2-8b30-f42789a88d7&title=%E6%99%AE%E9%80%9AView%E7%9A%84MeasureSpec%E7%9A%84%E5%88%9B%E5%BB%BA%E8%A7%84%E5%88%99&width=1121 "普通View的MeasureSpec的创建规则")

**来源**：[Android开发艺术探索-第三章、第四章](https://book.douban.com/subject/26599538/)

