---
id: qt-multithreading
title: "Qt 多线程的几种用法"
date: 2026-07-18
tags: Qt,多线程
excerpt: "Qt 中四种多线程实现方式：继承 QThread 重写 run()、moveToThread、QThreadPool + QRunnable、QtConcurrent。"
cover: linear-gradient(135deg, #a8edea 0%, #fed6e3 100%)
---

# <font style="color:rgb(37, 41, 51);">0、概述</font>
<font style="color:rgb(37, 41, 51);">Qt中目前有4种实现多线程的方式。如下：</font>

<font style="color:rgb(37, 41, 51);">（1）继承QThread，重写run()函数</font>

<font style="color:rgb(37, 41, 51);">（2）使用moveToThread将一个继承QObject的子类移至线程，内部槽函数均在线程中执行</font>

<font style="color:rgb(37, 41, 51);">（3）使用QThreadPool,搭配QRunnable（线程池）</font>

<font style="color:rgb(37, 41, 51);">（4）使用QtConcurrent（线程池）</font>

# <font style="color:rgb(37, 41, 51);">1、使用类QThread</font>
<font style="color:rgb(37, 41, 51);">代码完全在一个独立的线程中运行，需要继承QThread类，并且重写该类的run()方法。实现方式比较简单，线程在执行完run()函数之后退出。</font>

```cpp
class WorkerThread : public QThread
{
    Q_OBJECT
protected:
    void run() Q_DECL_OVERRIDE {
        QString result;
        /* ... here is the expensive or blocking operation ... */
        emit resultReady(result);
    }

private:
signals:
    void resultReady(const QString &s);
};

void MyObject::startWorkInAThread()
{
    WorkerThread *workerThread = new WorkerThread(this);
    connect(workerThread, &WorkerThread::resultReady, this, &MyObject::handleResults);
    connect(workerThread, &WorkerThread::finished, workerThread, &QObject::deleteLater);
    workerThread->start();
}
```

**<font style="color:rgb(37, 41, 51);">优点：</font>**

+ <font style="color:rgb(37, 41, 51);">可以使用信号槽进行通信</font>

**<font style="color:rgb(37, 41, 51);">缺点：</font>**

+ <font style="color:rgb(37, 41, 51);">实例化的子类是在创建线程的旧线程中，不是在新创建的子线程中，因此，与该线程相关所有槽队列（如果有的话），都会在创建的旧线程中执行。不能直接在新建的线程中使用槽，如果需要，则需要借助worker - object实现。</font>
+ <font style="color:rgb(37, 41, 51);">每创建一个线程都需要继承QThread，实现一个新的子类，使用不便。</font>
+ <font style="color:rgb(37, 41, 51);">需要自己管理资源，线程的创建和释放，都需要自己手动管理，并且，频繁的创建和删除会造成比较大的内存开销。</font>
+ <font style="color:rgb(37, 41, 51);">实例化子类的构造函数和run()函数在不同的线程中运行，因此，假设有成员变量两个函数中都能访问，则需要注意，多线程中资源的访问问题。</font>

**<font style="color:rgb(37, 41, 51);">适用场景：</font>**

+ <font style="color:rgb(37, 41, 51);">线程不会被频繁的创建和删除，常驻内存的线程。</font>

# <font style="color:rgb(37, 41, 51);">2、QThreadPool与QRunnable</font>
<font style="color:rgb(37, 41, 51);">QRunnable 是所有可执行对象的基类。 QRunnable类是一个接口, 用于表示需要执行的任务或代码段, 具体任务在run() 函数内部实现。实现的过程跟QThread类似，也是需要通过继承来实现。</font>

<font style="color:rgb(37, 41, 51);">可以通过搭配QThreadPool，使得其在单独的线程中执行代码。如果autoDelete()返回true(默认值)，QThreadPool将自动删除QRunnable。使用setAutoDelete()来更改自动删除标记。</font>

<font style="color:rgb(37, 41, 51);">QThreadPool支持通过在run()函数中调用QThreadPool::tryStart(this)来多次执行同一个QRunnable。如果autoDelete被启用，QRunnable将在最后一个线程退出run函数时被删除。</font>

<font style="color:rgb(37, 41, 51);">如果启用了autoDelete，使用相同的QRunnable多次调用QThreadPool::start()会造成多线程访问同一资源，形成竞争，因此不推荐使用。</font>

```cpp
class Runnable : public QRunnable
{
    //Q_OBJECT		QRunnable 不是QObject的子类，因此在这儿需要注意
public:
    ~Runnable()
    {
        qDebug() << "~Runnable..."  << endl;
    }
    void run()
    {
        qDebug() << " Runnable run thread id :" << QThread::currentThreadId() << endl;
        //...
    }
};
void MyObject::startWorkInRunable()
{
    qDebug() << "main thread id :" << QThread::currentThreadId() << endl;
    Runnable run;
    QThreadPool::globalInstance()->start(&run);
}
```

**<font style="color:rgb(37, 41, 51);">优点：</font>**

+ <font style="color:rgb(37, 41, 51);">不用资源管理，QThreadPool 启动线程执行完成后会自动释放</font>

**<font style="color:rgb(37, 41, 51);">缺点：</font>**

+ <font style="color:rgb(37, 41, 51);">可能会形成多线程资源竞争</font>
+ <font style="color:#DF2A3F;">不能使用信号槽（信号槽只能在QObject中使用）</font>

**<font style="color:rgb(37, 41, 51);">适用场景：</font>**

+ <font style="color:rgb(37, 41, 51);">QRunnable适用于线程任务量比较大，需要频繁创建线程。QRunnable能有效减少内存开销。</font>

# <font style="color:rgb(37, 41, 51);">3、使用QThread类moveToThread方法</font>
<font style="color:rgb(37, 41, 51);">创建一个继承QObject的类（MyObject）,并把创建的MyObject类通过方法 movetothread 到创建好的子线程中，然后start子线程，这样就实现了一个子线程。主线程通过发送信号，调用 MyObject 中的方法，从而实现在子线程中的计算。不需要继承QThread，不需要重写run()函数。</font>

```cpp
class Worker : public QObject
{
    Q_OBJECT

public slots:
    void doWork(const QString &parameter) {
        QString result;
        /* ... here is the expensive or blocking operation ... */
        emit resultReady(result);
    }

signals:
    void resultReady(const QString &result);
};

class Controller : public QObject
{
    Q_OBJECT
    QThread workerThread;
public:
    Controller() {
        Worker *worker = new Worker;
        worker->moveToThread(&workerThread);
        connect(&workerThread, &QThread::finished, worker, &QObject::deleteLater);
        connect(this, &Controller::operate, worker, &Worker::doWork);
        connect(worker, &Worker::resultReady, this, &Controller::handleResults);
        workerThread.start();
    }
    ~Controller() {
        workerThread.quit();
        workerThread.wait();
    }
public slots:
    void handleResults(const QString &);
signals:
    void operate(const QString &);
};
```

**<font style="color:rgb(37, 41, 51);">需要注意的点：</font>**

+ <font style="color:rgb(37, 41, 51);">只有在槽中执行的操作才是在线程中执行的，所以需要通过连接信号槽的方式来实现</font>
+ <font style="color:rgb(37, 41, 51);">如果object对象存在父对象，不能将其移到子线程中执行</font>
+ <font style="color:rgb(37, 41, 51);">相对来说比较方便，适用于一些比较复杂的业务中</font>

# <font style="color:rgb(37, 41, 51);">4、使用QtConcurrent</font>
<font style="color:rgb(37, 41, 51);">Concurrent是并发的意思，QtConcurrent是一个命名空间，提供了一些高级的 API，使得在编写多线程的时候，无需使用低级线程原语，如读写锁，等待条件或信号。使用QtConcurrent编写的程序会根据可用的处理器内核数自动调整使用的线程数。这意味着今后编写的应用程序将在未来部署在多核系统上时继续扩展。</font>

<font style="color:rgb(37, 41, 51);">QtConcurrent::run能够方便快捷的将任务丢到子线程中去执行，无需继承任何类，也不需要重写函数，使用非常简单。</font>

<font style="color:rgb(37, 41, 51);">调用非常简单，这样我们设置的函数会在一个单独的线程中执行，这个线程是从QThreadPool池中获取的，因此，</font>**<font style="color:rgb(37, 41, 51);">该线程可能不会直接执行（如果QThreadPool池中没有空闲的线程），当QThreadPool池中有空闲的线程后会执行</font>**<font style="color:rgb(37, 41, 51);">，执行完成后会将线程还给QThreadPool。</font>

<font style="color:rgb(37, 41, 51);">QtConcurrent::run()返回的QFuture不支持取消、暂停或进度报告。返回的QFuture只能用于查询函数的运行/完成状态和返回值。非void类型返回值可使用QFuture::result()访问获取。</font>

