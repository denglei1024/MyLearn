---
tags:
  - publish
---

### 前言

作为一个开发者，在使用ChatGPT、文心一言、星火这些大模型的时候，是不是想要把这些大模型和自己的应用集成，做出一款赚钱的产品呢？那么，如何在不需要很多AI专业知识背景的情况下，更轻松地集成大模型呢？这就是本篇文章要介绍给大家的一个微软的开源框架——Semantic Kernel，中文翻译为语义内核。

### Semantic Kernel 是什么

一个开源框架，旨在将大模型 AI 编程与传统编程语言集成。它提供了开箱即用的模板、连接器和规划器，允许开发者可以轻松地将 LLM 嵌入到他们的应用系统中。支持的大模型有 OpenAI、Azure OpenAI、Hugging Face 等。

### 为什么需要 Semantic Kernel

自从看到 GPT3.5 的惊艳之后，越来越多的企业想把 LLM 的能力和他们的软件系统相结合，打造更有价值的产品。但是，直接接入 LLM 的过程是比较复杂的，需要了解大模型的很多细节。为了解决这个问题，微软开源了这个框架。它提供了一个高效简洁的抽象层，隐藏了 LLM 内部细节，让不是专业的 AI 开发者也能具有构建 AI 应用的能力

Semantic Kernel语言丰富，支持主流的编程语言，包括 C#、Java、Python。

### 演示使用Semantic Kernel实现多轮对话

下面请你和我一起创建一个控制台的多轮对话应用，来体验Semantic Kernel。

> 使用C#创建控制台应用，使用.NET8.0版本。
>
> LLM使用Azure OpenAI

- 通过创建控制台应用

```powershell
dotnet new console -o dotnet-sk-first
```

- 添加依赖

首先添加Semantic Kernel核心依赖包

```powershell
# cd到 dotnet-sk-first 目录
dotnet add package Microsoft.SemanticKernel
```

- 编辑`program.cs`

使用你喜欢的工具，编写下面的代码。

```c#
using Microsoft.SemanticKernel;

var builder = Kernel.CreateBuilder();

var kernle = builder.AddAzureOpenAIChatCompletion("YourAzureDeploymentName", "YourAzureEndpoint","YourAzureKey")
.Build();
```

- 在Azure OpenAI上部署好模型后，添加到系统的环境变量

> 我不想把敏感信息放到配置文件里，而且我希望其他应用也可以直接使用环境变量，而不用重复的创建配置。
>
> 在环境变量里创建下面三个变量：
>
> - AOAI_MODEL_ID: 部署名称
> - AOAI_ENDPOINT: 终结点
> - AOAI_API_KEY: 密钥

添加依赖包，用来读取环境变量

```powershell
dotnet add package Microsoft.KernelMemory.Core
```

修改上面`program.cs`的代码，传入实际的配置信息。

```c#
var kernle = builder.AddAzureOpenAIChatCompletion(Env.Var("AOAI_MODEL_ID"), Env.Var("AOAI_ENDPOINT"),Env.Var("AOAI_API_KEY"))
.Build();
```

- 完善代码，编写多轮对话代码

```c#
var history = new ChatHistory();

var chatCompletionService = kernel.GetRequiredService<IChatCompletionService>();

var executionSettings = new OpenAIPromptExecutionSettings{
 Temperature= 0.5,
 MaxTokens = 8000
};

Console.Write("你: ");
string? userInput;
while((userInput=Console.ReadLine())!=null){
    history.AddUserMessage(userInput);
    var response = await chatCompletionService.GetChatMessageContentAsync(history, executionSettings, kernel);
    Console.WriteLine("AI: "+ response);
    history.AddMessage(response.Role, response.Content??string.Empty);
    Console.Write("你：");
}
```

- 运行结果截图

![image-20240423235340414](https://note-1251668647.cos.ap-nanjing.myqcloud.com/image-20240423235340414.png)

### 总结

这篇文章讲解了开源框架语义内核通过隐藏了LLM内部细节，提供高效抽象层，让开发者轻松使用LLM的能力来开发应用，并编写了一个多轮对话的示例来体验语义内核。希望对你在学习AI编程方面有一些帮助。