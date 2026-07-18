---
id: qt-qss-styles
title: "部分 QSS 样式集合"
date: 2026-07-18
tags: Qt,QSS
excerpt: "常用 QSS 样式片段集合：对话框标题栏、按钮、滚动条、输入框等组件的美化样式。"
cover: linear-gradient(135deg, #c471f5 0%, #fa71cd 100%)
---

```css
QWidget#dialogTitle
{
border-top-left-radius:5px;
border-top-right-radius:5px;
background-color: qlineargradient(spread:pad, x1:0, y1:0, x2:0, y2:1, stop:0 #3E3E3E, stop:0.95 #222222 , stop:1.0 #0C0E10);
border-left: 1px solid #5B5B5B;
border-right: 1px solid #5B5B5B;
border-top: 1px solid #5B5B5B;
}
QWidget#dialogCenter
{
background: #262626;
border-left: 1px solid #5B5B5B;
border-right: 1px solid #5B5B5B;
}

QWidget#dialogBottom
{
border-top: 1px solid #0C0E10;
border-bottom-left-radius:5px;
border-bottom-right-radius:5px;
background: #262626;
border-left: 1px solid #5B5B5B;
border-right: 1px solid #5B5B5B;
border-bottom: 1px solid #5B5B5B;
}
QPushButton#dialogAcceptBtn,QPushButton#dialogCancelBtn,QPushButton#dialogRejectBtn
{
width:58px;
height:25px;
border: 1px solid #FF6D00;
border-radius: 12px;
background: transparent
}

QPushButton#dialogAcceptBtn:hover
{
background: #414141;
}
QPushButton#dialogRejectBtn:hover,QPushButton#dialogCancelBtn:hover
{
background: rgba(255,255,255,0.10);
}

QPushButton#dialogAcceptBtn:pressed
{
background: #383838;
}
QPushButton#dialogRejectBtn:pressed,QPushButton#dialogCancelBtn:pressed
{
background: #383838;
}
QLabel#dialogTipLabel
{
font-family:"Microsoft YaHei";
font-size:14px;
color: #EFEFEF;
background-color:transparent;
padding-left:20px;
}
QTextEdit
{
border: 0px;
padding-left: 10px;
background: transparent;
}

/* 垂直滚动条 */
QScrollBar:vertical
{
background: #2c2c2c;
width:8px;
margin: 0px 0px 0px 0;
}

/* 滚动条中滑块的样式 */
QScrollBar::handle
{
background: #83878E;
border: 1px solid #83878E;
border-radius: 3px;
/*width:8px;
min-height:20px;
*/
}

/* 设置下箭头 */
QScrollBar::add-line:vertical
{
border: 0px;
height: 0px;
/* width:10px; */
}

/* 设置上箭头 */
QScrollBar::sub-line:vertical
{
border: 0px;
height: 0px;
/* width:10px; */
}

/* 当滚动条滚动的时候，上面的部分和下面的部分 */
QScrollBar::add-page,QScrollBar::sub-page
{
background:#323742;
/* background:rgb(150,116,125,10%);
border-radius:4px; */
}

/* 水平滚动条 */
QScrollBar:horizontal
{
border: 0px;
background-color: #323742;
height: 8px;
margin: 0px 0px 0px 0px;
}

QScrollBar::add-line:horizontal
{
border: 0px;
width: 0px;
}

QScrollBar::sub-line:horizontal
{
border: 0px;
width: 0px;
}

QListView{
border:0px;
/* border-top:1px solid #0C0E10;; */
}

QListView::item{
background:transparent;
height:25px;
}
QListView::item:hover{
background:#4D4D4D;
}
QListView::item:selected{
border: 1px solid #FF6D00;
color:#FFFFFF;
background: #4D4D4D;
}

/* 编辑输出的样式只有放到这里才能在对话框中有效 */

/* 舞台的背景 */

QWidget#kStageBase{
background: #090909;
border:0px;

}

QWidget#stage{
background: #090909;
border:0px;
}

QWidget#optionForm,QWidget#propertyform {
background: transparent;
border:0px;
height: 26px;
}

QLabel {
background: transparent;
border: 0px;
font-size: 14px;
font-family: "MicrosoftYaHei";
}

/* X，Y，宽度，高度，旋转角度的QSpinBox控件的样式 */
QSpinBox{
height:22px;
width:40px;
background: #161616;
border: 1px solid #686868;
border-radius: 2px;
font-size: 14px;
font-family: "MicrosoftYaHei";
color: #FFFFFF;
selection-background-color: #AC4A01;
}
QSpinBox::up-button
{
border: 0px;
image: url(:/img/Resources/image/up_arrow.png);
/* background: #161616; */
border-radius: 1px;
height: 11px;
}
QSpinBox::down-button{
border: 0px;
image: url(:/img/Resources/image/down_arrow.png);
/* background: #161616; */
border-radius: 1px;
height: 11px;
}
QSpinBox::up-button:disabled{
border: 0px;
image: url(:/img/Resources/image/up_arrow_disable.png);
background: #323232;
border-radius: 1px;
height: 11px;
}
QSpinBox::down-button:disabled {
border: 0px;
image: url(:/img/Resources/image/down_arrow_disable.png);
background: #323232;
border-radius: 1px;
height: 11px;
}

QSpinBox:hover{
background: #161616;
border: 1px solid #D8D8D8;
border-radius: 2px;
}
QSpinBox:focus{
background: #222328;
border: 1px solid #FF6D00;
border-radius: 2px;
}
QSpinBox:disabled{
background: #323232;
border: 1px solid #545454;
border-radius: 2px;
}

QPushButton#dragBtn {
width: 24px;
height: 24px;
border: 0px;
image:url(:/img/Resources/image/icon_drag_normal.png);
background-color:transparent;
}
QPushButton#dragBtn:hover {
image:url(:/img/Resources/image/icon_drag_hover.png);
}
QPushButton#dragBtn:pressed {
image:url(:/img/Resources/image/icon_drag_pressed.png);
}
QPushButton#dragBtn:checked {
width: 24px;
height: 24px;
border: 0px;
image:url(:/img/Resources/image/icon_drag_check_normal.png);
background-color:transparent;
}
QPushButton#dragBtn:checked:hover {
image:url(:/img/Resources/image/icon_drag_check_hover.png);
}
QPushButton#dragBtn:checked:pressed {
image:url(:/img/Resources/image/icon_drag_check_pressed.png);
}

QPushButton#zoomOutBtn {
width: 16.8px;
height: 16.8px;
border: 0px;
image:url(:/img/Resources/image/icon_zoomOut_normal.png);
background-color:transparent;
}
QPushButton#zoomOutBtn:hover {
image:url(:/img/Resources/image/icon_zoomOut_hover.png);
}
QPushButton#zoomOutBtn:pressed {
image:url(:/img/Resources/image/icon_zoomOut_pressed.png);
}
QPushButton#zoomInBtn {
width: 16.8px;
height: 16.8px;
border: 0px;
image:url(:/img/Resources/image/icon_zoomIn_normal.png);
background-color:transparent;
}
QPushButton#zoomInBtn:hover {
image:url(:/img/Resources/image/icon_zoomIn_hover.png);
}
QPushButton#zoomInBtn:pressed {
image:url(:/img/Resources/image/icon_zoomIn_pressed.png);
}

QPushButton#restoreBtn {
width: 24px;
height: 24px;
border: 0px;
image:url(:/img/Resources/image/icon_initpos_normal.png);
background-color:transparent;
}
QPushButton#restoreBtn:hover {
image:url(:/img/Resources/image/icon_initpos_hover.png);
}
QPushButton#restoreBtn:pressed {
image:url(:/img/Resources/image/icon_initpos_pressed.png);
}

QLabel#zoomValueLabel {
width: 37px;
height: 19px;
color: #FFFFFF;
}

QLabel#m_pBottomLeftSpacer{
background:transparent;
}

QLabel#m_pBottomCenterSpacer{
background:transparent;
}
```
