---
id: qt-signal-slot-principle
title: "Qt 信号槽原理"
date: 2026-07-18
tags: Qt,信号槽
excerpt: "Qt 信号与槽的实现基于元对象系统（MOC），预处理阶段将信号和槽的字符串值保存到容器中，运行时通过查找匹配完成调用。"
cover: linear-gradient(135deg, #f093fb 0%, #f5576c 100%)
---

<font style="color:rgb(77, 77, 77);">信号与槽的实现是借助了Qt 的元对象系统，元对象系统有一个元对象编译器，程序编译之前会有一个预处理过程，预处理将一个类/对象中的信号，槽的字符串值分别保存在一个容器中，可能是字符串或者其他的有序容器。</font>

<font style="color:rgb(77, 77, 77);"></font>

```cpp
#define Q_OBJECT \
public: \
    QT_WARNING_PUSH \
    Q_OBJECT_NO_OVERRIDE_WARNING \
    static const QMetaObject staticMetaObject; \
    virtual const QMetaObject *metaObject() const; \
    virtual void *qt_metacast(const char *); \
    virtual int qt_metacall(QMetaObject::Call, int, void **); \
    QT_TR_FUNCTIONS \
private: \
    Q_OBJECT_NO_ATTRIBUTES_WARNING \
    Q_DECL_HIDDEN_STATIC_METACALL static void qt_static_metacall(QObject *, QMetaObject::Call, int, void **); \
    QT_WARNING_POP \
    struct QPrivateSignal {}; \
    QT_ANNOTATE_CLASS(qt_qobject, "")



QMetaObject::Connection connect(sender,signal,receiver,slot,Qt::ConnectionType type = Qt::AutoConnection)
{
        typedef QtPrivate::FunctionPointer<Func1> SignalType;
        typedef QtPrivate::FunctionPointer<Func2> SlotType;

        const int *types = nullptr;
        if (type == Qt::QueuedConnection || type == Qt::BlockingQueuedConnection)
            types = QtPrivate::ConnectionTypes<typename SignalType::Arguments>::types();

        return connectImpl(sender, signal,receiver,slot,
    					new QtPrivate::QSlotObject<Func2, typename QtPrivate::List_Left<typename SignalType::Arguments, SlotType::ArgumentCount>::Value,						typename SignalType::ReturnType>(slot),type, types, &SignalType::Object::staticMetaObject);
}


QMetaObject::Connection QObject::connectImpl(sender,signal,receiver, slot,slotObj, Qt::ConnectionType type,types, const QMetaObject *senderMetaObject)
{
    if (!signal) {
        qCWarning(lcConnect, "QObject::connect: invalid nullptr parameter");
        if (slotObj)
            slotObj->destroyIfLastRef();
        return QMetaObject::Connection();
    }

    int signal_index = -1;
    void *args[] = { &signal_index, signal };
    for (; senderMetaObject && signal_index < 0; senderMetaObject = senderMetaObject->superClass()) {
        senderMetaObject->static_metacall(QMetaObject::IndexOfMethod, 0, args);
        if (signal_index >= 0 && signal_index < QMetaObjectPrivate::get(senderMetaObject)->signalCount)
            break;
    }
    if (!senderMetaObject) {
        qCWarning(lcConnect, "QObject::connect: signal not found in %s", sender->metaObject()->className());
        slotObj->destroyIfLastRef();
        return QMetaObject::Connection(nullptr);
    }
    signal_index += QMetaObjectPrivate::signalOffset(senderMetaObject);
    return QObjectPrivate::connectImpl(sender, signal_index, receiver, slot, slotObj, type, types, senderMetaObject);
}
```





<!-- 这是一张图片，ocr 内容为：QT信号槽源码剖析 ITWENYINA 微信公众号 信号 当对象改变其状态时,信号就由该对象发射(EMIT)出去,而且对象只负责发送信号,它不 知道另一端是谁在接收这个信号. 槽 用于接收信号,而且槽只是普通的对象成员函数.一个槽并不知道是否有任何信号与自己 相连接. -->
![](https://cdn.nlark.com/yuque/0/2023/png/1443385/1678101602957-df3331b5-a832-4e4e-9676-2c5b09918e97.png)

<!-- 这是一张图片，ocr 内容为：QT信号槽源码剖析 微信公众号 ITWENYINAN MOC,THE META OBJECT COMPILER.Q程序在交由标准编译器(例如MSVC)编译之前,先 使用MOC分析CPP头文件;如果它发现在一个头文件中包含了Q.OBJECT宏,则会生成另外 一个CPP源文件(MOC.文件名.CPP),该CPP源文件中包含了Q.OBJECT宏的实现,运行时信 息(反射)等.因此QT程序的完整编译过程为MOC->预处理->编译->链接 -->
![](https://cdn.nlark.com/yuque/0/2023/png/1443385/1678101729503-9fda71bc-cfcb-4045-a292-9e421e006cd5.png)



<!-- 这是一张图片，ocr 内容为：SIGNALS 代码演示推导过程 #DEFINE SIGNALS PUBLIC -->
![](https://cdn.nlark.com/yuque/0/2023/png/1443385/1678102152309-7d138ed3-8266-4d3a-8584-8967e4d3a4d8.png)



<!-- 这是一张图片，ocr 内容为：QT信号槽源码剖析 微信公众号 ITWENYINAN OBJECT 代码演示推导过程 #DEFINEQOBJECT\ PUBLIC:\ STATIC CONST OMETAOBJECT STATICMETAOBJECT;\ VIRTUAL CONST QMETAOBJECT*METAOBJECT( CONST; VIRTUAL VOID*AT METACAST(CONST CHAR*);\ VIRTUAL INT AT METACALL(QMETAOBIECT:CALL,INT,VOID **); QT TR FUNCTIONS\ PRIVATE:// @,DECL.HIDDEN,STATIC.METACALL STATIC VOID GT STATIC.METACALL(QOBJECT', QMETAOBJECT:CALL,INT, VOID ** -->
![](https://cdn.nlark.com/yuque/0/2023/png/1443385/1678102306315-59efebbb-9cca-4c50-9b0f-021364119349.png)

+ <font style="color:rgba(0, 0, 0, 0.75);">绑定信号、槽函数</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">调用信号函数（将信号写入队列）</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">主线程从队列中获取信号</font>

<font style="color:rgba(0, 0, 0, 0.75);"></font>

<font style="color:rgb(77, 77, 77);"></font>



