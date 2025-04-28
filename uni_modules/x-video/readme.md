# x-video 

### 介绍

封装原生video，重写了控制条，包含播放暂停、静音、进度条、倍速菜单、垂直全屏、缓冲Loading、断点播放。

### 使用说明

- 控制播放可以点击左下按钮和视频区域
- 进度条使用的内置组件slider，在videoLoaded事件根据播放时间滑动，可手动拖动
- progress，表示初始时间占比，例：0.1，就是10%，控制断点播放
- step，表示当前时间占比，例：5，每5%跳一次，取值必须大于0且为整数
- 控制条会在5秒后过渡隐藏，点击视频区域重新显示
- 运行到微信小程序videoSrc要为http/https开头，预览时超出体积限制需删除test.mp4

### 安装方式

本组件符合[easycom](https://uniapp.dcloud.io/collocation/pages?id=easycom)规范，`HBuilderX 2.5.5`起，只需将本组件导入项目，在页面`template`中即可直接使用，无需在页面中`import`和注册`components`。

```html
<template>
	<view>
		<x-video ref="videoPlayer" :src="videoSrc" :progress="progress" @play="videoPlay"
						 @pause="videoPause" @ended="videoEnded" @timeupdate="videoTimeUp" @loadeddata="videoLoaded"
						 @seeking="videoSeeking" @seeked="videoSeeked" @error="videoError"/>
	</view>
</template>

```
```javascript

export default {
	data() { 
		return {
			videoSrc:"",
			progress:0
		}
	},
	methods:{
		// 视频信息加载完成
		videoLoaded(durationTime) {},
		// 当前播放时间
		videoTimeUp(currentTime) {},
		// 点击原始播放
		videoPlay() {},
		// 正在拖动
		videoSeeking() {},
		// 拖动结束
		videoSeeked() {},
		// 触发暂停
		videoPause() {},
		// 播放结束
		videoEnded() {},
		// 播放错误
		videoError() {}
	}
}
```

### API 

| 属性名		     | 类型		  |可选值	 | 默认值	               | 说明						|
| :-:				    | :-:			|:-:		|:-:		               | :-:						|
|videoId				| String	|-			|myVideo			         | 视频ID					|
|src				    | String	|-			|-			               | 视频地址					|
|autoplay		    | Boolean	|-			|true		               | 自动播放					|
|poster			    | String	|-			|-			               | 视频封面					|
|progress		    | Number	|-			|-			               | 初始播放进度			|
|width			    | String	|-			|100%		               | 视频宽度					|
|height			    | String	|-			|484rpx	               | 视频高度					|
|step				    | Number	|-			|1			               | 步长							|
|errorTip		    | String	|-			|播放错误	              | 播放错误Toast提示	|
|showRate		    | Boolean	|-			|true				           | 是否展示倍速			  |
|playbackRates	| Array  	|-			|[0.8, 1, 1.25, 1.5, 2]	| 播放速率	       |