---
id: qt-custom-dialog
title: "重写对话框"
date: 2026-07-18
tags: Qt,QSS
excerpt: "自定义对话框的实现：无边框窗口、标题栏拖拽、关闭按钮、QSS 样式美化完整代码示例。"
cover: linear-gradient(135deg, #f6d365 0%, #fda085 100%)
---

```cpp
#pragma  once

/*!
* @file dialog.h
* @date 2019/06/15 15:34
*
*
* @brief 重写对话框
*/

#include <QDialog>
#include <QPushButton>
#include <QLayout>
#include <QSpacerItem>
#include "titlebar.h"

class DialogWidget :public QDialog
{
Q_OBJECT
public:
explicit DialogWidget(QWidget* parent);
~DialogWidget();

enum enDialogCode { enAccept, enReject, enCancel };
enum enDialogType { enQuestion, enInformation, enWarning, enCritical };
enum enDialogButton { AcceptBtn = 0x0004, RejectBtn = 0x8000, CancelBtn = 0x4000, NoBtn = 0x0000 };

public:
void setDialotType(enDialogType type);
void setTitle(const QString& title);
void showClose();
void setAcceptText(const QString& text);
void setRejectText(const QString& text);
void setCancelText(const QString& text);
void setCenterWidgetMargins(int left, int top, int right, int bottom);
void setStandardButtons(int btnType);

void setInfor(const QString& text, Qt::Alignment alignment = Qt::AlignLeft | Qt::AlignVCenter);
QPushButton* standardButton(int btnType);
void setCenterWidget(QWidget* w);
enDialogCode run();


signals:
void signalAccept();
void signalReject();
void signalCancel();
protected:
protected:
void changeEvent(QEvent* e) override;
private slots:
void slotAccept();
void slotReject();
void slotCancel();

private:
void init();
void setRejctVisible(bool visible);
void setCancelVisible(bool visible);
void setAccepVisible(bool visible);
void translateLanguage();

private:
QHBoxLayout* m_HLayoutBottom = nullptr;
QVBoxLayout* m_VLayoutMain = nullptr;
QVBoxLayout* m_VLayoutCenter = nullptr;

QLabel* m_pBottomLeftSpacer = nullptr;
QLabel* m_pBottomCenterSpacer = nullptr;

QPushButton* m_pAcceptBtn = nullptr;
QPushButton* m_pRejectBtn = nullptr;
QPushButton* m_pCancelBtn = nullptr;
TitleBar* m_pTitileBar = nullptr;
QWidget* m_pBottomWidget = nullptr;
QWidget* m_pCenterWidget = nullptr;
QLabel* m_pTipLabel = nullptr;

enDialogCode m_currentCode;
enDialogType m_currentType;
};
```



```cpp
#include "dialog.h"
#include "macrofunc.h"
#include <QDebug>
#include <QStylePainter>
#include <QStyleOption>
#include <QEvent>
#include "../language.h"
#pragma execution_character_set("utf-8")

DialogWidget::DialogWidget(QWidget* parent /*= nullptr*/)
    : m_currentType(enQuestion)
        , QDialog(parent)
        , m_currentCode(enCancel)  // enCancel 作为默认值，按 ESC 键，release 版本大概率为接受状态，debug 版本为随机值几乎都是拒绝状态。【网站输入源】按键盘的ESC按键，可以添加网站输入源
    {
        setWindowModality(Qt::ApplicationModal);
        setModal(true);
        #ifdef Q_OS_WIN
        setWindowFlags(windowFlags() | Qt::FramelessWindowHint | Qt::CoverWindow);
        #else
        setWindowFlags(Qt::FramelessWindowHint);
        #endif
        
        
        setAcceptDrops(true);
        //setAttribute(Qt::WA_TranslucentBackground, true);
        setContextMenuPolicy(Qt::NoContextMenu);
        init();
        setTitle(zhtitleTip);
        m_pBottomLeftSpacer->hide();
        LOAD_STYLESHEET("dialog.qss");
        connect(m_pAcceptBtn, &QPushButton::clicked, this, &DialogWidget::slotAccept);
        connect(m_pRejectBtn, &QPushButton::clicked, this, &DialogWidget::slotReject);
        connect(m_pCancelBtn, &QPushButton::clicked, this, &DialogWidget::slotCancel);
        connect(m_pTitileBar, &TitleBar::signalButtonCloseClicked, this, &DialogWidget::slotCancel);
        
    }

DialogWidget::~DialogWidget()
{
}

void DialogWidget::init()
{
    m_HLayoutBottom = new QHBoxLayout;
    m_VLayoutMain = new QVBoxLayout;
    m_VLayoutCenter = new QVBoxLayout;
    
    m_pBottomWidget = new QWidget;
    m_pBottomWidget->setObjectName(tr("dialogBottom"));
    m_pCenterWidget = new QWidget;
    m_pCenterWidget->setObjectName(tr("dialogCenter"));
    
    m_pTipLabel = new QLabel(zhToolTipInformation);
    m_pTipLabel->setAlignment(Qt::AlignLeft | Qt::AlignVCenter);
    m_pTipLabel->setContentsMargins(10, 15, 10, 15);
    m_pTipLabel->setObjectName(tr("dialogTipLabel"));
    
    m_pTitileBar = new TitleBar(this);
    m_pTitileBar->setContextMenuPolicy(Qt::NoContextMenu);
    
    m_pTitileBar->setVisibleMinBtn(false);
    m_pTitileBar->setVisibleMaxBtn(false);
    m_pTitileBar->setVisibleResumeBtn(false);
    m_pTitileBar->setVisibleColseBtn(false);
    m_pTitileBar->setEnabledDoubleClicked(false);
    
    m_pTitileBar->setObjectName(tr("dialogTitle"));
    
    m_pAcceptBtn = new QPushButton(zhFixAct);
    m_pAcceptBtn->setObjectName(tr("dialogAcceptBtn"));
    m_pRejectBtn = new QPushButton(zhRefuseAct);
    m_pRejectBtn->setObjectName(tr("dialogRejectBtn"));
    m_pCancelBtn = new QPushButton(zhCancelAct);
    m_pCancelBtn->setObjectName(tr("dialogCancelBtn"));
    
    
    m_pBottomLeftSpacer = new QLabel();
    m_pBottomCenterSpacer = new QLabel();
    
    LOAD_STYLESHEET_ARG(m_pBottomLeftSpacer, "dialog.qss");
    LOAD_STYLESHEET_ARG(m_pBottomCenterSpacer, "dialog.qss");
    
    m_pBottomLeftSpacer->setSizePolicy(QSizePolicy::Expanding, QSizePolicy::Minimum);
    m_pBottomCenterSpacer->setSizePolicy(QSizePolicy::Expanding, QSizePolicy::Minimum);
    m_pBottomCenterSpacer->setMaximumHeight(2);
    m_pBottomLeftSpacer->setMaximumHeight(2);
    m_VLayoutCenter->addWidget(m_pTipLabel);
    m_pCenterWidget->setLayout(m_VLayoutCenter);
    m_VLayoutCenter->setContentsMargins(0, 0, 0, 0);
    m_pCenterWidget->setContentsMargins(0, 0, 0, 0);
    
    m_HLayoutBottom->addWidget(m_pBottomLeftSpacer);
    m_HLayoutBottom->addWidget(m_pBottomCenterSpacer);
    m_HLayoutBottom->addWidget(m_pAcceptBtn);
    m_HLayoutBottom->addWidget(m_pRejectBtn);
    m_HLayoutBottom->addWidget(m_pCancelBtn);
    #ifdef Q_OS_WIN
    m_HLayoutBottom->setSpacing(10);
    m_HLayoutBottom->setContentsMargins(28, 8, 20, 8);
    #else
    m_HLayoutBottom->setSpacing(20);
    m_HLayoutBottom->setContentsMargins(28, 14, 20, 14);
    #endif
    m_pBottomWidget->setLayout(m_HLayoutBottom);
    m_pBottomWidget->setMaximumHeight(46);
    m_VLayoutMain->addWidget(m_pTitileBar);
    m_pTitileBar->show();
    m_VLayoutMain->addWidget(m_pCenterWidget);
    m_VLayoutMain->addWidget(m_pBottomWidget);
    m_VLayoutMain->setContentsMargins(0, 0, 0, 0);
    m_VLayoutMain->setSpacing(0);
    setLayout(m_VLayoutMain);
    m_pTitileBar->setMinimumHeight(35);
    m_pTitileBar->setMaximumHeight(35);
    this->setObjectName("dialogWidget");
    setMinimumSize(320, 170);
}

void DialogWidget::setRejctVisible(bool visible)
{
    if (!visible)
    {
        m_pRejectBtn->hide();
        m_pBottomCenterSpacer->hide();
        m_pBottomLeftSpacer->show();
    }
    else
    {
        m_pRejectBtn->show();
        m_pBottomCenterSpacer->show();
        m_pBottomLeftSpacer->hide();
    }
}


void DialogWidget::setCancelVisible(bool visible)
{
    if (!visible)
    {
        m_pCancelBtn->hide();
        m_pBottomLeftSpacer->show();
    }
    else
    {
        m_pCancelBtn->show();
        m_pBottomLeftSpacer->hide();
    }
}

void DialogWidget::setAccepVisible(bool visible)
{
    m_pAcceptBtn->setVisible(visible);
}

void DialogWidget::translateLanguage()
{
    m_pAcceptBtn->setText(zhFixAct);
    m_pRejectBtn->setText(zhRefuseAct);
    m_pCancelBtn->setText(zhCancelAct);
}

void DialogWidget::changeEvent(QEvent* e)
{
    QWidget::changeEvent(e);
    switch (e->type()) {
        case QEvent::LanguageChange:
            translateLanguage();
            break;
        default:
            break;
    }
}

void DialogWidget::slotAccept()
{
    emit signalAccept();
    m_currentCode = enAccept;
    if (m_currentType != enCritical)
    {
        close();
    }
}

void DialogWidget::slotReject()
{
    emit signalReject();
    m_currentCode = enReject;
    if (m_currentType != enCritical)
    {
        close();
    }   
}

void DialogWidget::slotCancel()
{
    emit signalCancel();
    m_currentCode = enCancel;
    if (m_currentType != enCritical)
    {
        close();
    }
}

void DialogWidget::setDialotType(enDialogType type)
{
    m_currentType = type;
}

void DialogWidget::setTitle(const QString& title)
{
    m_pTitileBar->setTitleText(title);
}

void DialogWidget::showClose()
{
    m_pTitileBar->setVisibleColseBtn(true);
}

void DialogWidget::setAcceptText(const QString& text)
{
    m_pAcceptBtn->setText(text);
}

void DialogWidget::setRejectText(const QString& text)
{
    m_pRejectBtn->setText(text);
}

void DialogWidget::setCancelText(const QString& text)
{
    m_pCancelBtn->setText(text);
}

void DialogWidget::setCenterWidgetMargins(int left, int top, int right, int bottom)
{
    m_VLayoutCenter->setContentsMargins(left, top, right, bottom);
}

void DialogWidget::setStandardButtons(int btnType)
{
    if (btnType == DialogWidget::AcceptBtn)
    {
        setCancelVisible(false);
        setRejctVisible(false);
    }
    else if (btnType == DialogWidget::RejectBtn)
    {
        setAccepVisible(false);
        setCancelVisible(false);
    }
    else if (btnType == DialogWidget::CancelBtn)
    {
        setAccepVisible(false);
        setRejctVisible(false);
    }
    else if (btnType == (DialogWidget::AcceptBtn | DialogWidget::RejectBtn))
    {
        setCancelVisible(false);
    }
    else if (btnType == (DialogWidget::AcceptBtn | DialogWidget::CancelBtn))
    {
        setRejctVisible(false);
    }
    else if (btnType == (DialogWidget::RejectBtn | DialogWidget::CancelBtn))
    {
        setAccepVisible(false);
    }
    else if (btnType == (DialogWidget::AcceptBtn | DialogWidget::RejectBtn | DialogWidget::CancelBtn))
    {
        
    }
    else if (btnType == DialogWidget::NoBtn)
    {
        setAccepVisible(false);
        setRejctVisible(false);
        setCancelVisible(false);
    }
}

void DialogWidget::setInfor(const QString& text, Qt::Alignment alignment)
{
    //QString disPlayText = m_pTipLabel->fontMetrics().elidedText(text, Qt::ElideRight, m_pTipLabel->width());
    m_pTipLabel->setText(text);
    m_pTipLabel->setAlignment(alignment);
}

/**
* @brief 返回一个按钮，如果参数是多个按钮的组合则返回AcceptButton 
*/
QPushButton* DialogWidget::standardButton(int btnType)
{
    if (btnType == DialogWidget::AcceptBtn)
    {
        return m_pAcceptBtn;
    }
    else if (btnType == DialogWidget::RejectBtn)
    {
        return m_pRejectBtn;
    }
    else if (btnType == DialogWidget::CancelBtn)
    {
        return m_pCancelBtn;
    }
    return m_pAcceptBtn;
}

void DialogWidget::setCenterWidget(QWidget* w)
{
    m_pTipLabel->hide();
    m_VLayoutCenter->addWidget(w);
    setCenterWidgetMargins(1,0,1,0);
}

DialogWidget::enDialogCode DialogWidget::run()
{
    QDialog::exec();
    m_VLayoutCenter->removeWidget(m_VLayoutCenter->widget());
    return m_currentCode;
}
```



```cpp
#pragma once
/*!
* @file titlebar.h
* @date 2019/04/24 9:10
*
* @author gengwh
* Contact: gengwh@novastar.tech
*
* @brief 主窗口的标题栏
*/
#include <QWidget>
#include <QLabel>
#include <QPushButton>
#include <QMenuBar>
#include <QTimer>
#include <QHBoxLayout>

enum ButtonType
{
MIN_BUTTON = 0,         // 最小化和关闭按钮;
MIN_MAX_BUTTON,        // 最小化、最大化和关闭按钮;
MIN_RESTOR_BUTTON,        // 最小化、恢复和关闭按钮;
ONLY_CLOSE_BUTTON       // 只有关闭按钮;

};

const int EDGET_WIDTH = 3;   //边框的宽度，在边框上拖动时不移动
#ifdef Q_OS_WIN
class TitleBar : public QMenuBar
#else
class TitleBar : public QWidget
#endif
{
Q_OBJECT

public:
TitleBar(QWidget *parent);
//这里parent没有给默认值NULL，保证在创建MyTitleBar对象时父指针必须得赋值;且赋值不为NULL;
~TitleBar();

void setTitleText(QString titleContent, int titleFontSize = 9);

void setButtonType(ButtonType buttonType);
ButtonType getButtonType();
void addMenu(int index, QWidget *menu);
void addStretch();

// 保存/获取 最大化前窗口的位置及大小;
void saveRestoreInfo(const QPoint point, const QSize size);
void getRestoreInfo(QPoint& point, QSize& size);
void setVisibleMinBtn(bool visible);
void setVisibleMaxBtn(bool visible);
void setVisibleResumeBtn(bool visible);
void setVisibleColseBtn(bool visible);
void setEnabledDoubleClicked(bool enable);

void installMouseMoveFilter(QWidget *fifter);
void translateLanguage();

protected:
void paintEvent(QPaintEvent *event);
void changeEvent(QEvent* e);
private:
void mouseDoubleClickEvent(QMouseEvent *event);
void mousePressEvent(QMouseEvent *event);
void mouseMoveEvent(QMouseEvent *event);
void mouseReleaseEvent(QMouseEvent *event);

// 初始化控件;
void initControl();
// 信号槽的绑定;
void initConnections();

signals:
// 按钮触发的信号;
void signalButtonMinClicked();
void signalButtonRestoreClicked();
void signalButtonMaxClicked();
void signalButtonCloseClicked();
void signalMove(int x, int y);
void signalShowNormal(); //最大化拖动窗口时，窗口的大小要改变为上次最大化之前的大小，但是位置是当前鼠标的位置

private slots:
// 按钮触发的槽;
void onButtonMinClicked();
void onButtonRestoreClicked();
void onButtonMaxClicked();
void onButtonCloseClicked();

private:
QLabel* m_pTitleLabel;              // 标题栏内容;
QPushButton* m_pButtonMin;          // 最小化按钮;
QPushButton* m_pButtonRestore;      // 最大化还原按钮;
QPushButton* m_pButtonMax;          // 最大化按钮;
QPushButton* m_pButtonClose;        // 关闭按钮;
QHBoxLayout* m_pLayout;

// 最大化，最小化变量;
QPoint m_restorePos;
QSize m_restoreSize;
// 移动窗口的变量;
bool m_isPressed;
bool m_isDoulePressed;
QPoint m_startMovePos;
// 标题栏内容;
QString m_titleContent;
// 按钮类型;
ButtonType m_buttonType;
bool m_bRestoreFlag = false;
bool m_bMaxFlag = false;

// 标题栏是否透明;
bool m_isTransparent;
bool m_isForbidMove;
bool m_bEnableDoubleClicked;
};
```



```cpp
/*!
* @file titlebar.cpp
* @date 2019/04/24 9:12
*
* @author gengwh
* Contact: gengwh@novastar.tech
*
* @brief TitleBar具体实现
*/
#define BUTTON_HEIGHT 15       // 按钮高度;
#define BUTTON_WIDTH 15         // 按钮宽度;
#define TITLE_HEIGHT 30         // 标题栏高度;

#include <QPainter>
#include <QFile>
#include <QMouseEvent>
#include <QStylePainter>
#include <QStyleOption>
#include <QDebug>
#include "titlebar.h"
#include "macrofunc.h"
#include "../language.h"
#pragma execution_character_set("utf-8")
#ifdef Q_OS_WIN
TitleBar::TitleBar(QWidget *parent /*= nullptr*/)
: QMenuBar(parent)
, m_isPressed(false)
, m_buttonType(MIN_RESTOR_BUTTON)
, m_isTransparent(false)
#else
TitleBar::TitleBar(QWidget *parent /*= nullptr*/)
: QWidget(parent)
, m_isPressed(false)
, m_buttonType(MIN_RESTOR_BUTTON)
, m_isTransparent(false)
#endif
{
    // 初始化;
    //setObjectName("mainTitleBar");
    initControl();
    initConnections();
    setMouseTracking(true);
    setButtonType(m_buttonType);
    m_bEnableDoubleClicked = true;
}

TitleBar::~TitleBar()
{

}

void TitleBar::setTitleText(QString titleContent, int titleFontSize /*= 9*/)
{
    m_titleContent = titleContent;
    m_pTitleLabel->setText(titleContent);
    m_pTitleLabel->show();

}

void TitleBar::setButtonType(ButtonType buttonType)
{
    m_buttonType = buttonType;

    switch (buttonType)
        {
            case MIN_BUTTON:
                {
                    m_pButtonRestore->setVisible(false);
                    m_pButtonMax->setVisible(false);
                }
                break;
            case MIN_MAX_BUTTON:
                {
                    m_pButtonRestore->setVisible(false);
                    m_pButtonMax->setVisible(true);
                }
                break;
            case MIN_RESTOR_BUTTON:
                {
                    if (m_bRestoreFlag)
                    {
                        m_pButtonRestore->setVisible(false);
                        break;
                    }
                    m_pButtonMax->setVisible(false);
                    m_pButtonRestore->setVisible(true);
                }
                break;
            case ONLY_CLOSE_BUTTON:
                {
                    m_pButtonMin->setVisible(false);
                    m_pButtonRestore->setVisible(false);
                    m_pButtonMax->setVisible(false);
                }
                break;
            default:
                break;
        }
}

ButtonType TitleBar::getButtonType()
{
    return m_buttonType;
}

void TitleBar::addMenu(int index, QWidget *menu)
{
    m_pLayout->insertWidget(index, menu);
}

void TitleBar::addStretch()
{
    m_pLayout->addStretch();
}

void TitleBar::saveRestoreInfo(const QPoint point, const QSize size)
{
    m_restorePos = point;
    m_restoreSize = size;
}

void TitleBar::getRestoreInfo(QPoint& point, QSize& size)
{
    point = m_restorePos;
    size = m_restoreSize;
}

void TitleBar::setVisibleMinBtn(bool visible)
{
    m_pButtonMin->setVisible(visible);
}

void TitleBar::setVisibleMaxBtn(bool visible)
{
    m_pButtonMax->setVisible(visible);
    m_bMaxFlag = !visible;
}

void TitleBar::setVisibleResumeBtn(bool visible)
{
    m_pButtonRestore->setVisible(visible);
    m_bRestoreFlag = !visible;
}

void TitleBar::setVisibleColseBtn(bool visible)
{
    m_pButtonClose->setVisible(visible);
}

void TitleBar::setEnabledDoubleClicked(bool enable)
{
m_bEnableDoubleClicked = enable;
}

void TitleBar::installMouseMoveFilter(QWidget *fifter)
{
m_pTitleLabel->setMouseTracking(true);
m_pButtonMin->setMouseTracking(true);
m_pButtonRestore->setMouseTracking(true);
m_pButtonMax->setMouseTracking(true);
m_pButtonClose->setMouseTracking(true);

m_pTitleLabel->installEventFilter(fifter);
m_pButtonMin->installEventFilter(fifter);
m_pButtonRestore->installEventFilter(fifter);
m_pButtonMax->installEventFilter(fifter);
m_pButtonClose->installEventFilter(fifter);
}
void TitleBar::mouseDoubleClickEvent(QMouseEvent *event)
{
if (Qt::RightButton == event->button() || !m_bEnableDoubleClicked)
{
return;
}
// 只有存在最大化、还原按钮时双击才有效;
if (m_buttonType == MIN_MAX_BUTTON)
{
onButtonMaxClicked();
m_buttonType = MIN_RESTOR_BUTTON;
}
else if (m_buttonType == MIN_RESTOR_BUTTON)
{
onButtonRestoreClicked();
}
m_isDoulePressed = true;

return QWidget::mouseDoubleClickEvent(event);
}

void TitleBar::mousePressEvent(QMouseEvent *event)
{
if (Qt::RightButton == event->button())
{
return;
}
// 判断鼠标是否处于边界区域
bool isLeftEdge = event->pos().x() < EDGET_WIDTH;
bool isTopEdge = event->pos().y() < EDGET_WIDTH;
bool isRightEdge = event->pos().x() > width() - EDGET_WIDTH;
if (!isLeftEdge && !isTopEdge && !isRightEdge)
{
m_isPressed = true;
m_startMovePos = event->globalPos();
}
return QWidget::mousePressEvent(event);
}

void TitleBar::mouseMoveEvent(QMouseEvent *event)
{
if (Qt::RightButton == event->button())
{
return;
}
if (m_isPressed && !m_isDoulePressed)
{
QPoint movePoint = event->globalPos() - m_startMovePos; //移动的距离
if (abs(movePoint.x()) <= 15 && abs(movePoint.y()) <= 15)
{
return;
}
QPoint widgetPos = this->parentWidget()->pos();

int x = widgetPos.x() + movePoint.x();
int y = widgetPos.y() + movePoint.y();

if (m_buttonType == MIN_RESTOR_BUTTON)
{
if (!m_bRestoreFlag)
{
setButtonType(MIN_MAX_BUTTON);
int beforeWidth = this->parentWidget()->width();
emit signalShowNormal();
int afterWidth = this->parentWidget()->width();

x = m_startMovePos.x() - 1.0 * afterWidth / beforeWidth * (m_startMovePos.x() - widgetPos.x());
}
}
m_startMovePos = event->globalPos();
this->parentWidget()->move(x, y);

emit signalMove(x, y);

}
return QWidget::mouseMoveEvent(event);
}

void TitleBar::mouseReleaseEvent(QMouseEvent *event)
{
m_isPressed = false;
m_isDoulePressed = false;
return QWidget::mouseReleaseEvent(event);
}

void TitleBar::initControl()
{
m_pButtonMin = new QPushButton;
m_pButtonRestore = new QPushButton;
m_pButtonMax = new QPushButton;
m_pButtonClose = new QPushButton;
m_pTitleLabel = new QLabel();
m_pTitleLabel->setContextMenuPolicy(Qt::NoContextMenu);
m_pTitleLabel->setSizePolicy(QSizePolicy::Expanding, QSizePolicy::Fixed);
m_pTitleLabel->setAlignment(Qt::AlignCenter);
LOAD_STYLESHEET_ARG(m_pTitleLabel, "titlebar.qss");

#ifdef Q_OS_WIN
m_pTitleLabel->hide();
#else
m_pTitleLabel->show();
#endif



//     m_pIcon->setObjectName("TitleIcon");
//     m_pTitleContent->setObjectName("TitleContent");
m_pButtonMin->setObjectName("ButtonMin");
m_pButtonRestore->setObjectName("ButtonRestore");
m_pButtonMax->setObjectName("ButtonMax");
m_pButtonClose->setObjectName("ButtonClose");
m_pTitleLabel->setObjectName("TitleLabel");
m_pButtonMin->setToolTip(zhMinAct);
m_pButtonRestore->setToolTip(zhReductionAct);
m_pButtonMax->setToolTip(zhMaxAct);
m_pButtonClose->setToolTip(zhCloseAct);

m_pLayout = new QHBoxLayout(this);

m_pLayout->addWidget(m_pTitleLabel);
m_pLayout->addWidget(m_pButtonMin);
m_pLayout->addWidget(m_pButtonRestore);
m_pLayout->addWidget(m_pButtonMax);
m_pLayout->addWidget(m_pButtonClose);

m_pLayout->setContentsMargins(10, 0, 8, 0);
m_pLayout->setSpacing(16);
//m_pLayout->setAlignment(Qt::AlignCenter);
#ifdef Q_OS_WIN
setFixedHeight(TITLE_HEIGHT);
setWindowFlags(Qt::FramelessWindowHint);
#endif
}

void TitleBar::initConnections()
{
connect(m_pButtonMin, SIGNAL(clicked()), this, SLOT(onButtonMinClicked()));
connect(m_pButtonRestore, SIGNAL(clicked()), this, SLOT(onButtonRestoreClicked()));
connect(m_pButtonMax, SIGNAL(clicked()), this, SLOT(onButtonMaxClicked()));
connect(m_pButtonClose, SIGNAL(clicked()), this, SLOT(onButtonCloseClicked()));
}

void TitleBar::onButtonMinClicked()
{
emit signalButtonMinClicked();
}

void TitleBar::onButtonRestoreClicked()
{
setButtonType(MIN_MAX_BUTTON);
emit signalButtonRestoreClicked();
}

void TitleBar::onButtonMaxClicked()
{
setButtonType(MIN_RESTOR_BUTTON);
emit signalButtonMaxClicked();
}

void TitleBar::onButtonCloseClicked()
{
emit signalButtonCloseClicked();
}

void TitleBar::paintEvent(QPaintEvent *event)
{
QStylePainter painter(this);
//用style画背景 (会使用setstylesheet中的内容)

QStyleOption opt;
opt.initFrom(this);
opt.rect = rect();
painter.drawPrimitive(QStyle::PE_Widget, opt);
QWidget::paintEvent(event);
}



void TitleBar::changeEvent(QEvent* e)
{
QWidget::changeEvent(e);
switch (e->type()) {
case QEvent::LanguageChange:
translateLanguage();
break;
default:
break;
}
}

void TitleBar::translateLanguage()
{
if(m_pButtonMin)m_pButtonMin->setToolTip(zhMinAct);
if (m_pButtonRestore)m_pButtonRestore->setToolTip(zhReductionAct);
if (m_pButtonMax)m_pButtonMax->setToolTip(zhMaxAct);
if (m_pButtonClose)m_pButtonClose->setToolTip(zhCloseAct);
}
```

