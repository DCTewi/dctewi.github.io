---
title: '【总结】我的 Unity C# 编码规范'
date: 2020-09-04 22:05:50
tags: [Unity,笔记,编码规范]
cover: https://s-sh-2563-tewi-box.oss.dogecdn.com/img%2Fblog-header%2FSB%E5%8D%A1%E7%A5%96%E7%AC%9B.png
coverWidth: 1280
coverHeight: 726
categories:
- 旧站迁移
---
好久没水文章了。所以在这个考试过去了一半了的时间里发点东西，算是起个头，之后又要肝起来咯。

本套编码规范只代表个人习惯，符合习惯的规范就是好规范。有观点欢迎讨论，但谢绝人身攻击哦。

<!-- more -->

## 命名

1. 不使用匈牙利命名法，不在变量前添加`m_`等前缀。
2. 使用大驼峰命名**属性**、**方法**、**事件**、**枚举**、**类型**和**命名空间**。

```csharp
public class MyClass
{
    public event Action SomeEvent;
    public int Name { get; set; }
    
    [MenuItem("Window/SomeMenu")]
    public void DoSomething()
    {
    }
}
```

3. 使用大驼峰命名**静态变量**，**常量**，和**公开字段(包括公开只读字段)**。
4. 使用小驼峰命名**私有字段**，**临时变量**，**参数**，不应/不能被其他对象访问的私有字段应添加一个下划线前缀。
5. 布尔变量由小写介词短语开头，表示逻辑类型。

```csharp
public static string AssetPath = "Asset/";
public const double Pi = 3.1415926;
public readonly int PhoneNumber = 123456;
public string Name = "Hanpi";

private int _health;

public bool isOnline;
private bool canPlayGame;
```

5. 接口名以大写I作为前缀。

```csharp
public interface IManager
{
}
```

6. 类名应该是名词或者名词性短语。
7. 方法名应该是动宾/介宾短语。
8. 局部变量的名称要有意义，慎用简写/缩写。

```csharp
public class GameManager
{
    public string GetName()
    {
        var tempVar = MakeName();
        return tempVar;
    }
}
```

## 结构

类型成员的排列顺序自上而下依次为：

1. 静态：静态字段，静态属性，静态方法；
2. 字段：私有字段，受保护字段（尽量不要使用公开字段）；
3. 属性：私有属性，受保护属性，公开属性；
4. 事件：私有事件，受保护事件，公开事件；
5. 嵌套枚举，嵌套类；
6. 构造函数（如果有），按参数数量升序排列；
7. 重载方法（如果有），按需排列；
8. 方法，按需排列；
9. Unity 保留方法（如`Update()`），按调用时间前后排序；

注1：如有需要，应该使用`partial`将一部分放在`ClassName.PartialType.cs`文件中。比如玩家的不同状态机类。

注2：如果文件行数较多，则应该使用`#region`分开代码块并折叠。

## 格式

### 缩进

1. 不使用 Tab 字符，使用四个空格字符控制一个缩进。

### 括号

1. 左大括号总是在开始块的语句下一行的行首，块内容应再缩进四个空格。

```csharp
if (condition)
{
    DoSth();
}
```

2. Switch-Case 语句的缩进方法：

```csharp
switch (s)
{
    case 0:
        {
            DoSth();
        }
        break;
    default: break;
}
```

3. 单行语句中，可以有在同一行开始或结束的括号。

```csharp
public class Foo
{
   int bar;
   public int Bar
   {
      get { return bar; }
      set { bar = value; }
   }
}
```

### 空格

1. 函数参数之间逗号后使用单个空格。

```csharp
Right:       Console.In.Read(myChar, 0, 1);
Wrong:       Console.In.Read(myChar,0,1);
```

2. 圆括号和参数后面不能使用空格。

```csharp
Right:       CreateFoo(myChar, 0, 1)
Wrong:       CreateFoo( myChar, 0, 1 )
```

3. 函数名称和括号之间不能使用空格。

```csharp
Right:       CreateFoo()
Wrong:       CreateFoo ()
```

4. 不要在括号内使用空格。

```csharp
Right:       x = dataArray[index];
Wrong:       x = dataArray[ index ];
```

5. 在流程控制语句之前使用单个空格。
6. 在算术运算符前后添加一个空格。

```csharp
Right:       while (x == y)
Wrong:       while(x==y)
```

### 注释

1. 版权声明（若选择）

版权声明卸载文件头，不应该使用三斜杠注释，应该使用双斜杠注释。

```csharp
//-----------------------------------------------------------------------
// <copyright file="ContainerControl.cs" company="Microsoft">
//     Copyright (c) Microsoft Corporation.  All rights reserved.
// </copyright>
//-----------------------------------------------------------------------
```

2. 方法注释

方法注释应该使用三斜杠XML文档注释。对于开发者互相的评论，应该使用`<devdoc>`标签。

`<summary>`标签格式应该为：`xxx of/which/about/... something`。

```csharp
public class Foo 
{
/// <summary>Public stuff about the method</summary>
/// <param name=”bar”>What a neat parameter!</param>
/// <devdoc>Cool internal stuff!</devdoc>
///
public void MyMethod(int bar) { … }
}
```

3. 代码注释

使用双斜杠注释。注释应放在代码的上方。

```csharp
// I'm a comment.
var rb = GetComponent<RigidBody2D>();
```

### 命名空间

命名空间应根据路径分配，脚本文件的命名空间在`using`语句下方空出一行处。

```csharp
using System;

namespace ProjectName.Core
{
    public class MyClass() { }
}
```