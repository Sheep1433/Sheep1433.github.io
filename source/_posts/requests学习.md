---
title: requests学习
author: 
date: 2022-04-17 10:22:51
tags: requests
---

## __init__模块	

init总体结构只包含check_compatibility和_check_ctyptography两个函数，对版本信息进行校验

收获：

在函数内部多次assert，如果没有完全通过，则会抛出AssertError

对版本信息验证是否在2.0.0~3.0.0之间，可以使用 assert (2,0,0) <= (major, minor, patch) < (3,0,0) 或者assert [2,0,0] <= (major, minor, patch) < [3,0,0]，归因于python列表(元组)比较大小

导包过程中 from . import utils中的.表示当前位置

## compat、_internal_utils模块

负责python2、python3以及简单的编码处理

## certs模块

CA证书相关

## exceptions模块

异常处理模块，应用到了super、hasattr、多继承等，还有字典的pop方法，pop方法可以加一个默认返回值kwargs.pop('response', None)，具体工作原理还需要结合调用时学习

cookies模块

hooks模块

packages模块	

## status_codes模块

该模块中有所有的状态信息，但是内部直接通过_init()执行，似乎没有被调用类或变量，还需要models等模块中结合学习

## auth模块

身份认证相关，在models、adapter、sessions等模块中调用了，学习这几个模块再返回学习

## API模块

api模块中，包含request、get、option、post等函数，而get等通过调用request实现函数功能，而request调用的是session.request函数，同时使用with sessions.Session() as session:对session进行上下文管理，确保会话及时关闭，避免出现类似内存泄漏的情况

## structures模块

新建了两种数据结构，CaseInsensitiveDict和LookupDict，其中CaseInsensitiveDict继承于MutableMapping，实现了__setitem__,__getitem__,__delitem__,__iter__,__len__等方法，解决大小写问题，在self._store中存储的键值形式为 "lower_key" : ("real_key","value")，查询时用小写查询，同时能保留原信息，具体实现方法很值得学习

## session模块

1. 为什么通过requests.Session能访问sessions模块中的Session类，因为在requests库的__init__文件中导入了Session

   实现了跨请求保持cookies方法，具体实现方法还没有看懂，核心在于request方法，首先通过models模块中生成Request类，再执行相应的prepare_request方法，最后执行send方法实现功能，常规通过requests.get方法不能保持cookies可能是因为在get方法中调用的是及时关闭session的请求方法

   session和api中都有request、get等方法，api中的本质上仍然调用的是session的方法

Session类有一个mount方法，def mount(self, prefix, adapter):	是将对应的adapter注册到对应的会话前缀上，使用场景比如为所有请求设置一个超时时间

```python
from requests.adapters import HTTPAdapter
DEFAULT_TIMEOUT = 5 # seconds

class TimeoutHTTPAdapter(HTTPAdapter):
    def __init__(self, *args, **kwargs):
        self.timeout = DEFAULT_TIMEOUT
        if "timeout" in kwargs:
            self.timeout = kwargs["timeout"]
            del kwargs["timeout"]
        super().__init__(*args, **kwargs)

    def send(self, request, **kwargs):
        timeout = kwargs.get("timeout")
        if timeout is None:
            kwargs["timeout"] = self.timeout
        return super().send(request, **kwargs)
```

使用时

```python
import requests

http = requests.Session()

# 此挂载对http和https都有效
adapter = TimeoutHTTPAdapter(timeout=2.5)
http.mount("https://", adapter)
http.mount("http://", adapter)

# 设置默认超时为2.5秒
response = http.get("https://api.twilio.com/")

# 通常为特定的请求重写超时时间
response = http.get("https://api.twilio.com/", timeout=10)
```

## models模块

 models模块中首先创建了RequestEncodingMixin、RequestHooksMixin基类，类中只有相关方法，供后续子类调用，其中RequestHooksMixin类中使用了self.hooks属性，hooks属性由其子类创建，因此此处使用了Mixin模式，并且父类调用了子类的属性。

```python
class RequestHooksMixin(object):
    def register_hook(self, event, hook):
        """Properly register a hook."""

        if event not in self.hooks:
            raise ValueError('Unsupported event specified, with event name "%s"' % (event))

        if isinstance(hook, Callable):
            self.hooks[event].append(hook)
```

此外用到了Callable，这是一种可调用执行对象，并且可以执行参数，也就是在对象后面使用小括号执行代码，那么这个对象就是Callable对象，callable包含函数、类、类里的方法、实现了__callable__方法的实例对象

```python
class Stu(object):
    def __init__(self, name):
        self.name = name

    def __call__(self, *args, **kwargs):
        self.run()

    def run(self):
        print('{name} is running'.format(name=self.name))

stu = Stu('小明')
print(callable(stu))    # True
stu() 
```

这个stu就是实例对象，但是在内部实现了__callable__方法，就可以像函数一样调用

在该模块中定义了request和response类，并实现了相关方法

cookies

CookieJar是什么

## hooks模块

hooks即钩子方法，用于在某个框架固定的某个流程执行是捎带执行（钩上）某个自定义的方法，requests中只支持了response的钩子，我们可以做一些响应检查或者在响应中添加信息等功能，示例如下：

**场景一：对响应状态码进行判断，如果失败抛出错误**

常规做法

```python
# 每个请求都需要实现一次
r = http.get("https://api.github.com/user/repos?page=1")
r.raise_for_status()
```

结合hooks

```python
http = requests.Session()

assert_status_hook = lambda response, *args, **kwargs: response.raise_for_status()
http.hooks["response"] = [assert_status_hook]

http.get("https://api.github.com/user/repos?page=1")
http.post("https://api.github.com/user/repos?page=1")
```



## 文件上传请求

requests发送上传文件请求

```python
url = "https://www.example.com/upload"
filepath = "/path/to/file.txt"

with open(filepath, "rb") as f:
    # 创建一个字典，用于包装要上传的文件数据
    file_data = {"file": (filepath, f)}
    # 发送上传文件的请求
    response = requests.post(url, files=file_data)
    # 打印服务器端返回的响应内容
    print(response.text)
```

urllib3发送上传文件请求

```python
url = "https://www.example.com/upload"
filepath = "/path/to/file.txt"

with open(filepath, "rb") as f:
    # 创建一个 urllib3.PoolManager 对象
    http = urllib3.PoolManager()
    # 构造要上传的文件数据
    file_data = {"file": (filepath, f, "application/octet-stream")}
    # 发送上传文件的请求
    response = http.request("POST", url, fields=file_data)
    # 打印服务器端返回的响应内容
    print(response.data.decode("utf-8"))
```

urllib3需要指明文件的MIME类型
