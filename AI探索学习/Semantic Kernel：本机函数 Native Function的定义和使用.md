---
tags:
  - publish
---

## 前言

作为SK（Semantic Kernel）核心组件之一，Plugins是一组为了实现功能而定义的函数的集合，可以达到增强LLM（大语言模型）的能力。

LLM的训练数据都是截止到过去的某一个时间，所以如果问它现在发生的事情，那么生成的内容，基本上是它在一本正经的胡说八道。LLM也不能进行调用API、发送邮件、发布文章等等操作，如果想让LLM获取新的知识和能力，就需要为它开发一些插件。

这里有一张从微软官方网站上找到了这篇图。

![](https://note-1251668647.cos.ap-nanjing.myqcloud.com/cross-platform-plugins.png)

> 演示示例使用的控制台应用程序，编程语言是C#，.net版本是8.0
>
> 使用Azure OpenAI部署的gpt-3.5-turbo大模型
>
> 本机函数功能是获取今天的日期

## 本机函数

SK的插件有本机函数和语义函数两类。下面我会演示如何创建本机函数，并手动调用它。

1、创建控制台应用

```shell
dotnet new console -o dotnet-sk-plugins
```

2、cd到`dotnet-sk-plugins`目录下，添加相关的依赖包

```shell
dotnet add package Microsoft.KernelMemory.Core
dotnet add package Microsoft.SemanticKernel
```

3、创建`TimePlugin`插件类

项目结构如下：

```markdown
dotnet-sk-plugins
|--Plugins
|--|--Plugin.Time
|--|--|--TimePlugin.cs
|--Program.cs
```

代码示例如下：

```c#
using System.ComponentModel;
using Microsoft.SemanticKernel;

namespace dotnet_sk_plugins.Plugins.Plugin.Time;

public sealed class TimePlugin
{
    [KernelFunction, Description("获取今天的日期")]
    public static string GetToday()
    {
        return DateTime.Today.ToString("yyyy-M-d dddd");
    }
}
```

4、手动调用插件

```c#
using dotnet_sk_plugins.Plugins.Plugin.Time;
using Microsoft.KernelMemory;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;
using Microsoft.SemanticKernel.Connectors.OpenAI;

var builder = Kernel.CreateBuilder()
    .AddAzureOpenAIChatCompletion(Env.Var("AOAI_MODEL_ID"), Env.Var("AOAI_ENDPOINT"), Env.Var("AOAI_API_KEY"));

builder.Plugins.AddFromType<TimePlugin>();

//1、手动调用
var kernel = builder.Build();
var answer = await kernel.InvokeAsync<string>(
    "TimePlugin",
    "GetToday"
);

Console.WriteLine("今天是: " + answer);
```

## 结语

这篇文章展示了如何定义一个本机函数，并手动调用它。