### 做一个能够翻译md的Agent

#### 技术栈
1. LangChainjs

#### 模型
1. `Helsinki-NLP/opus-mt-zh-en`（huggingface中的模型）

#### 总结
1. 调用`huggingface`模型的时候，我在浏览器中运行是可以的，但是我在`node`端就不行，并且报`fetch failed`的错误，我一直以为是`node`版本的问题，但是直接请求`fetch`是可以的，因为`LangChain`内部的请求封装使用的就是`fetch`，那么只能是`huggingface`请求地址的问题，一般，内部使用的都是`https://api-inference.huggingface.co`，如果你是自定义请求地址可以使用中国镜像`https://api-inference.hf-mirror.com`,后面使用`Proxifier`软件，就可以了，在浏览可以是因为代理生效，但是在终端执行，并没有代理成功，使用该软件，让所有的请求都走代理就可以了