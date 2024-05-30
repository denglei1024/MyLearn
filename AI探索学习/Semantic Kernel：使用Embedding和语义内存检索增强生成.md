---
tags:
  - publish
---

## 前言

最近在捣鼓研究Semantic Kernel，对如何在AI对话中引用私有的知识库比较感兴趣。目前比较常见的做法是fine-tuning或embedding。这篇文章就来看看我是如何使用semantic kernel搭配embeddings模型的。

> 示例采用控制台应用，编程语言是C#，使用的是.net 8.0。
>
> GPT模型使用的是Azure OpenAI GPT-3.5-turbo。

> embeddings简单来说，是将非结构化的文本通过embedding转换为数值向量的方法。这些向量反映了文本的语义和关系。

## 操作演示

下面开始演示操作步骤：

1、首先确认已经在Azure OpenAI服务中，部署了两个模型，分别是GPT-3.5-turbo和text-embedding-3-small。

2、创建控制台应用`kmdemo01`

3、引入包

semantic kernel相关的包

```shell
dotnet add package Microsoft.KernelMemory.Core
dotnet add package Microsoft.SemanticKernel
```

环境配置包

```shell
dotnet add package dotenv.net
```

4、修改`program.cs`文件，添加如下代码

```C#
using Microsoft.KernelMemory;
using Microsoft.SemanticKernel;

DotEnv.Load();

var env = DotEnv.Read();
var embeddingConfig = new AzureOpenAIConfig()
{
    APIKey = env["API_KEY"],
    Deployment = env["EMBEDDING_NAME"],
    Endpoint = env["ENDPOINT"],
    APIType = AzureOpenAIConfig.APITypes.EmbeddingGeneration,
    Auth = AzureOpenAIConfig.AuthTypes.APIKey
};
var textConfig = new AzureOpenAIConfig()
{
    APIKey = env["API_KEY"],
    Deployment = env["TEXT_NAME"],
    Endpoint = env["ENDPOINT"],
    APIType = AzureOpenAIConfig.APITypes.ChatCompletion,
    Auth = AzureOpenAIConfig.AuthTypes.APIKey
};
var kernel = Kernel.CreateBuilder()
    .AddAzureOpenAIChatCompletion(env["TEXT_NAME"], env["ENDPOINT"], env["API_KEY"])
    .Build();

var memory = new KernelMemoryBuilder()
    .WithAzureOpenAITextEmbeddingGeneration(embeddingConfig)
    .WithAzureOpenAITextGeneration(textConfig)
    .WithSimpleVectorDb()
    .Build<MemoryServerless>();

await memory.ImportWebPageAsync("https://raw.githubusercontent.com/microsoft/kernel-memory/main/README.md");
await memory.ImportWebPageAsync("https://juejin.cn/post/7323408577709080610");
Console.WriteLine("文档已经准备好，开始提问吧！");
while (true)
{
    var userInput = Console.ReadLine();
    var answer = await memory.AskAsync(userInput);
    Console.WriteLine(answer.Result);
    Console.WriteLine("参考:");
    foreach (var source in answer.RelevantSources)
    {
        Console.WriteLine($" - {source.SourceName}, {source.Link}[{source.Partitions.First()}{source.Partitions.First().LastUpdate:D}]");
    }
}
```

运行程序，在控制台可以看到如下面截图所示的内容，这里就是导入文本到语义内存里的过程。

![image-20240418000858996](https://note-1251668647.cos.ap-nanjing.myqcloud.com/image-20240418000858996.png)

下面是我根据代码中导入的文档进行问答的截图

![image-20240418001031086](https://note-1251668647.cos.ap-nanjing.myqcloud.com/image-20240418001031086.png)

## 结语

这篇文章展示了通过kernel memory导入私有知识库来检索增强生成。embeddings保证了知识库的隐私，又利用了大模型的能力，相比fine-turning来说，则它是成本更低并且效果很好的一种方式。通过semantic kernel来开发，操作起来也是非常的简单。
