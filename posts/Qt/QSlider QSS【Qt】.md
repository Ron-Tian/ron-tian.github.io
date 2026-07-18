---
id: qt-qslider-qss
title: "QSlider QSS 样式美化"
date: 2026-07-18
tags: Qt,QSS
excerpt: "使用 QSS 美化 QSlider 滑动条： groove 槽、handle 滑块、sub-page 已填充部分的样式定制。"
cover: linear-gradient(135deg, #667eea 0%, #764ba2 100%)
---

```json
//滑动条槽（整体）的美化
QSlider::groove:horizontal{ 
  height: 12px; 
  left: 0px; 
  right: 0px; 
  border:0px;    //指定无边框
  border-radius:6px;    //指定圆角
  background:rgba(0,0,0,50);
} 
//滑块的美化
QSlider::handle:horizontal{ 
  width:  50px; 
  height: 50px; 
  margin-top: -20px; 
  margin-left: 0px; 
  margin-bottom: -20px; 
  margin-right: 0px; 
  border-image:url(:/res/images/setting_slider_handle.png);
} 
//已滑过的进度美化
QSlider::sub-page:horizontal{
  background:rgba(80,166,234,1);
}

QSlider#nodeVolumeSlider{
	background-color:transparent;
	height: 15px;
}

QSlider#nodeVolumeSlider::groove:horizontal {  
	background-color:#383838;
	height: 3px;
    border-radius: 2px;
} 

QSlider#nodeVolumeSlider::handle:horizontal {
	background: white;
	width: 16px;
	height: 16px;
    margin: -3px 0;
	border-radius: 4px;
}
QSlider#nodeVolumeSlider::handle:horizontal:hover {
	background: #FF6D00;
	width: 16px;
	height: 16px;
	margin: -3px 0; 
	border-radius: 4px;
}

```

<!-- 这是一张图片，ocr 内容为：定义和用法 MARGIN简写属性在一个声明中设置所有外边距属性.该属性可以有1到4个值. 说明 这个简写属性设置一个元素所有外边距的宽度,或者设置各边上外边距的宽度. 块级元素的重直接的重直接会合井,而行内元表交际上不占上下外边距,行内元素的的应存外边起不会合井,同样地,浮动元素的 外边距也不会合并.允许指定负的外边距值,不过使用时要小心. 注释:允许使用负值. 例子1 X 5PX 15PX 20PX; MARGIN:10PX 上外边距是10PX 右外边距是5PX 下外边距是15PX 左外边距是20PX 例子2 MARGIN:10PX SPX 15PX; 上外边距是10PX 右外边距和左外边距是5PX 下外边距是15PX 例子3 MARGIN:10PX SPX; 上外边距和下外边距是10PX 右外边距和左外边距是5PX -->
![](https://cdn.nlark.com/yuque/0/2022/png/1443385/1657765718413-dd22cde6-452a-480c-a952-f6c20e7a0192.png)















