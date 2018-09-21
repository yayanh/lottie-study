# Lottie动画的入门和实践（1）Lottie动画的介绍

## Lottie简介

lottie是Airbnb公司开源的一个动画库，能够在Android，iOS和web上解析AE通过Bodymovin导出的json文件。将解析后的帧数据使用svg，canvas，html三种方式显示处理。
	
本文主要介绍lottie在web中的使用。

## HelloWorld

1. 安装

   ``` yarn add lottie-web```

2. 简单使用

```javascript
   import lottie from 'lottie-web'
   let lot = new lottie()
   //载入lottie动画
   lot.loadAnimation({
       container: element,   //lottie动画挂载的元素
       renderer:'svg',  //渲染方式
       loop:true, //是否循环播放
       autoplay:true, //是否自动播放
       path:'data.json' //使用的json数据
   })
   
   //按照以上代码就可以将一个json数据中存储的数据，渲染成对应的动画显示到页面上。
```

## Lottie API

1.`loadAnimation()` 加载动画信息。

​    这个api是加载Lottie动画数据用的，主要属性有renderer指定渲染方式。

`path`指定json文件地址。

`animationData`指定json数据。

*以上两个属性选择一个使用。*

`assetsPath`指定资源地址，可以指定动画中img所在的文件夹，缺省表示json文件同级的images文件夹。

`loop`表示是否循环播放，缺省true，

`autoplay`表示是否自动播放，缺省true。

2. `play`用来播放动画。

通常配合`autoplay：false`时使用。默认从第一帧开始播放，到最后一帧，该第一帧表当前阶段的第一帧。

3. `goToAndPlay()` 表示跳转到指定帧，并开始播放。接受两个参数，第一个为数字，第二个表示第一个参数是指定帧数（`true`）还是进度百分百（`false`）（缺省）。该指定帧是相对当前阶段的第一帧计算。
4. `goToAndStop()` 表示跳转到指定帧。接受两个参数，第一个为数字，第二个表示第一个参数是指定帧数（`true`）还是进度百分百（`false`）（缺省）。该指定帧是相对当前阶段的第一帧计算。
5. `destroy()` 表示销毁元素，释放资源。

## Lottie Event

1. `'complete'` 在动画播放完成，会触发该动效。
2. `'loaded_images'` 图片资源载入完成后触发，可能会在DOMLoaded触发之后。
3. `'DOMLoaded'` DOM加载完成后触发。

```javascript
   const lot = lottie.loadAnimation(param)
   lot.addEventListener('DOMLoaded', () => {
   })
   //事件的使用方法
```
本文只是简单介绍相对重要的api和event，全部内容请看[lottie-web](https://github.com/airbnb/lottie-web)。

**未完待续**

