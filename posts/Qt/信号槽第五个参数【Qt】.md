---
id: qt-signal-slot-connection-type
title: "Qt 信号槽第五个参数（连接类型）"
date: 2026-07-18
tags: Qt,信号槽
excerpt: "Qt 信号槽连接类型详解：AutoConnection、DirectConnection、QueuedConnection、BlockingQueuedConnection、UniqueConnection。"
cover: linear-gradient(135deg, #ffecd2 0%, #fcb69f 100%)
---

# 1、Qt::AutoConnection
 	默认值，使用这个值则连接类型会在信号发送时决定。如果接收者和发送者在同一个线程，则自动使用Qt::DirectConnection类型。如果接收者和发送者不在一个线程，则自动使用Qt::QueuedConnection类型。

# 2、Qt::DirectConnection
槽函数会在信号发送的时候直接被调用，槽函数运行于信号发送者所在线程。效果看上去就像是直接在信号发送位置调用了槽函数。这个在多线程环境下比较危险，可能会造成奔溃。（相当于直接调用槽函数，但是当信号发出的线程和槽的对象不再一个线程的时候，则槽函数是在发出的信号中执行的。）

<font style="color:rgb(18, 18, 18);">假设当前有4个slot连接到QPushButton::clicked(bool)，当按钮被按下时，QT就把这4个slot按连接的时间顺序调用一遍。显然这种方式不能跨线程（传递消息）。</font>

# 3、Qt::QueuedConnection
槽函数在控制回到接收者所在线程的事件循环时被调用，槽函数运行于信号接收者所在线程。发送信号之后，槽函数不会立刻被调用，等到接收者的当前函数执行完，进入事件循环之后，槽函数才会被调用。多线程环境下一般用这个。（内部通过postEvent实现的。不是实时调用的，槽函数永远在槽函数对象所在的线程中执行。如果信号参数是引用类型，则会另外复制一份的。线程安全的。）

<font style="color:rgb(18, 18, 18);">假设当前有4个slot连接到QPushButton::clicked(bool)，当按钮被按下时，QT就把这个signal包装成一个 QEvent，放到消息队列里。QApplication::exec()或者线程的QThread::exec()会从消息队列里取消息，然后调用 signal关联的几个slot。这种方式既可以在线程内传递消息，也可以跨线程传递消息。</font>

# 4、Qt::BlockingQueuedConnection
槽函数的调用时机与Qt::QueuedConnection一致，不过发送完信号后发送者所在线程会阻塞，直到槽函数运行完。接收者和发送者绝对不能在一个线程，否则程序会死锁。在多线程间需要同步的场合可能需要这个。（此连接方式只能用于信号发出的线程（一般是先好对象的线程） 和 槽函数的对象不再一个线程中才能用。通过信号量+postEvent实现的。不是实时调用的，槽函数永远在槽函数对象所在的线程中执行。但是发出信号后，当前线程会阻塞，等待槽函数执行完毕后才继续执行。）

<font style="color:rgb(18, 18, 18);">与Qt::QueuedConnection类似，但是会阻塞等到关联的slot都被执行。这里出现了阻塞这个词，说明它是专门用来多线程间传递消息的。</font>

<font style="color:rgb(18, 18, 18);">很显然在不同的线程间发送信号还希望发送信号的一端必须阻塞性的等待槽函数返回</font>

<font style="color:rgb(0, 0, 0);">但是Qt::BlockingQueuedConnection在子线程不受你控制或者不在你自己的代码里的时候，容易发生死锁，比如一个主线程调用子线程的某方法，在这个方法里又发送了信号交给主线程处理事件，由于子线程一直等待主线程处理完事件返回继续往下，但主线程又卡在了调用子线程方法的地方，导致二者直接死锁，所以这个连接方式要慎用，至少你要明白引发这种连接信号一系列的调用关系再来确认怎么处理；另外还有一种情况需要block，那就是参数中有指针的情况，如果这个指针的指向对象是临时变量，当非阻塞方式交出去的时候，可能对象已经析构回收掉了，那么调用参数会访问一个野指针，所以要么拷贝一份指针数据到可靠的地方（至少确认执行的时候数据还在）,要么就得用阻塞方式，等待处理结束后，再释放指针指向对象。</font>

<font style="color:rgb(100, 100, 100);">使用Qt::BlockingQueuedConnection时必须小心，以避免死锁。您正在向同一个线程发送一个事件，然后锁定该线程，等待事件被处理。由于线程被阻塞，事件将永远不会被处理，线程将永远被阻塞，从而导致死锁。</font>

Qt::BlockingQueuedConnection：槽函数的调用时机与Qt::QueuedConnection一致，不过发送完信号后发送者所在线程会阻塞，直到槽函数运行完。接收者和发送者绝对不能在一个线程，否则程序会死锁。在多线程间需要同步的场合可能需要这个。

# 5、Qt::UniqueConnection
这个flag可以通过按位或（|）与以上四个结合在一起使用。当这个flag设置时，当某个信号和槽已经连接时，再进行重复的连接就会失败。也就是避免了重复连接。（防止重复连接。如果当前信号和槽已经连接过了，就不再连接了。）

# 6、Qt::SingleShotConnection（Qt 6新增）
<font style="color:rgb(18, 18, 18);">插槽只被调用一次，一旦发出信号，连接就被断开。</font>



