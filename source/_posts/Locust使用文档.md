---
title: Locust使用文档
tag: Locust、性能测试
---

# Locust使用文档

## 简介

`Locust`是一个基于协程、简单易用的分布式用户负载测试工具。它用于web站点(或其他系统)的负载测试，并计算一个系统可以处理多少并发用户。

优点

- **用普通的Python编写用户测试场景** 不需要笨拙的UI或庞大的XML，只需像通常那样编码即可。基于协程而不是回调，您的代码看起来和行为都与正常的、阻塞Python代码一样。
- **分布式和可扩展——支持成千上万的用户** Locust支持在多台机器上运行负载测试。由于基于事件，即使一个Locust节点也可以在一个进程中处理数千个用户。这背后的部分原因是，即使你模拟了那么多用户，也不是所有用户都积极的访问你的系统。通常，用户无所事事，想知道下一步该怎么做。**每秒请求数**不等于在**线用户数**。
- **基于web的UI** Locust具有简洁的HTML + JS用户界面，可实时显示相关的测试细节。而且由于UI是基于Web的，因此它是跨平台且易于扩展的。

## 安装

0.安装3.7或更高版本python

1.安装包

```shell
pip install locust
```

要查看命令的使用，可以执行：

```shell
locust --help
```

## 快速入门

编写locust脚本，下面是一个简单的locustfile.py的例子：

```python
from locust import HttpUser, TaskSet, between


class UserBehavior(TaskSet):

    def on_start(self):
        login(self)

    def on_stop(self):
        logout(self)
    @task
	def hello_world(self):
        self.client.get("/hello")
        self.client.get("/world")
	
    @task(3)
    def view_items(self):

class WebsiteUser(HttpUser):
    tasks = UserBehavior
    wait_time = between(5.0, 9.0)
```

在当前路径执行locust命令，

```
locust
[2021-07-24 09:58:46,215] .../INFO/locust.main: Starting web interface at http://*:8089
[2021-07-24 09:58:46,285] .../INFO/locust.main: Starting Locust 2.15.1
```

打开web UI界面

Open [http://localhost:8089](http://localhost:8089/)，设定好用户数、用户上升速率和host即可触发性能测试。

## 编写locustfile

根据以上一个基本的脚本分析编写一个locustfile文件需要完成哪些部分。

```python
class UserBehavior(TaskSet):
```

首先声明一个UserBehavior类，该类继承自TaskSet，负责定义我们需要执行的任务集，具体任务定义通过使用task装饰器进行装饰。上述示例中使用task装饰了两个任务，其中一个被赋予了更高的权重（3），当任务启动时，每个用户会从这两个任务重随机挑选一个，而view_items被挑选中的概率大概是hello_world被挑选的3倍，在完成一次任务后，用户会进行等待wait_time重新进行任务选择。

此外，还有on_start和on_stop则是在启动、停止该任务会执行的一次操作，常见的登陆退出操作。

通过self.client属性发送的请求会被Locust记录下来，以便于展示数据。

```
class WebsiteUser(HttpUser):
```

其次声明一个继承自HttpUser的WebsiteUser类，该类表示用户，其中tasks属性指明需要执行的任务，wait_time是一个等待时间函数，默认的between(min_wait, max_wait函数是指用户在执行任务后会等待min_wait-max_wait秒，具体时间通过随机数生成。

HttpUser的父类是User，要使文件成为有效的locustfile，它必须包含至少一个继承自User的类，此外为获得更好的性能或者使用其他协议，也可以替换成FastHttpUser或其他，具体使用会在后面进行介绍。

### 请求分组

在数据统计时，有时存在一系列相同类型的url，我们希望将这些请求进行分组展示，这时可以传递name参数，如下：

```python
@task(3)
def view_items(self):
    for item_id in range(10):
        self.client.get(f"/item?id={item_id}", name="/item")
        time.sleep(1)
```

在某些情况下，将参数传入请求函数是不可能的，例如当与包装请求会话的库/SDK交互时。通过设置客户机提供了分组请求的另一种方法。request_name属性。

```python
# Statistics for these requests will be grouped under: /blog/?id=[id]
self.client.request_name="/blog?id=[id]"
for i in range(10):
    self.client.get("/blog?id=%i" % i)
self.client.request_name=None
```

如果希望用最少的样板文件链接多个组，可以使用client.rename_request()上下文管理器。

### 多用户

如果文件中存在多个用户类，并且在命令行上没有指定用户类，则Locust将生成相同数量的每个用户类。你也可以通过将用户类作为命令行参数传递给同一个locustfile来指定要使用的用户类:

```
locust -f locust_file.py WebUser MobileUser
```

如果希望模拟某种类型的更多用户，可以在这些类上设置权重属性。例如，网络用户的可能性是移动用户的三倍:

```python
class WebUser(User):
    weight = 3
    ...

class MobileUser(User):
    weight = 1
    ...
```

还可以设置fixed_count属性。在这种情况下，weight属性将被忽略，并将生成确切的用户计数。首先生成这些用户。在下面的示例中，将只生成一个AdminUser实例，以便独立于总用户数更准确地控制请求计数。

```python
class AdminUser(User):
    wait_time = constant(600)
    fixed_count = 1

    @task
    def restart_app(self):
        ...

class WebUser(User):
    ...
```

### host属性

host属性指的是要测试的主机的URL前缀，可以在web UI或者命令行--host选项或者User类中进行设置。

### tasks属性

指明任务

```python
class MyUser(User):
    wait_time = between(5, 15)

    @task(3)
    def task1(self):
        pass

    @task(6)
    def task2(self):
        pass
```

```python
class WebsiteUser(HttpUser):
    tasks = UserBehavior
    wait_time = between(5.0, 9.0)
```

在用户类中，可以使用task装饰器标记任务，也可以使用tasks属性标记，tasks属性可以是一个任务列表，也可以是一个<Task: int>字典，其中Task科一是一个python可调用对象，或者是一个TaskSet类。如果任务是一个普通的python函数，它们会接收一个参数，即正在执行任务的User实例。

如果将tasks属性指定为列表，则每次要执行任务时，将从tasks属性中随机选择该任务。然而，如果任务是一个字典——可调用对象作为键，整型作为值——要执行的任务将随机选择，但以整型作为比率。有一个任务是这样的:

```
{my_task: 3, another_task: 1}
```

### on_start和on_stop方法

用户(和任务集)可以声明一个on_start方法和/或on_stop方法。用户将在开始运行时调用它的on_start方法，在停止运行时调用它的on_stop方法。对于TaskSet, on_start方法在模拟用户开始执行该TaskSet时被调用，on_stop方法在模拟用户停止执行该TaskSet时被调用(当调用interrupt()或用户被杀死时)。

### tag装饰器

通过使用@tag装饰器标记任务，可以使用--tags和--exclude-tags参数对测试期间执行的任务进行筛选。考虑下面的例子:

```python
from locust import User, constant, task, tag

class MyUser(User):
    wait_time = constant(1)

    @tag('tag1')
    @task
    def task1(self):
        pass

    @tag('tag1', 'tag2')
    @task
    def task2(self):
        pass

    @tag('tag3')
    @task
    def task3(self):
        pass

    @task
    def task4(self):
        pass
```

### Event

模块级别的setup、teardown动作是通过Event实现的，比如需要在性能测试的开始或停止时运行一些代码，则应该使用test_start和test_stop事件。你可以在locustfile的模块级别为这些事件设置监听器:

```python
from locust import events

@events.test_start.add_listener
def on_test_start(environment, **kwargs):
    print("A new test is starting")

@events.test_stop.add_listener
def on_test_stop(environment, **kwargs):
    print("A new test is ending")
```

#### init事件

init事件在每个蝗虫进程开始时触发。这在分布式模式下特别有用，因为每个工作进程(而不是每个用户)都需要有机会进行一些初始化。例如，假设你有一些全局状态，所有从这个进程中生成的用户都需要:

### client属性

client是HttpSession的一个实例。HttpSession是请求的子类/包装器。会话，所以它的特性有很好的文档记录，应该为许多人所熟悉。HttpSession添加的内容主要是向Locust报告请求结果(成功/失败、响应时间、响应长度、名称)。

就像[`requests.Session`](https://requests.readthedocs.io/en/latest/api/#requests.Session)一样。会话，它保留请求之间的cookie，因此它可以很容易地用于登录网站。

HttpSession捕获任何请求。由Session抛出的RequestException(由连接错误，超时或类似原因引起)，而不是返回一个虚拟的Response对象，status_code设置为0，内容设置为None。

### 校验响应

默认判断响应是否正确是通过响应码是否<400进行判断，此外也可以自己定义额外的响应校验规则。

可以通过使用catch_response参数、with语句和调用response.failure()将请求标记为失败。

```python
with self.client.get("/", catch_response=True) as response:
    if response.text != "Success":
        response.failure("Got wrong response")
    elif response.elapsed.total_seconds() > 0.5:
        response.failure("Request took too long")
```

甚至可以通过抛出一个异常，然后在with-block之外捕获它，从而完全避免记录请求。或者您可以抛出一个locust异常，就像下面的例子一样，让locust捕获它。

```python
from locust.exception import RescheduleTask
...
with self.client.get("/does_not_exist/", catch_response=True) as response:
    if response.status_code == 404:
        raise RescheduleTask()
```

### 连接池

如果要在所有用户之间共享连接，则可以使用连接池管理器。将pool_manager属性设置为`urllib3.PoolManager`的实例。`

```
from locust import HttpUser
from urllib3 import PoolManager

class MyUser(HttpUser):
    # All users will be limited to 10 concurrent connections at most.
    pool_manager = PoolManager(maxsize=10, block=True)
```

### 安全模式

HTTP客户端配置为以safe_mode运行。这样做的目的是，由于连接错误、超时或类似原因而失败的任何请求都不会引发异常，而是返回一个空的虚拟Response对象。该请求将在Locust的统计信息中标记为失败。返回的虚拟Response的content属性将设置为None，其status_code=0。

### TaskSequence类

TaskSequence类是一个TaskSet，但是它的任务是按顺序执行的。
要定义此顺序，您应该执行以下操作:

```
@seq_task(1)
def first_task(self):
    pass

@seq_task(2)
def second_task(self):
    pass

@seq_task(3)
@task(10)
def third_task(self):
    pass
```

复制
在上面的示例中，执行顺序定义为先执行first_task，然后执行second_task，最后执行Third_task 10次。可以看到，可以使用@task装饰器组合@seq_task，当然也可以将TaskSet嵌套在TaskSequences中，反之亦然。



