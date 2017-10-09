---
title: 解决flask_restful无法使用@app.errorhandler
date: 2016-10-22 14:01:08
tags:
---
在使用`flask_restful`插件开发RESTfulAPI的时候，会发现无法在非debug模式下使用flask的`@app.errorhandler`装饰器来捕获处理自己定义自己的异常，这是一件比较让人头痛的事情。经过对restful插件的源码阅读之后，想到了一种解决这个问题的办法。便是定义一个装饰器，用于捕获处理自定义异常，然后在每一个`get/post`等方法上注册这个装饰器。
先放上部分异常声明代码：

```python
class ASError(Exception):

    def __init__(self, status, code, message):
        self._status = status
        self._code = code
        self._message = message

    @property
    def status(self):
        return self._status

    @property
    def code(self):
        return self._code

    @property
    def message(self):
        return self._message

class ASBadRequest(ASError):

    def __init__(self, code, message):
        super(ASBadRequest, self).__init__(400, code, message)

class ASArgMissing(ASBadRequest):

    def __init__(self, msg):
        super(ASArgMissing, self).__init__(_ARG_MISSING, msg)


class ASArgFormat(ASBadRequest):

    def __init__(self, msg):
        super(ASArgFormat, self).__init__(_ARG_FORMART, msg)
...
```

然后是异常处理的装饰器代码:

```python
def errorhandler(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        try:
            resp = func(*args, **kwargs)
            return resp
        except ASError as err:
            status = err.status()
            respMsg = jsonify({'status': err.code(), 'message': err.message()})
            return make_response(respMsg, status)
    return wrapper
```

然后是在`get/post`等方法上注册这个装饰器，这里当然不会在每一个方法上使用@errohandler,这样子太丑了。比较好的做法是这样子, 向api的decorators属性传入一个装饰器的列表，列表中的所有装饰器会作用到每一个`get/post`等方法上去。

```python
app = Flask(__name__)
api = Api(app, decorators=[errorhandler])
```

这样就可以避免使用`@app.errorhandler`,而导致在非debug模式下，异常无法被捕获。最后附上[github](https://github.com/DashShen/flask-restful-template)地址。