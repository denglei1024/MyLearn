---
tags:
  - publish
---

#### 前言

在前一篇里有一个思考：如何创建复用性更好的语义函数？本篇文章，将回答这个问题，是通过结构化处理语义函数（Semantic Function，下文直接称语义函数）的各个部分，来达到复用的效果。

> 演示案例是C#编写的控制台应用，使用的是.net8.0版本。
>
> LLM服务使用的是Azure OpenAI



#### 结构化语义函数

1、创建`Plugins`目录，这个目录存放所有的插件，通常会有多个插件，每个插件一个目录，最后一级的目录是特定的函数。

> 例如下面这个图中，Plugins是所有插件的顶级目录。
>
> ConverterPlugin表示有个“转换插件”。
>
> Json2Model表示在转换插件下有个特定功能的函数，可以把JSON文档转换为特定语言的数据模型。

![image-20240422221928690](https://note-1251668647.cos.ap-nanjing.myqcloud.com/image-20240422221928690.png)

2、在`Json2Model`这个函数目录下，创建`config.json`和`skprompt.txt`两个文件

其中`config.json`的示例代码如下：
```json
{
  "schema": 1,
  "type": "completion",
  "description": "将JSON文档转换为指定语言的数据模型",
  "execution_settings": {
    "default": {
      "max_tokens": 1200,
      "temperature": 0.8
    }
  },
  "input_variables": [
    {
      "name": "input",
      "description": "用户输入的JSON文档",
      "required": true
    }
  ]
}
```

> type:  提示词类型
>
> description:  描述函数的功能
>
> execution_settings: 模型的参数
>
> input_variables: 定义提示词模板的参数
>
> - name: 参数名称
> - description: 参数描述

`skprompt.txt`是提示词模板文件，示例内容如下：

```
担任高级{{$language}}开发人员。将此 JSON 文档转换为 {{$language}} 数据模型。

--- 开始 ---
{{$input}}
--- 结束 ---
```

如果要定义参数，可以使用`{{$参数}}`定义，参数名称对应`input_variables`下的`name`。

#### 案例演示

对前一篇示例进行改写。

1、导入插件

```c#
var skill = kernel.ImportPluginFromPromptDirectory("Plugins/ConverterPlugin");
```

2、准备用户输入内容

```C#
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
```

3、调用函数，获得大模型返回结果

```
var result = await kernel.InvokeAsync<string>(skill["Json2Model"],new KernelArguments() { ["input"] = input, ["language"]="java" });
Console.WriteLine(result);
```

结果示例

![image-20240422223257630](https://note-1251668647.cos.ap-nanjing.myqcloud.com/image-20240422223257630.png)

#### 总结

本文说明了如何通过对语义函数的配置和提示词模板结构化处理后，可以方便的在不同的应用中使用，以达到复用的目标。



#### 进一步思考

不管是本机函数还是语义函数，都是由程序主动调用的。如果我希望在生产应用中，可以动态的组合函数，交给AI来调用，如何达成这个目标呢？