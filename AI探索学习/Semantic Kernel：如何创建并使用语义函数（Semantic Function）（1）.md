
#### 前言

接上篇，本文讲解语义函数（Semantic Function）。语义函数不像本机函数那样需要我们使用编程语言编写代码实现特定的功能，它是**自然语义编程**，通过定制提示词模板，**让LLM做它本来就可以做的事情**。有了Semantic Kernel，在这个定制化的提示词模板中，可以动态的传入一些参数。

> 本篇以C#编写的控制台程序为例，使用.Net8.0。
>
> LLM服务使用的是Azure OpenAI

#### 内嵌语义函数

语义函数也有多种写法，比较简单的就是直接写在特定的程序里。假如现在要做一个网页应用，将用户输入的JSON文档，转化为特定语言的数据模型。

##### 1、创建项目，并安装依赖包

```powershell
# 创建控制台应用
dotnet new console -o dotnet-sk-semantic-function
# cd到`dotnet-sk-semantic-function`目录
dotnet add package Microsoft.KernelMemory.Core
dotnet add package Microsoft.SemanticKernel
```

##### 2、创建`kernel`对象

```c#
var builder = Kernel.CreateBuilder()
    .AddAzureOpenAIChatCompletion(Env.Var("AOAI_MODEL_ID"), Env.Var("AOAI_ENDPOINT"), Env.Var("AOAI_API_KEY"));

var kernel = builder.Build();
```

##### 3、定义提示词模板

这里的提示词模板，可以用你知道的各种提示词技巧来写。如果想在提示词里传入参数，可以使用`{{$参数}}`来定义。

```
var skPrompt = """
               担任高级{{$language}}开发人员。将此 JSON 文档转换为 {{$language}} 数据模型。

               --- 开始 ---
               {{$input}}
               --- 结束 ---
               """;
```

##### 4、初始化 OpenAI 提示词的执行请求设置

可以指定温度、生成的最大令牌数等等，更多可配置项，查看类的定义。

```c#
var openAiPromptExecutionSettings = new OpenAIPromptExecutionSettings
{
    MaxTokens = 8000,
    Temperature = 1
};
```

 ##### 5、创建函数对象

通过`kernel`创建函数对象。`convertJson2Model`是函数的名称，最后是函数的描述。

```
var function = kernel.CreateFunctionFromPrompt(skPrompt, openAiPromptExecutionSettings, "convertJson2Model", "将JSON文档转换为数据模型");
```

##### 6、调用函数

示例中的`input`和`language`是提示词模板中定义的参数，组合提示词后会一并传入到LLM中，得到回应。

```c#
//用户输入示例内容
var input = """
            {
              "person": {
                "name": "Alice",
                "age": 28,
                "city": "New York",
                "occupation": "Software Engineer"
              },
              "pets": [
                {
                  "name": "Fluffy",
                  "species": "Cat",
                  "age": 5
                },
                {
                  "name": "Max",
                  "species": "Dog",
                  "age": 3
                }
              ],
              "favorite_foods": ["Sushi", "Pizza", "Chocolate"]
            }

            """;
//转换为C#数据模型
await kernel.InvokeAsync(function, new KernelArguments() { ["input"] = input, ["language"] = "c#" })
    .ContinueWith((convertResult) => { Console.WriteLine(convertResult.Result); });
```

生成结果的截图如下：
![image-20240421170921308](https://note-1251668647.cos.ap-nanjing.myqcloud.com/image-20240421170921308.png)

#### 总结

本篇使用了内嵌的方式编写了语义函数，语义函数的创建步骤可以概况为下面四个步骤：
1、定义提示词模板

2、初始化执行请求设置

3、创建函数对象

4、调用函数

#### 进一步思考

这样编写的代码，复用性不好，只能在特定的应用中使用，如何创建复用性更好的语义函数呢？





