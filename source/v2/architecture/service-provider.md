---
title: 服务提供者
---

# 服务提供者

服务提供者是`组件`和`CatLib`联系的桥梁。同时也是CatLib启动的中心，所有的服务都是通过服务提供者定义的。

## 名词定义

- `组件` 组件与CatLib没有任何关系，她们可以独立的运行在不同的框架中。
- `服务` 是由服务提供者将由`一个`或者`多个`组件组合而成，并提供一组可以被开发者使用的接口。
- `容器` CatLib 依赖注入容器。

## 架构图

<img src="../imgs/architecture-diagram.svg" alt="architecture-diagram" style="max-width:1200px;">

## 创建服务提供者

服务提供者是用来描述一个服务如何为使用者提供服务的，这些关系常常包括：服务是否是单例的，服务在构建后如何初始化等等。

服务提供者类的命名我们建议以：`Provider`开头，如：

- `ProviderFileSystem.cs`
- `ProviderNetwork.cs`

##### 为什么要使用服务提供者

- 服务提供者类可以从目录结构清晰的明确哪个类是服务注册类，如果不使用服务提供者类，那么开发者无法快速明确哪一个类为框架提供了服务。
- 服务提供者类似于胶水，粘连了服务和框架，依赖注入是通过构造函数自动进行的，这样在服务的开发过程中甚至可以完全不依赖于框架进行开发。
- 服务提供者类可以用于启动某些程序，例如在服务提供者中的Init方法注册一些事件，当事件触发时启动某些程序。

> 服务提供者只是一个管理工具，请善用服务提供者来引导或管理你的组件，以便于组件之间互相组合和协作。请充分发挥你的想象吧！

##### 写服务提供者和使用接口可能太麻烦？

有些人可能会批判，我明明一个简单的功能却要实现这么一大圈功能，意思是让代码显得太冗长，你必须定义接口然后实现它，再通过服务提供者注册。这要多敲多少代码！

对于小而简单的程序来说，以上说法并没有什么问题，在简单功能中接口通常是不必要的，将代码耦合到那些你认为不会改变的地方也是可行的，比如一次性的任务，或者一些原型或演示项目，毕竟这种灵活性会带来更多的代码量。
有时候的确不会改。而且小型的程序也不需要架构师，架构师们都是为大型项目服务的。

在大型项目中，接口是很有帮助的。和提升的代码灵活性、可测试性相比，多敲几下键盘花费的时间就显得微不足道了。

如果你在写小型程序的时候不想遵守接口原则，回退到原始模式，别觉得有什么问题，那没什么不对。但是你要为其带来的弊端做好评估。

#### 通过IServiceProvider来建立服务提供者

您的服务提供者类必须实现`CatLib.IServiceProvider`接口, 接口中包含`Register()`和`Init()`2个方法。

在`Register()`方法中，你`唯一`要做的事情就是`绑定服务实现`到服务容器，不要尝试在其中执行任何其它功能，否则将会引发一个`CodeStandardException`异常。

``` csharp
public class ProviderFileSystem : IServiceProvider
{
    public void Init(){ }
    public void Register()
    {
        App.Singleton<IFileSystem, FileSystem>();
    }
}
```

`Init()`方法会在所有的服务提供者的`Register()`方法执行后执行，这意味着我们可以在`Init()`中访问其他服务提供者所提供的服务。

``` csharp
public class ProviderConfig : IServiceProvider
{
    public void Init()
    { 
        var fileSystem = App.Make<IFileSystem>();
    }
    public void Register(){ }
}
```

#### 通过ServiceProvider来建立

您还可以通过继承`ServiceProvider`来简化服务提供者的构建方式：

``` csharp
public class ProviderConfig : ServiceProvider
{
    public override void Init()
    { 
        // todo: 
    }
}
```

## 注册服务提供者

如果框架的使用者想要使用某个服务，那么必须先对这个服务进行注册：

``` csharp
App.Register(new ProviderFileSystem());
```

> 注册了服务提供者并不意味着服务都会被立即实例化，通常情况下很多是延迟实例化的，只有真的用到它们的时候才会实例化。