
## 介绍

这篇文章将介绍如何配置Azure OpenAI，以及如何在Python中如何使用LangChain完成一次对话。

> 使用ipynb notebook
> 虚拟环境python版本 3.11.3

## 演示
### 1、安装依赖包
安装并更新`langchain`、`langchain-openai`包

```shell
%pip install -qU langchain
%pip install -qU langchain-openai
```

### 2、配置环境变量
你应该配置至少配置下面四个参数到环境变量中：
- `AZURE_OPENAI_API_KEY`，`AZURE_OPENAI_ENDPOINT`：在你创建 Azure OpenAI 服务后，可以在“密钥和终结点”菜单下查看

- `OPENAI_API_VERSION`：在这里查看 [https://learn.microsoft.com/en-us/azure/ai-services/openai/reference](https://learn.microsoft.com/en-us/azure/ai-services/openai/reference)

- `AZURE_OPENAI_CHAT_DEPLOYMENT_NAME`：部署的模型名称，如下图
![image.png](https://note-1251668647.cos.ap-nanjing.myqcloud.com/20240520164751.png)


### 3、对话代码
实例化一个AzureChatOpenAI的对象，指定 `openai_api_version` 和 `azure_deployment` 两个参数。定义消息列表 `messages`，包含系统信息和用户信息。调用 `invoke` 方法，访问LLM获得回应。

```python
import os
from langchain_core.messages import HumanMessage,SystemMessage
from langchain_openai import AzureChatOpenAI  

# create a llm
model = AzureChatOpenAI(
  openai_api_version = os.environ["AZURE_OPENAI_API_VERSION"],
  azure_deployment = os.environ["AZURE_OPENAI_CHAT_DEPLOYMENT_NAME"]
)  

# define the messages
messages = [
  SystemMessage(content="translate the following from english into chinese"),
  HumanMessage(content="hello!")
]  

# call llm
result = model.invoke(messages)
```


### 4、提取回应内容

result 一个AIMessage结构的数据，通常，我们需要把 `content` 的内容解析出来，展示给用户。
现在，让我们从 AIMessage 数据结构中提取`content`内容。

```python
from langchain_core.output_parsers import StrOutputParser
parser = StrOutputParser()
parser.invoke(result)
```

## 总结

在这篇文章里，介绍了 Azure OpenAI 的配置，以及在Python中使用LangChain完成一次简单的对话，希望对你有帮助。