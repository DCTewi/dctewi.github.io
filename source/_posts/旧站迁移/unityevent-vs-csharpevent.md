---
title: '【总结】UnityEvent 和 C# event 的差异与选择'
date: 2020-02-03 13:54:04
tags: [笔记,Unity,UnityEvent]
cover: https://s-sh-2563-tewi-box.oss.dogecdn.com/img/blog-header/kaf-hand-dark.jpg
coverWidth: 1280
coverHeight: 726
categories:
- 旧站迁移
---
Event 是一种十分方便的机制。一个 Event 就好像是一个 Youtuber，而他的粉丝 Subscribers/Listeners 会在这个 Event 发生的时候进行响应（观看）。许多现代语言都有类似的订阅机制，比如 Java 中的 EventBus 类。

<!-- more -->

​这种结构有利于保持各对象之间的独立性和架构的可扩展性。比如当玩家类进行开枪操作时，玩家类可以不必关心他的这个操作造成了什么结果，而只是 Invoke 一个事件。此时，粒子系统等就可以根据这个事件的发生进行在自己的处理，而玩家类并不关心这个事件造成了什么后果。这样可以起到减少耦合的作用。

## C# Delegate

​delegate，巨硬给出的官方翻译是委托 。delegate 本质上是一个用来存放函数的容器。众所周知，在面向对象的语言中，变量可以是值类型，也可以是引用类型。而 C# 则用 delegate 的机制将函数引用存储起来，并且可以在运行时被改变。delegate 特别用于实现事件和回调方法，一种委托将提前约定好存放的函数的返回类型和参数列表。

​delegate 可以被初始化成一个函数的引用，也可以通过 +=，-= 等运算符进行存放多个函数引用。存放多个函数时，调用 delegate 将会调用所有的函数引用，这种行为叫做 multicast delegate，即委托的多播。

## C# 原生 event

C# 的原生 event 其实基于 C# 的语言特性“委托” delegate。从另一个角度看，C# 的 event 就是一种 multicast delegate。

​C# 原生事件的使用其实十分易懂。每个 event 的都需要约定一种委托类型用于存储该事件的所有订阅者，如果委托不需要参数，则使用`EventArgs.Empty`来进行占位。event 不需要进行实例化，订阅者只需要在订阅时对 event 进行 `+=` 操作，退订时使用 `-=` 操作即可。触发事件则有直接像调用函数一样使用 `()` 运算符触发和使用成员函数 `Invoke()` 触发两种方式。

​以下是在 Unity 中使用 C# event 的一个示例（EventManager 为事件， Listener 为订阅者）。

```c#
// EventManager.cs //
using System;
using UnityEngine;

public class EventManager : MonoBehaviour
{
    // 事件参数
    public class MouseDownEventArgs : EventArgs
    {
        public string Message { get; private set; }
		// 构造函数
        public MouseDownEventArgs(string message)
        {
            Message = message;
        }
    }
    // 事件的委托声明
    public delegate void MouseDownEventHandler(object sender, MouseDownEventArgs e);
	// 事件
    static public event MouseDownEventHandler OnMouseDownEvent;

    private void Update()
    {
        if (Input.GetMouseButtonDown(0))
        {
            // 触发事件
            OnMouseDownEvent.Invoke(this, new MouseDownEventArgs("Hello!"));
           	// OnMouseDownEvent(this, new MouseDownEventArgs("Hello!")); // 另一种触发方式
        }
    }
}
```

```c#
// Listener.cs //
using System;
using UnityEngine;

public class Listener : MonoBehaviour
{
    private void OnEnable()
    {
        EventManager.OnMouseDownEvent += OnMouseDownEventInvoked; // 订阅
    }

    private void OnDisable()
    {
        EventManager.OnMouseDownEvent -= OnMouseDownEventInvoked; // 退订
    }

    // 订阅者函数
    private void OnMouseDownEventInvoked(object sender, EventManager.MouseDownEventArgs e)
    {
        Debug.Log(string.Format("Mouse click! {0}, {1}", sender, e.Message), gameObject);
    }
}
```

## UnityEvent

​C# event 设计之初当然不会为了 Unity 进行优化和适配，所以 Unity 对 C# event 进行了一番魔改和封装得到了 `UnityAction` 和 `UnityEvent` 类，都包含在 `UnityEngine.Event`命名空间下。一定程度上简化了事件的使用（我觉得本来已经够简化了），并且支持在 Unity Editor 的 Inspector 中直接进行可视化编辑。比如 Unity 的 uGUI 的 Button 等的诸如 `OnClick()` 事件都是通过 `UnityEvent` 来实现的，所以可以在 Inspector 上可视化编辑。

​一个`UnityEvent`的使用示例如下。

```c#
// 无参数 来源于 Unity Doc
using UnityEngine;
using UnityEngine.Events;
using System.Collections;

public class ExampleClass : MonoBehaviour
{
    // 事件定义
    UnityEvent m_MyEvent;

    void Start()
    {
        // 实例化事件
        if (m_MyEvent == null)
            m_MyEvent = new UnityEvent();
		// 添加订阅者
        m_MyEvent.AddListener(Ping);
    }

    void Update()
    {
        if (Input.anyKeyDown && m_MyEvent != null)
        {
            // 事件触发
            m_MyEvent.Invoke();
        }
    }

    // 订阅者函数
    void Ping()
    {
        Debug.Log("Ping");
    }
}
```

```c#
// 单个参数 来源于 Unity Doc
using UnityEngine;
using UnityEngine.Events;

// 添加 System.Serializable 属性的参数才可以被 UnityEvent 使用
[System.Serializable]
public class MyIntEvent : UnityEvent<int>
{
}

public class ExampleClass : MonoBehaviour
{
    public MyIntEvent m_MyEvent;

    void Start()
    {
        if (m_MyEvent == null)
            m_MyEvent = new MyIntEvent();

        m_MyEvent.AddListener(Ping);
    }

    void Update()
    {
        if (Input.anyKeyDown && m_MyEvent != null)
        {
            m_MyEvent.Invoke(5);
        }
    }

    void Ping(int i)
    {
        Debug.Log("Ping" + i);
    }
}
```

## UnityEvent or C# event ？

​经过 Jackson Dunstan 的性能分析（链接在后方参考资料处），可以发现 `UnityEvent` 在内存使用、GC回收和响应速度上都不如 C# event。如果订阅者小于等于一的情况下，`UnityEvent` 产生的垃圾小于 C# event，反之则要多。`UnityEvent`在首次触发事件时会产生垃圾，而 C# event 则没有垃圾，且前者的速度比后者慢两倍之多。C# event 在所有批量测试上都优于 `UnityEvent`，且触发事件越多，差距越大。最好情况下 `UnityEvent` 花费的时间是 C# event 的2.25倍，最坏情况下达到了40倍。

​在实际使用中，除了性能问题之外，还会发现另外的问题。`UnityEvent` 是一个类，需要实例化的特点让开发者需要注意在使用该事件时该事件的实例化情况。并且在进行自定义事件参数的时候会出现很麻烦的问题，比如在 Inspector 中不显示而导致可视化的优势也莫得了等等。

​所以，对于有一定基础并且对性能有追求的开发者来说，选择 C# event 是较好的选择。如果对性能要求不大，并且喜欢可视化编程的开发者来说，可以使用 `UnityEvent` 来得到更好的开发体验。当然，如果是开发 Editor Plugin 的场景，则显然使用 `UnityEvent` 更为恰当。所以两者的选择也需要具体情况具体分析。

## 参考资料

- [event (C# reference) - Microsoft .NET Doc](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/event)
- [Unity - Scripting API - UnityEvent](https://docs.unity3d.com/ScriptReference/Events.UnityEvent.html)
- [C# Event vs UnityEvent vs Messaging performance - Reddit](https://www.reddit.com/r/Unity3D/comments/a4ucmv/c_event_vs_unityevent_vs_messaging_performance/)
- [Why choose UnityEvent over native C# events? - StackOverFlow](https://stackoverflow.com/questions/44734580/why-choose-unityevent-over-native-c-sharp-events)
- [Event Performance: C# vs. UnityEvent - Jackson Dunstan](https://jacksondunstan.com/articles/3335)

