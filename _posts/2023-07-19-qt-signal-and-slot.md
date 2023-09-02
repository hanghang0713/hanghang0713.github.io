---
layout: post
title:  "Qt 中的信号与槽"
tags: C++ Qt
categories: Qt 
---

# 信号与槽的实现原理 {ignore=true}

[toc]

## 信号与槽

在 GUI 编程中，当更改一个地方时，我们通常希望通知另外一个地方。即对象之间能够相互通信。一般的 UI 框架往往都会使用回调的方式来实现。而在 Qt 中。则采用了另外一种方式，即信号与槽，它是 Qt 中的一个核心机制，基于 Qt 的 `meta-object system` 实现的。

![singnal and slot usage](/assets/images/signal_slot_usage.png "信号与槽使用示意")

- 信号与槽是松散耦合的，发出信号的类既不知道也不关心哪些槽接收信号
- 信号与槽是类型安全的，信号的签名必须与接收槽的签名相匹配。（槽的签名可能更短，因为可以忽略某些参数）


### signals 
当对象内部状态发生变化时，对象就会发出信号，信号是一个公共访问函数，可以从任何地方发出，但是建议从定义信号的类和它的子类中发出。
当发出信号时，与其连接的槽会立即被执行。此时，信号和槽机制完全独立于 GUI 事件循环。
当所有的槽执行完成后，`emit` 后面的代码会被执行（如果槽里面执行了耗时操作，会等待这些耗时操作完成。会造成阻塞）。
当使用 `queue connnection` 时，会先执行 `emit` 后面的代码，槽会在稍后执行。
如果多个槽连接到同一个信号，则按照连接的顺序依次执行。
信号由 `moc` 自动生成，不能在 `.cpp` 文件中实现。不能有返回类型。

### slots 
槽是一个普通的成员函数，可以正常调用，它的唯一特点是可以连接到信号。
不管槽的访问级别，它们可以由任何组件通过信号连接槽调用。从任意类的实例发出的信号可能会导致在不相关的类的实例中调用私有槽。
可以将槽定义为虚拟槽
信号和槽提供了更高的灵活性，因此比回调函数更慢，但是在实际应用程序的差异微不足道。



## meta-object

```C++ 
// 以一个最基础的 Counter 类为例
class Counter : public QObject
{
    Q_OBJECT

public:
    int value() const { return m_value; }
public slots:
    void setValue(int value);
signals:
    void valueChanged(int newValue);

private:
    int m_value;
};

```

Qt 的 `meta-object system` 为对象间通信提供了信号与槽机制，还实现了运行时类型信息，动态属性系统。
`meta-object system` 主要基于下面三件事情：
- `QObject` 可以为任何想要使用 `meta-object system` 的类提供基类
- 类的私有部分中的 `Q_OBJECT` 宏用于声明启用 `meta-object system`
- `Meta-Object compiler` (moc) 为每个子类提供实现  `meta-object system` 功能的代码


moc 工具会读取 `C++` 源文件，当找到有 `Q_OBJECT` 宏的类声明时，moc 会为这些类生成包含 `meta-object system` 代码的另一份 `C++` 文件。这些生成的文件，要么被包含到原来的源文件中，要么被编译和链接到原来的类的实现中（更通用的做法）。

MOC 为 Qt 提供了运行时类型检查，这意味着能够列出一个对象的属性和方法，还有他们相关的参数以及其他信息。（`C++` 本身并不提供这个支持）。因此，MOC 是一个代码生成器而不是预处理器（MOC 生成的文件在输出目录中）。

> we strongly recommend that all subclasses of QObject use the Q_OBJECT macro regardless of whether or not they actually use signals, slots, and properties.


## Magic Macros
那些不是 `C++` 关键字的关键字， signals, slots, Q_OBJECT, emit, SIGNAL, SLOT.
这些关键字是 Qt 对 `C++` 的扩展，他们实际上是简单的宏，定义在 [qobjectdefs.h](https://code.woboq.org/qt5/qtbase/src/corelib/kernel/qobjectdefs.h.html#66) 中.

```C++
#define signals public
#define slots /* nothing */
```
由此可以看出信号和槽就是普通的成员函数，但是关键字可以被 MOC 识别，用来生成代码。

```C++
#define Q_OBJECT \
public: \
    static const QMetaObject staticMetaObject; \
    virtual const QMetaObject *metaObject() const; \
    virtual void *qt_metacast(const char *); \
    virtual int qt_metacall(QMetaObject::Call, int, void **); \
    QT_TR_FUNCTIONS /* translations helper */ \
private: \
    Q_DECL_HIDDEN static void qt_static_metacall(QObject *, QMetaObject::Call, int, void **);

```
`Q_OBJECT` 定义了一个静态的 `QMetaObject` 和一系列的函数，并会在 MOC 生成的代码中实现它们。


```C++
#define emit /* nothing */
```
`emit` 是一个空宏，甚至 MOC 也不会解析，只用于提示。


```C++
Q_CORE_EXPORT const char *qFlagLocation(const char *method);
#ifndef QT_NO_DEBUG
# define QLOCATION "\0" __FILE__ ":" QTOSTRING(__LINE__)
# define SLOT(a)     qFlagLocation("1"#a QLOCATION)
# define SIGNAL(a)   qFlagLocation("2"#a QLOCATION)
#else
# define SLOT(a)     "1"#a
# define SIGNAL(a)   "2"#a
#endif
```
这里的宏将参数转换成字符串并在前面加上一个码。`qFlagLocation` 会将字符串地址注册到一个包含两个条目的表中。

## MOC Generated Code
### QMetaObject

```C++
// 以 counter 类为例
const QMetaObject Counter::staticMetaObject = {
    { &QObject::staticMetaObject, qt_meta_stringdata_Counter.data,
      qt_meta_data_Counter,  qt_static_metacall, Q_NULLPTR, Q_NULLPTR}
};


const QMetaObject *Counter::metaObject() const
{
    return QObject::d_ptr->metaObject ? QObject::d_ptr->dynamicMetaObject() : &staticMetaObject;
}
```
在这里实现了 `staticMetaObject` 和函数 `metaObject()` 。 `QObject::d_ptr->metaObject` 一般只用于动态元数据类型（QML Object）, 所以大多数情况下会返回 `staticMetaObject`。


```C++
struct QMetaObject
{
    /* ... Skiped all the public functions ... */

    enum Call { InvokeMetaMethod, ReadProperty, WriteProperty, /*...*/ };

    struct { // private data
        const QMetaObject *superdata;
        const QByteArrayData *stringdata;
        const uint *data;
        typedef void (*StaticMetacallFunction)(QObject *, QMetaObject::Call, int, void **);
        StaticMetacallFunction static_metacall;
        const QMetaObject **relatedMetaObjects;
        void *extradata; //reserved for future use
    } d;
};
```

使用 `d` 间接访问是为了表明所有的成员应该是私有的，但实际上，这里的数据成员都不是私用的，是为了保持 `POD` 的特性并允许静态初始化。
当父对象（`staticMetaObject`）初始化的时候，`superdata`，`stringdata`,`data` 都会进行初始化。
`static_metacall` 是一个函数指针。


### Introspection Tables
首先，这里会生成一个整数数据的表。

```C++
static const uint qt_meta_data_Counter[] = {

 // content:
       7,       // revision
       0,       // classname
       0,    0, // classinfo
       2,   14, // methods
       0,    0, // properties
       0,    0, // enums/sets
       0,    0, // constructors
       0,       // flags
       1,       // signalCount

 // signals: name, argc, parameters, tag, flags
       1,    1,   24,    2, 0x06 /* Public */,

 // slots: name, argc, parameters, tag, flags
       4,    1,   27,    2, 0x0a /* Public */,

 // signals: parameters
    QMetaType::Void, QMetaType::Int,    3,

 // slots: parameters
    QMetaType::Void, QMetaType::Int,    5,

       0        // eod
};
```
最开始的 13 个数据是表头，当有两列数据时，第一列是数量，第二列是描述信息的开始索引。
方法的描述由 5 个整数符合而成。


### <span id="string_table">String Table </span>
```C++

struct qt_meta_stringdata_Counter_t {
    QByteArrayData data[6];
    char stringdata0[46];
};

#define QT_MOC_LITERAL(idx, ofs, len) \
    Q_STATIC_BYTE_ARRAY_DATA_HEADER_INITIALIZER_WITH_OFFSET(len, \
    qptrdiff(offsetof(qt_meta_stringdata_Counter_t, stringdata0) + ofs \
        - idx * sizeof(QByteArrayData)) \
    )

static const qt_meta_stringdata_Counter_t qt_meta_stringdata_Counter = {
    {
        QT_MOC_LITERAL(0, 0, 7), // "Counter"
        QT_MOC_LITERAL(1, 8, 12), // "valueChanged"
        QT_MOC_LITERAL(2, 21, 0), // ""
        QT_MOC_LITERAL(3, 22, 8), // "newValue"
        QT_MOC_LITERAL(4, 31, 8), // "setValue"
        QT_MOC_LITERAL(5, 40, 5) // "value"
    },

    "Counter\0valueChanged\0\0newValue\0setValue\0"
    "value"
};
#undef QT_MOC_LITERAL


```
这里实现了一个静态的 `QByteArray` 的数组，通过 `QT_MOC_LITERAL` 创建了一个静态的 `QByteArray`， 它指向下面字符串中特定的索引。


### signals
``` C++ 
// SIGNAL 0
void Counter::valueChanged(int _t1)
{
    void *_a[] = { Q_NULLPTR, const_cast<void*>(reinterpret_cast<const void*>(&_t1)) };
    QMetaObject::activate(this, &staticMetaObject, 0, _a);
}
``` 
信号的实现本质上是一个简单的函数，在函数里面创建了一个指向参数的指针数组，第一个参数是它的返回值。并将这个数组传递给了 `QMetaObject::activate` 函数。`QMetaObject::activate` 的第三个参数是信号的[索引](#index)。


### Calling a Slot
``` C++ 
void Counter::qt_static_metacall(QObject *_o, QMetaObject::Call _c, int _id, void **_a)
{
    if (_c == QMetaObject::InvokeMetaMethod) {
        Q_ASSERT(staticMetaObject.cast(_o));
        Counter *_t = static_cast<Counter *>(_o);
        Q_UNUSED(_t)
        switch (_id) {
        case 0: _t->valueChanged((*reinterpret_cast< int(*)>(_a[1]))); break;
        case 1: _t->setValue((*reinterpret_cast< int(*)>(_a[1]))); break;
        default: ;
        }
    }
}
```
在 `qt_static_metacall` 函数中，通过 `slot` 的索引来调用它。指向参数的数组指针与用于信号中的数组指针格式相同（_a）。`a[0]` 没有被用到，因为所有的返回值都是 `void`.


## <span id="index">Index</span>
在每个 `QMetaObject` 中，该对象的 `signal`, `slot`, 以及其他可被调用的方法都会被赋予一个索引，索引从 0 开始，它们的顺序是 `signal`, `slot` 然后是其他方法。这个索引在内部被称为相对索引。

总的来说，我们不想要知道一个更加全局化的索引，这个索引并不相对于一个特定的类，而是包括整个继承链中的所有其他方法。为此，我们可以给相对索引加上一个偏移获得绝对索引。绝对索引用在公共的 API 中，通过类似 `QMetaObject::indexOf{Signal,Slot,Method}` 这样的函数返回。

连接机制使用了由 `signal` 索引的 `vector`, 但是所有的 `slots` 都浪费了 `vector` 中的空间，并且往往一个对象中， `slots` 都多于 `signal`。
从 Qt4.6 开始，使用了新的内部信号索引，它仅包含信号的索引



## How Connecting Works
Qt 在进行连接时，做的第一件事情就是找到 `signal` 和 `slot` 的索引。Qt 会在 [String Table](#string_table) 中查找到 `meta object` 相应的索引。

然后创建一个 `QObjectPrivate::Connection` 对象，并将它添加到一个内部的链表中。

对于一个给定的 `signal index`, 我们需要能够快速的去访问它。因为可能有多个槽连接到同一个信号，对于每个信号，都需要有个记录槽的链表。每个连接都需要包含接收对象和槽的索引，因为我们希望当对象销毁时，这个连接也自动销毁，所以接收者需要知道谁连向了它。

```C++
struct QObjectPrivate::Connection
{
    QObject *sender;
    QObject *receiver;

    union {
        StaticMetaCallFunction callFunction;
        QtPrivate::QSlotObjectBase *slotObj;
    };

    // The next pointer for the singly-linked ConnectionList
    Connection *nextConnectionList;
    //senders linked list
    Connection *next;
    Connection **prev;
    QAtomicPointer<const int> argumentTypes;
    QAtomicInt ref_;
    ushort method_offset;
    ushort method_relative;
    
    uint signal_index : 27; // In signal range (see QObjectPrivate::signalIndex())
    ushort connectionType : 3; // 0 == auto, 1 == direct, 2 == queued, 4 == blocking
    ushort isSlotObject : 1;
    ushort ownArgumentTypes : 1;
    
    Connection() : nextConnectionList(0), ref_(2), ownArgumentTypes(true) {
        //ref_ is 2 for the use in the internal lists, and for the use in QMetaObject::Connection
    }

    ~Connection();
    int method() const { return method_offset + method_relative; }
    void ref() { ref_.ref(); }
    void deref() {
        if (!ref_.deref()) {
            Q_ASSERT(!receiver);
            delete this;
        }
    }
};
```
![connection linked list](/assets/images/connection_list.png "连接链表")

每个对象都维护了一个连接的 `vector`, 它为每个信号关联了一个 `QObjectPrivate::Connection` 的链表。

每个对象同时也维护了一个相反的链表，它是一个双向链表，用于自动删除对象所链接的 `Connection`

![connection linked list detail](/assets/images/connection_detail_list.png "连接链表详情")

`prev` 指针是一个指向指针的指针，因为它并没有实际指向前一个节点。这个指针只在连接被销毁时用到，而不是用于向后迭代。

## Signal Emission
当我们调用信号时，实际会调用 `MOC` 生成的代码，会调用到 `QMetaObject::activate` 函数。

``` C++ 
void QMetaObject::activate(QObject *sender, const QMetaObject *m, int local_signal_index,
                           void **argv)
{
    activate(sender, QMetaObjectPrivate::signalOffset(m), local_signal_index, argv);
    /* We just forward to the next function here. We pass the signal offset of
     * the meta object rather than the QMetaObject itself
     * It is split into two functions because QML internals will call the later. */
}

void QMetaObject::activate(QObject *sender, int signalOffset, int local_signal_index, void **argv)
{
    int signal_index = signalOffset + local_signal_index;

    /* The first thing we do is quickly check a bit-mask of 64 bits. If it is 0,
     * we are sure there is nothing connected to this signal, and we can return
     * quickly, which means emitting a signal connected to no slot is extremely
     * fast. */
    if (!sender->d_func()->isSignalConnected(signal_index))
        return; // nothing connected to these signals, and no spy

    /* ... Skipped some debugging and QML hooks, and some sanity check ... */

    /* We lock a mutex because all operations in the connectionLists are thread safe */
    QMutexLocker locker(signalSlotLock(sender));

    /* Get the ConnectionList for this signal.  I simplified a bit here. The real code
     * also refcount the list and do sanity checks */
    QObjectConnectionListVector *connectionLists = sender->d_func()->connectionLists;
    const QObjectPrivate::ConnectionList *list =
        &connectionLists->at(signal_index);

    QObjectPrivate::Connection *c = list->first;
    if (!c) continue;
    // We need to check against last here to ensure that signals added
    // during the signal emission are not emitted in this emission.
    QObjectPrivate::Connection *last = list->last;

    /* Now iterates, for each slot */
    do {
        if (!c->receiver)
            continue;

        QObject * const receiver = c->receiver;
        const bool receiverInSameThread = QThread::currentThreadId() == receiver->d_func()->threadData->threadId;

        // determine if this connection should be sent immediately or
        // put into the event queue
        if ((c->connectionType == Qt::AutoConnection && !receiverInSameThread)
            || (c->connectionType == Qt::QueuedConnection)) {
            /* Will basically copy the argument and post an event */
            queued_activate(sender, signal_index, c, argv);
            continue;
        } else if (c->connectionType == Qt::BlockingQueuedConnection) {
            /* ... Skipped ... */
            continue;
        }

        /* Helper struct that sets the sender() (and reset it backs when it
         * goes out of scope */
        QConnectionSenderSwitcher sw;
        if (receiverInSameThread)
            sw.switchSender(receiver, sender, signal_index);

        const QObjectPrivate::StaticMetaCallFunction callFunction = c->callFunction;
        const int method_relative = c->method_relative;
        if (c->isSlotObject) {
            /* ... Skipped....  Qt5-style connection to function pointer */
        } else if (callFunction && c->method_offset <= receiver->metaObject()->methodOffset()) {
            /* If we have a callFunction (a pointer to the qt_static_metacall
             * generated by moc) we will call it. We also need to check the
             * saved metodOffset is still valid (we could be called from the
             * destructor) */
            locker.unlock(); // We must not keep the lock while calling use code
            callFunction(receiver, QMetaObject::InvokeMetaMethod, method_relative, argv);
            locker.relock();
        } else {
            /* Fallback for dynamic objects */
            const int method = method_relative + c->method_offset;
            locker.unlock();
            metacall(receiver, QMetaObject::InvokeMetaMethod, method, argv);
            locker.relock();
        }

        // Check if the object was not deleted by the slot
        if (connectionLists->orphaned) break;
    } while (c != last && (c = c->nextConnectionList) != 0);
}
```
- 首先检查一个64位的位掩码 `signal_index`， 判断是否有信号连接。
- 然后对发射信号的对象加锁，开始操作它的连接列表。
- 拿到这个信号对应的连接列表。
- 检查一遍 `last` 指针，确保在此次信号发射期间添加进来的槽，不会被遍历到。
- 开始遍历所有连接到该信号的槽，
- 根据连接的类型判断是否需要直接发送，还是放到事件队列中
- 调用接收方的槽函数

## 相关
[Qt 中的信号与槽 (2)]({% link _posts/2023-08-13-qt-signal-and-slot-two.md %})


## 引用
- https://doc.qt.io/qt-6/signalsandslots.html
- https://doc.qt.io/qt-6/metaobjects.html
- https://woboq.com/blog/how-qt-signals-slots-work.html