---
title: '【总结】Unity Input System 使用简要'
date: 2020-02-10 18:36:02
tags: [Unity,笔记]
cover: https://s1.ax1x.com/2020/03/13/8KPHT1.jpg
coverWidth: 1280
coverHeight: 726
categories:
- 旧站迁移
---
有关 Unity 新输入系统 Input System 的一些简单介绍和基础总结。目前关于新输入系统的中文资料略少，在 Unity 官方文档中学习了一番之后，我将一些基本的知识记录了下来。

<!-- more -->

## 新输入系统 (Input System) 概述

在 Unity 设计之初，并没有预见到现在如此丰富的平台和设备支持规模，所以一开始设计的输入系统 `UnityEngine.Input` 使用起来并不舒适，在多设备和多平台输入处理时显得力不从心且不够优雅，甚至连游戏中热插拔手柄这种操作都显得十分臃肿和复杂。Unity Tec 从 2016 年起开始逐步开发新一代输入系统 `UnityEngine.InputSystem` ，到今天（Unity 2019.3.0f6），已经迭代到了 `1.0.0-preview.4` 版本。官方表示大概在 Unity 2020 版本推出新输入系统的正式版，但是旧输入系统的下架时间并没有被明确说明。

新输入系统基于事件，输入设备和动作逻辑互相分离，通过配置映射来处理输入信息。具有易用，多设备多平台协调一致的特点。现在已经可以在 Package Manager 中安装使用了。

## Input System 安装

在 Unity 2019.1 之后的版本中，打开包管理器（Windows -> Package Manager），在 Advanced 菜单中勾选预览版支持（Show Preview Packages），接着就可以在 All Packages 列表中找到 Input System 了，点击 Install 安装即可。

导入过程中会跳出警告窗口告知需要激活新输入系统的后端，点击是会重启编辑器，此时便启用了新输入系统。

接着打开 Edit -> Project Settings，在 Player 选项页中的 Other Settings -> Configuration 中设置 Active Input Handling 为 Input System 以禁用旧输入系统，以防止两个输入系统的冲突和杜绝隐患。

此时，你的工程中已经成功安装了新输入系统。

## Input System 简单介绍

有关新输入系统的内容几乎都在`UnityEngine.InputSystem`命名空间下，使用时不要忘记 `using UnityEngine.InputSystem`。一些特殊的对象可能在子命名空间下，具体请查阅文档。

### `Class InputAction`

这个类表示的是一种响应动作逻辑，可以绑定到多个物理输入上，这些绑定的物理输入和得到的输入值将影响同一个 `Input Action` 对象。该类只代表一种动作“逻辑”而不代表任何物理输入。

每个动作逻辑在每一时刻都有现在的状态阶段（Phase），通过 `Enum InputActionPhase` 表示，有五种阶段。分别为：`Canceled, Disabled, Performed, Started, Waiting`。在`Start, Performed, Canceled` 阶段会分别触发三个对应的 C# 事件（类型为`event Action<InputAction.CallbackContext>`）。

每次动作逻辑被触发时，可以通过`ReadValue<TValue>()`成员来获取本次触发的具体值。

声明：

```CSharp
[Serializable] public sealed class InputAction : ICloneable, IDisposable
```

### `Struct InputAction.CallbackContext `

这个结构体将作为动作逻辑被触发时事件的参数使用，内部含有关于本次触发情况的所有参数。

### `InputBindings`

表示动作逻辑和输入设备之间的绑定关系。每个 Binding 需要有源输入设备（Path）和绑定的动作逻辑（Input Action），可以有输入条件限制（Interactions）限制设备的触发条件和后处理器（Processors）来进行输入源数据的后处理。

一个绑定可能有以下几种合成类型（Composite Type）：`1D Axis, 2D Vector, Button, Button with modifier, Custom composite`，分别表示单轴输入，双轴向量输入，按钮输入，带有条件的按钮输入和自定义输入数据类型。

`Asset InputActionAssets`

Input Action Assets 是一种存储动作绑定关系的资源文件，扩展名为`.InputActions`，数据格式为 JSON。可以通过 Project 页面中 Create -> Input Actions 来创建一个新的 Input Action Assets。

Input Action Assets 的编辑界面如下：

![编辑页面](https://s2.ax1x.com/2020/02/10/15WiH1.png)

每个资源文件可以存储多个映射列表，最左侧一栏黄色标签代表不同的映射列表。中间一栏表示当前映射列表中存储的 Input Action（绿色标签），每个Input Action可以有多个Input Bindings（蓝色标签），每个Input Binding 视类型可以有多个合成分量（Composite Bindings）（粉色标签）。右侧一栏表示当前对象的属性，可以进行调整。

Input Action Assets 可以生成指定格式的 C# 类，可以通过操作该类来进行相同的操作。可以根据喜好选择生成与否。

### `Class PlayerInput`

新输入系统提供的类，可以通过添加到 Game Object 上作为组件使用。本文只介绍其 Inspector 可配置内容。

Inspector 中的 Player Input 组件：

![Player Input](https://s2.ax1x.com/2020/02/10/15hyX4.png)

其中：

- Actions 表示其使用的 Input Action Assets 资源文件，Default Map 表示其使用资源文件中的哪个映射列表。
- UI Input Module 项可以通过指定一个 `InputSystemUIInputModule`对象来与 UI 协作。
- Camera 项用于分屏游戏时指定该 Player Input 对应的摄像机对象。
- Behavior 项表示当有动作逻辑触发时的相应方式，有`Send Messages, Broadcast Messages, Unity Events, CSharp Events`四种触发模式。分别有着慢，更慢，可视化，快的特点。但是前两个实现方式比较无脑简单，在游戏对象内实现命名为对应的 `On{ActionName}()`函数即可接受对应触发效果，Unity Events 方式需要在 Inspector 中分配各种函数，虽然可视化，但是在函数和动作逻辑较多时比较麻烦。C# Events 一如既往的优雅而又有效率，但是唯一的不足是所有自定义触发都共享同一个事件`event Action<InputAction.CallbackContext> onActionTriggerd`，需要在订阅事件之后自行比较触发的动作逻辑是否是该订阅者需要的，这种方式可以说是有利有弊。

## Input System 四种基本使用方式

官方给出的 Sample 给出了四种使用新输入系统的基本方式：

### 使用输入 State 

直接使用当前输入设备的状态来控制对象，没什么特别的。大概相当于用旧输入系统的思想使用新输入系统，唯一的进步大概是输入设备变得标准化了。

示例（都是英文句子就不注释了）：

```CSharp
public void Update()
{
    var gp = Gamepad.current;
    if (gp == null) return;
    
    Vector2 leftStick = gp.leftStick.ReadValue(),
            rightStick = gp.rightStick.ReadValue();
    
    if (gp.buttonSouth.wasPressedThisFrame) Debug.Log("Pressed");
    if (gp.buttonSouth.wasReleasedThisFrame) Debug.Log("Released");
    
    //...
}
```

### 使用 Player Input 控件

通过添加 Player Input 控件来管理输入，这种方式绝大部分绑定操作需要在 Inspector 中完成，代码只需要写出每个事件触发时的订阅者（命名必须为`On{ActionName}()`，C# 事件方式除外）即可。

下面是两种类型的动作逻辑值的订阅者示例（只演示两种事件类型，我拒绝`SendMessage()`）：

*使用 Unity Event 的 Player Input 在 Inspector 中的设置:*

![UnityEventPlayerInput](https://s2.ax1x.com/2020/02/10/15X978.png)

```CSharp
public class Player : MonoBehaviour
{
    // Unity Event的情况，需要在Inspector中进行本函数的订阅操作
    public void OnAttack(InputAction.CallbackContext callback)
    {
        switch (callback.phase)
        {
            case InputActionPhase.Performed:
                Debug.Log("Attacking!");
        }
        move = callback.ReadValue<Vector2>();
    }
    
    // C# 事件的情况，与上方情况不共存
    private PlayerInput input;
    private Vector2 move;
    private void Awake()
    {
        // 添加订阅者函数, 当然也可以写成独立的函数
        input = GetComponent<PlayerInput>().onActionTriggered += 
            callback =>
        {
            if (callback.action.name == "Move")
            {
                move = callback.ReadValue<Vector2>();
            }
        };
        
        // ...
    }
}
```

### 使用 Action Assets 生成的 C# 类

刚才说过 Action Assets 资源文件可以生成一个 C# 类，通过建立一个这个类对象来进行控制可以达到使用资源文件相同的效果。

假设我们导出了一个`CustomControl` 类，则有示例：

```CSharp
public class Player : MonoBehaviour
{
    private CustomControl input;
    
    private void Awake()
    {
        input = new CustomControl(); // 实例化
        
        // 通过订阅事件的方式添加控制方式
        // Player是映射列表, Attack是一个InputAction名字
        input.Player.Attack.performed += 
            callback =>
        {
            Debug.Log("Attacking!")
        }
    }
    
    private void Update()
    {
        // 通过主动读取InputAction状态获取输入
        Vector2 move = input.Player.Move.ReadValue<Vector2>();
        
        //...
    }
}
```

### 使用自定义的 Input Action 列表

喜闻乐见的一点，Input Action 对象是可以在 Inspector 中显示的（废话，不然 Assets 编辑列表显示什么）。所以可以通过自定义 Input Action 对象来处理输入。

以下是一种示例：

```CSharp
public class Player : MonoBehaviour
{
    public InputAction moveAction;
    
    public void OnEnable()
    {
        moveAction.Enable();
    }
    
    public void OnDisable()
    {
        moveAction.Disable();
    }
    
     // 此时我们自定义的InputAction即可正常使用
    public void Update()
    {
        var move = moveAction.ReadValue<Vector2>();
        
        //...
    }
}
```

*自定义的 Input Action 在 Inspector 中的显示效果：*

![CustomInputAction](https://s2.ax1x.com/2020/02/10/15XUHK.png)

## 结语

当前网上关于新输入系统的中文资料真的很少，希望[这篇文章](https://blog.dctewi.com/post/unity-new-input-system)能够帮到你，也希望 Unity 越来越好。

## 参考资料

- [Unity新一代输入系统介绍 - Unity Connect](https://connect.unity.com/p/unityxin-yi-dai-shu-ru-xi-tong-jie-shao)
- [Manual | Input System | 1.0.0-preview.4  - Unity Docs](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.0/manual/)
- [Scripting API | Input System | 1.0.0-preview.4 - Unity Docs](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.0/api/index.html)

