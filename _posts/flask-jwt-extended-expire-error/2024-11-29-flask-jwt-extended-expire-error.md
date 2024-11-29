---
title: 当 Flask-JWT-Extended 遇到 ExpiredSignatureError
date: 2024-11-29 00:00:00 +08:00
modified: 2024-11-29 00:00:00 +08:00
tags: [Python, Flask, JWT, Flask-JWT-Extended]
description: "最近我们在使用 Flask 开发后端接口的时候，遇到一个奇怪的问题。它的具体表现是：在默认的 Debug 环境下运行服务时，如果请求头带有一个已经过期的 Access Token 它能正常返回预期的 401 错误。但是一旦上线到线上环境（基于 uWSGI + Nginx），接口就会返回 500 错误，后台记录错误信息是 `jwt.exceptions.ExpiredSignatureError: Signature has expired`。经过一番排查，找到了问题的原因。"
---
最近我们在使用 Flask 开发后端接口的时候，遇到一个奇怪的问题。它的具体表现是：在默认的 Debug 环境下运行服务时，如果请求头带有一个已经过期的 Access Token 它能正常返回预期的 401 错误。但是一旦上线到线上环境（基于 uWSGI + Nginx），接口就会返回 500 错误，后台记录错误信息是 `jwt.exceptions.ExpiredSignatureError: Signature has expired`。经过一番排查，找到了问题的原因。

在 Stack Overflow 上有人提出了跟我们遇到的[完全一样的问题](https://stackoverflow.com/questions/56281886/api-with-flask-jwt-extended-with-authentication-problems)。在这个提问下，有人引用了一个来自 Flask-JWT-Extended 官方代码仓库的 [Issue](https://github.com/vimalloc/flask-jwt-extended/issues/20)。

提出这个 Issue 的人也是遇到了完全一样的问题，而且描述得更详细。

对于样例代码：
```python
'''In app.py'''
from flask_jwt_extended import JWTManager
from flask import Flask

jwtmanager = JWTManager()

def create_app():
    app = Flask(__name__)

def register_extensions(app):
    jwtmanager.init_app(app)


'''in views.py'''
from flask_restful import Resource, Api
from flask_jwt_extended import jwt_required

api = Api()

class TokenResource(Resource):
    method_decorators = [jwt_required]

class HelloWorld(TokenResource):
    def get(self):
        return 'Hello, World!'

api.add_resource(HelloWorld, '/hello')
```

该接口运行在调试模式下，可以正常返回预期的结果:
```json
{
  "msg": "Token has expired"
}
```

但是如果使用 gunicorn 启动服务，则会返回 Internal Server Error:

```json
{
    "message": "Internal Server Error"
}
```

在 Issue 的讨论中 Flask-JWT-Extended 项目的发起人给出了解决的方案，即在 Flask 的配置文件里增加一个配置：
```python
PROPAGATE_EXCEPTIONS = True
```

这个问题同时也存在于一个已经被归档的项目 [flask-jwt](https://github.com/pallets-eco/flask-jwt)，并在  [flask-jwt/issues/74](https://github.com/pallets-eco/flask-jwt/issues/74) 被提出过。

至于安全性问题，Flask-JWT-Extended 的作者认为这个设置并不会引起内部代码的对外泄露风险。并举了例子：

```python
from flask import Flask, jsonify
from flask_jwt_extended import  JWTManager, jwt_required
from flask_jwt_extended.exceptions import NoAuthorizationError

app = Flask(__name__)

# Setup the Flask-JWT-Extended extension
app.config['JWT_SECRET_KEY'] = 'super-secret'
app.config['PROPAGATE_EXCEPTIONS'] = True
jwt = JWTManager(app)

@app.route('/login', methods=['POST'])
def login():
    raise ValueError("exception without errorhandler")

@app.route('/protected', methods=['GET'])
def protected():
    raise NoAuthorizationError("exception with errorhandler")

if __name__ == '__main__':
    app.run()
```

```
$ http POST :5000/login
HTTP/1.0 500 INTERNAL SERVER ERROR
Content-Length: 291
Content-Type: text/html
Date: Sun, 24 Jun 2018 03:38:22 GMT
Server: Werkzeug/0.14.1 Python/3.6.4

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<title>500 Internal Server Error</title>
<h1>Internal Server Error</h1>
<p>The server encountered an internal error and was unable to complete your request.  Either the server is overloaded or there is an error in the application.</p>


$ http :5000/protected
HTTP/1.0 401 UNAUTHORIZED
Content-Length: 38
Content-Type: application/json
Date: Sun, 24 Jun 2018 03:38:31 GMT
Server: Werkzeug/0.14.1 Python/3.6.4

{
    "msg": "exception with errorhandler"
}
```

关于 Flask 的 `PROPAGATE_EXCEPTIONS` 配置的作用，根据[官方文档](https://flask.palletsprojects.com/en/stable/config/#PROPAGATE_EXCEPTIONS)的描述：
> Exceptions are re-raised rather than being handled by the app's error handlers. If not set, this is implicitly true if `TESTING` or `DEBUG` is enabled.

根据描述，这个参数是用于控制异常是否重新抛出，而不是被应用的错误处理器处理。如果没有设置，当 `TESTING` 或 `DEBUG` 启用时，这个参数会被隐式设置为 `True`。这也就是为什么在调试模式下可以正常返回 401 错误，而在生产环境下会返回 500 错误的原因。

所以在使用 Flask-JWT-Extended 的时候，应该显性地在 Flask 的生产环境配置文件里设置 `PROPAGATE_EXCEPTIONS = True` 才能充分利用插件对异常的再处理能力。
