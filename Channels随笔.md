#  Channels随笔



##  Consumers模块

+ consumers == 消费者

Consumers通常要做两件事

+ 将代码结构化为在事件发生时要调用的一系列函数，而不是使您编写事件循环。

- 允许您编写同步或异步代码，并为您处理切换和线程。

基本例子

+ SyncConsumer

```python
from channels.consumer import SyncConsumer

class EchoConsumer(SyncConsumer):

    def websocket_connect(self, event):
        self.send({
            "type": "websocket.accept",
        })

    def websocket_receive(self, event):
        self.send({
            "type": "websocket.send",
            "text": event["text"],
        })
```

+ AsyncConsumer

```python
from channels.consumer import AsyncConsumer

class EchoConsumer(AsyncConsumer):

    async def websocket_connect(self, event):
        await self.send({
            "type": "websocket.accept",
        })

    async def websocket_receive(self, event):
        await self.send({
            "type": "websocket.send",
            "text": event["text"],
        })
```

**Scope**

关于WebSocket的方法

+ scope['headers']   WebSocket协议的头部
+ scope['path']    请求的路径



## Routing 模块

+ 代码示例

```python
import os

from channels.auth import AuthMiddlewareStack
from channels.routing import ProtocolTypeRouter, URLRouter
from django.conf.urls import url
from django.core.asgi import get_asgi_application

from chat.consumers import AdminChatConsumer, PublicChatConsumer

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "django_channels.settings")

application = ProtocolTypeRouter({
    # Django's ASGI application to handle traditional HTTP requests
    "http": get_asgi_application() # HTTP请求

    # WebSocket chat handler
    "websocket": AuthMiddlewareStack(
        URLRouter([
            url(r"^chat/admin/$", AdminChatConsumer.as_asgi()),
            url(r"^chat/$", PublicChatConsumer.as_asgi()),
        ])
    ),
})

Django2.2无法使用get_asgi_alpplication()方法
```

+ 如何获得URL中的命名组

```
name = self.scope['url_route']['kwargs']['name']
```



##  Datebase Access 模块

+ 注意一点，数据库操作使用异步，如果要从异步代码访问它，则需要进行特殊处理以确保其连接正确关闭，否则可能出现数据混乱
+ **database_sync_to_async**

```python
from channels.db import database_sync_to_async

async def connect(self):
    self.username = await database_sync_to_async(self.get_name)()

def get_name(self):
    return User.objects.all()[0].name
```

+ **database_sync_to_async用作装饰器**

```python
from channels.db import database_sync_to_async

async def connect(self):
    self.username = await self.get_name()

@database_sync_to_async
def get_name(self):
    return User.objects.all()[0].name
```



##  Channel Layers模块

(通道层模块)

```
django.settings中设置
	CHANNEL_LAYERS = {}
	就可以设置Django通道层模块
```

+ Channel配置Redis

```python
CHANNEL_LAYERS = {
    "default": {
        "BACKEND": "channels_redis.core.RedisChannelLayer",
        "CONFIG": {
            "hosts": [("127.0.0.1", 6379)],
        },
    },
}
```

+ Channel配置内存

```python
CHANNEL_LAYERS = {
    "default": {
        "BACKEND": "channels.layers.InMemoryChannelLayer"
    }
}
```

+ 同步功能

默认情况下`send()`，`group_send()`，`group_add()`等功能异步功能，这意味着你要`await`他们。如果需要从同步代码中调用它们，则需要使用方便的 `asgiref.sync.async_to_sync`包装器：

```python
from asgiref.sync import async_to_sync

async_to_sync(channel_layer.send)("channel_name", {...})
```

