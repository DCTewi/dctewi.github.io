---
title: '【开发】Unity的类MVC架构总结'
date: 2019-03-27 23:00:35
tags: [Unity,旧文归档]
cover: https://s-sh-2563-tewi-box.oss.dogecdn.com/img/blog-header/minoto-paint.jpg
coverWidth: 1280
coverHeight: 726
categories:
- 旧站迁移
---
> 画UI画的头疼，想水一篇文章。本来想总结一下DP，但是发现东西实在是太多了，准备多写一些东西再放上来。所以先总结一下最近学到的Unity架构知识。

<!-- more -->

作为一个常年写竞赛型代码的菜鸡选手，往往不太会用这种写工程用的高级架构。但是写游戏又是一个不小的工程，所以一个清晰的架构就显得特别重要了。Unity编程中，这种类MVC架构我之前就在Unity官方案例中见到过，但是在图书馆借了相关材料之后才进行了系统的学习。

## MVC架构

> MVC模式（Model–view–controller）是软件工程中的一种软件架构模式，把软件系统分为三个基本部分：模型（Model）、视图（View）和控制器（Controller）。

- 控制器（Controller）- 负责转发请求，对请求进行处理。
- 视图（View） - 界面设计人员进行图形界面设计。
- 模型（Model） - 程序员编写程序应有的功能（实现算法等等）、数据库专家进行数据管理和数据库设计(可以实现具体的功能)。

而这种由Manager管理的结构和MVC的分工稍有出入，被书中称为类MVC架构。

## 类MVC架构

由一个`IGameManager`接口封装所有Manager类的基本结构，由一个`Managers`总类来管理和实例化所有的Manager类，每个Manager类负责不同的模块。

这样做的优点是代码结构清晰，管理、添加和删除功能模块的时候可以把互相之间的影响降低到最小。

## 具体实现

### 接口 IGameManager

首先是IGameManager接口类：

```c#
public enum ManagerStatus
{
    Shutdown,
    Initializing,
    Started
}

public interface IGameManager
{
    ManagerStatus status { get; }

    void Startup();
}

```

`ManagerStatus`枚举用来表示Manager类的实例化情况，`Startup()`负责分配`Manager`类在启动时所需要经历的行为，每个`Manager`内不需要单独实现`private void Start()`，而是在总的`Managers`类中统一执行`Startup()`。这么做可以手动控制`Manager`类之间的先后加载顺序来避免相互之间的依赖关系而导致的引用错误。

### 管理类 Managers

我们拟添加存档管理器`DataManager`、背包管理器`InventoryManager`和玩家管理器`PlayerManager`。

总管理类不需要继承自接口，代码大概如下：

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[RequireComponent(typeof(DataManager))]		// 注1
[RequireComponent(typeof(InventoryManager))]
[RequireComponent(typeof(PlayerManager))]

public class Managers : MonoBehaviour
{
    // Managers Instance
    public static Managers instance { get; private set; }	// 注2

    // Manager List
    public static DataManager Data;
    public static InventoryManager Inventory;
    public static PlayerManager Player;
    

    // Manager Load Sequeue
    private List<IGameManager> _startSequeue;	// 注3

    private void Awake()
    {
        // Check if there isn't an instance
        if (instance == null)
        {
            instance = this;
        }
        // Check if there is an instance
        else if (instance != this)
        {
            Destroy(gameObject);
            return;
        }
        // Set this Managers instance Don't Destroy on Load
        DontDestroyOnLoad(gameObject);

        // Get Managers Instances
        Data = GetComponent<DataManager>();
        Inventory = GetComponent<InventoryManager>();
        Player = GetComponent<PlayerManager>();

        // Add Managers to Load Sequeue
        _startSequeue = new List<IGameManager>
        {
            Player,
            Inventory,
            Data
        };
        // Start Load Managers
        StartCoroutine(StartupManagers());
    }

    private IEnumerator StartupManagers()
    {
        // Call Startup() of Every Manager
        foreach (var manager in _startSequeue)
        {
            manager.Startup();
        }
        // Wait for Next Frame
        yield return null;

        // Check the number of Managers
        int numModules = _startSequeue.Count, numReady = 0;
        // While not Loaded
        while (numReady < numModules)
        {
            int lastReady = numReady;
            numReady = 0;
            // Get the Status of Managers
            foreach (var manager in _startSequeue)
            {
                if (manager.status == ManagerStatus.Started)
                {
                    numReady++;
                }
            }

            // If undone
            if (numReady > lastReady)
            {
                Debug.Log("Progress: " + numReady + "/" + numModules);
                // Wait for Next Frame
                yield return null;
            }
        }
        // Complete
        Debug.Log("All managers started up");
    }
}
```

- 注1：

  这里的三个属性主要有两个作用。一是表明依赖关系，二是在绑定Game Object的时候可以只拖动Managers脚本，Unity会自动添加他需要的剩下的脚本文件。

- 注2：

  获取当前的实例，用来直接调用当前示例以及防止多次实例化。

- 注3：

  需要被初始化的`Manager`列表。众所周知，出列顺序是固定的，所以解决了`Start()`函数先后顺序无法确定的问题。

其他的都是一些朴实的模拟过程，配合注释即可食用。

### 玩家管理器 PlayerManager

只管理血量和血量上限，顺便带了一个更改血量的函数的简略代码：

```c#
using UnityEngine;

public class PlayerManager : MonoBehaviour, IGameManager
{
    public ManagerStatus status { get; private set; }

    public float health;
    public float maxHealth;

    public void Startup()
    {
        Debug.Log("Player manager starting...");

        health = 50f;
        maxHealth = 100f;

        status = ManagerStatus.Started;
    }

    public void ChangeHealth(float delta)
    {
        health += delta;
        if (health > maxHealth)
        {
            health = maxHealth;
        }
        else if (health < 0)
        {
            health = 0f;
        }

        Debug.Log("Player HP: " + health + "/" + maxHealth);
    }
}

```

很朴实的代码……继承自`MonoBehaviour`和`IGameManager`，实现一下接口的要求就没什么了。

### 背包管理器 InventoryManager

简略功能后的代码：

```c#
using System.Collections.Generic;
using UnityEngine;

public class InventoryManager : MonoBehaviour, IGameManager
{
    public ManagerStatus status { get; private set; }

    private Dictionary<string, int> _items;

    public void Startup()
    {
        Debug.Log("Inventory manager starting...");

        _items = new Dictionary<string, int>();

        status = ManagerStatus.Started;
    }

    public void UpdateData(Dictionary<string, int> newItems)
    {
        _items = newItems;
    }

    public Dictionary<string, int> GetData()
    {
        return _items;
    }
}

```

使用了`Dictonary<string, int>`来储存背包信息，但是这么做没有办法记录物品的摆放位置。可以再写一个位置类和物品类，然后用`Dictionary<Place, Item>`来储存，这里不再展开。

### 存档管理器 DataManager

保留这个主要是顺带记录一下序列化/反序列化来储存游戏存档的方法：

```c#
using System.IO;
using System.Collections.Generic;
using System.Runtime.Serialization.Formatters.Binary;
using UnityEngine;

public class DataManager : MonoBehaviour, IGameManager
{
    public static string SaveName = "save.dat";

    public ManagerStatus status { get; private set; }

    private string _savepath;

    public void Startup()
    {
        Debug.Log("Data manager starting...");

        _savepath = Path.Combine(Application.persistentDataPath, SaveName);
        Debug.Log("Set the " + SaveName + " in " + _savepath);

        status = ManagerStatus.Started;
    }

    [ContextMenu("Save")]
    public void Save()
    {
        // Dic of objects which should be saved
        Dictionary<string, object> gamestate = new Dictionary<string, object>
        {
            { "inventory", Managers.Inventory.GetData() },
            { "health", Managers.Player.health },
            { "maxHealth", Managers.Player.maxHealth }
        };

        // Format the serializable object to stream
        FileStream stream = File.Create(_savepath);
        BinaryFormatter formatter = new BinaryFormatter();
        formatter.Serialize(stream, gamestate);
        stream.Close();
    }
    
    [ContextMenu("Load")]
    public void Load()
    {
        try
        {
            if (!File.Exists(_savepath))
            {
                Debug.Log("No save file");
                throw new FileNotFoundException();
            }
        }
        catch (FileNotFoundException)
        {
            EventManager.SaveNotFoundEvent.Invoke();
            return;
        }

        Dictionary<string, object> gamestate;

        BinaryFormatter formatter = new BinaryFormatter();
        FileStream stream = File.Open(_savepath, FileMode.Open);

        // Deserialize the stream to object
        gamestate = formatter.Deserialize(stream) as Dictionary<string, object>;
        stream.Close();

        Managers.Inventory.UpdateData(gamestate["inventory"] as Dictionary<string, int>);
        //...
    }
}

```

这个保存和加载的过程涉及到了序列化/反序列化以及C#的装箱/拆箱操作。

- 保存

  1. `Dictionary<string, object> gamestate = new Dictionary<string, object>{...}`

     把所有想要保存的数据装箱。

  2. 实例化一个`FileStream`类对象用来输出文件，配合`BinaryFormatter`来把装箱后的数据保存到想要保存的位置。

  3. 关闭文件流。

- 加载

  1. 检查存档是否存在，若不存在则抛出异常、触发事件并返回（在这里不展开）。
  2. 创建一个接受数据的`Dictionary<string, object>`
  3. `BinaryFormatter`配合 `FileStream`从存档位置读入存档内容，关闭文件流
  4. 转换，拆箱，赋值。

## 小结

以上就是Unity工程中类MVC架构的大致结构。

~~Unity天下第一！！！（被拖走暴打）~~