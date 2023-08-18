---
layout: post
title:  "Qt 中的信号与槽 (2)"
---

# 信号与槽的实现原理（2） {ignore=true}

[toc]

## Qt5 中的新语法
```C++
QObeject::connect(&a, &Counter::valueChanged, &b, &Counter::setValue); 
```

新的语言支持编译时类型检查，当信号与槽的参数不匹配时，会进行自动类型转换。同时，还支持 lambda 表达式。

## 新的重载
``` C++
// 非实际的代码
QObject::connect(const QObject *sender, PointerToMemberFunction signal,
                 const QObject *receiver, PointerToMemberFunction slot,
                 Qt::ConnectionType type)

QObject::connect(const QObject *sender, PointerToMemberFunction signal,
                 PointerToFunction method)

QObject::connect(const QObject *sender, PointerToMemberFunction signal,
                 Functor method)
```
通过增加一个新的重载实现新的语法，参数为函数支持，而不是之前的字符串。

## 类型萃取：`QtPrivate::FunctionPointer`

`traits` 是一个辅助类的函数，帮助获取一个给定类型的元数据。

```C++
    template<class Obj, typename Ret, typename... Args> struct FunctionPointer<Ret (Obj::*) (Args...)>
    {
        typedef Obj Object;
        typedef List<Args...>  Arguments;
        typedef Ret ReturnType;
        typedef Ret (Obj::*Function) (Args...);
        enum {ArgumentCount = sizeof...(Args), IsPointerToMemberFunction = true};
        template <typename SignalArgs, typename R>
        static void call(Function f, Obj *o, void **arg) {
            FunctorCall<typename Indexes<ArgumentCount>::Value, SignalArgs, R, Function>::call(f, o, arg);
        }
    };
```

`template<typename T> struct FunctionPointer` 会通过它的成员变量获得类型 `T` 的信息。
- ArgumentCount: 参数数量
- Object: 指针指向成员函数时才存在，是函数所属类的类型别名
- Arguments: 参数列表
- call(T &function, QObject *receiver, void **args): 静态函数，会使用给定的参数调用原函数。

### `QObject::connect`
```C++
template <typename Func1, typename Func2>
static inline QMetaObject::Connection connect(
    const typename QtPrivate::FunctionPointer<Func1>::Object *sender, Func1 signal,
    const typename QtPrivate::FunctionPointer<Func2>::Object *receiver, Func2 slot,
    Qt::ConnectionType type = Qt::AutoConnection)
{
  typedef QtPrivate::FunctionPointer<Func1> SignalType;
  typedef QtPrivate::FunctionPointer<Func2> SlotType;

  //compilation error if the arguments does not match.
  Q_STATIC_ASSERT_X(int(SignalType::ArgumentCount) >= int(SlotType::ArgumentCount),
                    "The slot requires more arguments than the signal provides.");
  Q_STATIC_ASSERT_X((QtPrivate::CheckCompatibleArguments<typename SignalType::Arguments,
                                                         typename SlotType::Arguments>::value),
                    "Signal and slot arguments are not compatible.");
  Q_STATIC_ASSERT_X((QtPrivate::AreArgumentsCompatible<typename SlotType::ReturnType,
                                                       typename SignalType::ReturnType>::value),
                    "Return type of the slot is not compatible with the return type of the signal.");

  const int *types;
  /* ... Skipped initialization of types, used for QueuedConnection ...*/

  QtPrivate::QSlotObjectBase *slotObj = new QtPrivate::QSlotObject<Func2,
        typename QtPrivate::List_Left<typename SignalType::Arguments, SlotType::ArgumentCount>::Value,
        typename SignalType::ReturnType>(slot);


  return connectImpl(sender, reinterpret_cast<void **>(&signal),
                     receiver, reinterpret_cast<void **>(&slot), slotObj,
                     type, types, &SignalType::Object::staticMetaObject);
}
```
在实际的实现中，函数签名中的 `sender` 和 `receiver` 并不仅仅是 `QObject*`, 它们指向了 `typename FunctionPointer::Object`.这里使用了 [SFISNAE](https://en.wikipedia.org/wiki/Substitution_failure_is_not_an_error) 技巧确保这个函数只能用于成员函数指针, 因为前文中的 `Object` 只在 ` FunctionPointer` 的类型为成员函数指针时才存在。

一系列 `Q_STATIC_ASSERT` 宏用于检测条件是否满足并给出错误提示。

然后会创建一个 `QSlotObject` 的对象，并给到函数 `connectImpl()`, 它是槽的一个包裹器，用于协助调用槽函数。它也知道信号的参数，所以可以进行一些适当的参数类型转换。

使用了 `List_left` 只传递了和槽函数参数数量相等的参数，这样就允许将参数较少的槽连接到参数较多的信号上。

`QObject::connectImpl` 是一个私有的内部函数，实现了连接的操作。它并没有保存方法的索引在 `QObjectPrivate::Connection` 结构体中，而是保存了 `QSlotObjectBase` 的指针。 



## 引用
- [Signals and Slots in Qt5](https://woboq.com/blog/how-qt-signals-slots-work-part2-qt5.html)

- [SFISNAE](https://en.wikipedia.org/wiki/Substitution_failure_is_not_an_error)