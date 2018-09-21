# Lottie动画的入门和实践（2）Lottie动画的分段播放

## 前言

lottie动画的播放有很多种情况，可以循环播放，自动播放等等，不过我们将所有的播放类型分为两种。

1. 整体json文件动画播放，简单的说就是将json文件中的动画从第一帧播放到最后一帧。

   这个实现方式比较简单，直接使用lottie提供的api就可以轻松实现，本文不做过多说明。

2. 将json文件中的动画分段播放，分段播放的意思是将一个动画文件按照指定的帧数范围进行播放。

   对lottie动画使用时可以将多个相似的动画合并为一个大的动画然后进行分段播放。本文主要介绍如何使用lottie提供的api进行分段播放。


## Lottie分段播放

1. 单段播放

   单段播放主要是说，只播放一个json数据中某一段动画。可以理解为动效师提供了一个json格式的动画，我们只想这个动画中某一段。

   lottie中提供了一个api 专门处理指定节播放的功能。

   ```javascript
   const lot = lottie.loadAnimation(options)
   lot.playSegment([firstFrame, lastFrame],true)  //加载完动画之后直接使用playSegment进行播放。
   ```

2. 多段播放

   多段播放主要是说，播放一个json数据中的某几段动画。可以理解为动效师提供了一个json格式的动画，我们需要将这个动画拆分成多段进行播放。

   `lottie.playSegment()`可以接受两个参数，第一个参数表示要播放的段落帧数信息，第二个表示是否等待上一动画播放完毕播放。

   第一个参数有两种形式：

   1. `[num1,num2]`，`num1`和`num2`表示起始帧和结束帧
   2. `[[num1,num2],[num3,num4]]`，`num1`和`num2`表示第一个阶段起始帧和结束帧，`num3`和`num4`表示第一个阶段起始帧和结束帧。下文称为**段落队列**。

   ​

   在lottie播放动画时，有两个参数要介绍一下：

   1. `firstFrame`表示当前播放动画的第一帧，
   2. `currentFrame`表示**当前播放过程**的第n帧。相对整体json动画的帧来说是 `firstFrame + currentFrame`。

   ​

   在使用`playSegment()`播放段落队列时，lottie会自动将段落队列中一个元素的第一个值设置为`firstFrame`，将第二个值减去第一个值设置为`totalFrames`，当。

   在`playSegment()`执行的时候会定义一个变量`currentFrame`，这个变量是相对于`firstFrame`来定义的，表示当前播放过程中的当前帧数，`currentFrame`随着动画的播放会进行自增。

   下面以一个明确的帧数来说明一下多段播放时遇到的问题。

   例如执行这段代码`anim.playSegments([[169,240],[249,418]],true)`

   `[169,240]`这一段的播放过程的初始状态是，`firstFrame = 169,currentFrame=0,totalFrames=71`。

   此时动画的起始帧数是`absolutelyCurrent = firstFrame + currentFrame = 169`

   `currentFrame`自增。

   结束状态是，`firstFrame = 169,currentFrame=70,totalFrames=71`。

   这一段播放结束后，`currentFrame`不会归零，仍然会继续增加。

   `[249,418]`这一段的播放过程的初始状态是，`firstFrame = 249,currentFrame=70,totalFrames=129`。

   此时动画的起始帧数是`absolutelyCurrent = firstFrame + currentFrame = 319`， **并不是我们设定的249**。**因此在队列中除第一个元素之外其余元素的第一个值应该减去前面播放的帧数之和。**

   `currentFrame`自增。

   结束状态是，`firstFrame = 249,currentFrame=128,totalFrames=129`。

   ​

   ​

   ```javascript
   anim.addEventListener('segmentStart',function(){
     console.log('segmentStart',anim.firstFrame,anim.currentFrame)
   })
   anim.onComplete = function(){
     console.log('onComplete',anim.firstFrame,anim.currentFrame)
   }

   anim.playSegments([[169,240],[249,418]],true)
   //segmentStart 169 0.001
   //segmentStart 249 70.07360000000001
   //onComplete 249 168
   anim.playSegments([[169,240],[289,418]],true)
   //segmentStart 169 0.001
   //segmentStart 289 70.07042000000006
   //onComplete 289 128
   anim.playSegments([[169,240]],true)
   //segmentStart 169 0.001
   //onComplete 169 70
   anim.playSegments([[289,418]],true)
   //segmentStart 289 0.001
   //onComplete 289 128
   ```


**未完待续**