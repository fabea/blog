---
title: "3月7日Flask学习记录"
date: 2021-03-07T09:33:27+08:00
draft: false
toc: true
images:
tags: 
  - learning
---

# 应用工厂

## why?
应用工厂函数和蓝图的模块化结合之后，会让程序更加的有条理。不会出现所有的东西都在一个`app.py`这种情况。
当然**优点缺点**都有。

**优点**：通过使用工厂函数，可以轻松获得多个实例。同时也可以满足不同的测试需求。
```python
def create_app(config_filename):
    app = Flask(__name__)
    app.config.from_pyfile(config_filename)

    from yourapplication.model import db
    db.init_app(app)

    from yourapplication.views.admin import admin
    from yourapplication.views.frontend import frontend
    app.register_blueprint(admin)
    app.register_blueprint(frontend)

    return app
```
但是**缺点**是，每一个蓝图之中，没有办法直接使用这个app，因此，Flask提供了`current_app`来解决这个问题。

蓝图之中，需要调用app本身的时候，可以这么用。

```python
from flask import current_app, Blueprint, render_template
admin = Blueprint('admin', __name__, url_prefix='/admin')

@admin.route('/')
def index():
    return render_template(current_app.config['INDEX_TEMPLATE'])
```

## 工厂和扩展
不要将数据库等扩展，直接在工厂里实例化，否则会过早的出现在应用中。
应该将其分开，例如，在单独的`model.py`中：

```python
db = SQLAlchemy()
```
并把他封装在`init_app()`函数中。

然后在主程序`app.py`中，引入和使用这个`init_app()`函数。

```python
def create_app(config_filename):
    app = Flask(__name__)
    app.config.from_pyfile(config_filename)

    from yourapplication.model import db
    db.init_app(app) # SQLAlchemy用法。
```

## 应用工厂的使用
```bash
$ export FLASK_APP=myapp
$ flask run
```

# 应用调度 Application Dispatch

## WSGI层面调度

只能用于非生产环境：

`werkzeug.serving.run_simple()`可以用来运行app。

下面是一个最简单的app。

```python
from flask import Flask
from werkzeug.serving import run_simple

app = Flask(__name__)
app.debug = True

@app.route('/')
def hello_world():
    return 'Hello World!'

if __name__ == '__main__':
    run_simple('localhost', 5000, app,
               use_reloader=True, use_debugger=True, use_evalex=True)
```

## 多应用独立运行。

直接看例子吧：
```python
from flask import Flask
from werkzeug.middleware.dispatcher import DispatcherMiddleware
from werkzeug.serving import run_simple

main_app = Flask(__name__)
test1 = Flask(__name__)
test2 = Flask(__name__)


@main_app.route('/')
def hello_world():
    return 'Hello World from main app!'


@test1.route('/')
def hello_world_test1():
    return 'Hello World from test1!'


@test2.route('/')
def hello_world_test2():
    return 'Hello World from test2!'


app = DispatcherMiddleware(main_app, {'/test1': test1, '/test2': test2})

if __name__ == "__main__":
    run_simple('0.0.0.0', 5000, app, use_reloader=True, use_debugger=True)

```


# 额外的异常
内建的异常往往不能满足需求，因此可能需要创建自己的异常。

```python
from flask import jsonify

class MyOwnException(Exception):
    status_code=400

    def __init__(self,message, status_code=None,payload=None):
        Exception.__init__(self)
        self.message=message
        if status_code not None:
          self.status_code=status_code
        self.payload=payload
      
    def to_dict(self):
        rv = dict(self.payload or ())
        rv['message'] =self.message
        return rv

# 注册这个异常

@app.errorhandler(MyOwnException)
def handle_my_own_exception(error):
    response = jsonify(error.to_dict())
    response.status_code=error.status_code
    return response

# 使用这个异常

@app.route('/error')
def error():
    raise MyOwnException('This is a very unique exception!', status_code=409)

```


# SQLAlchemy

SQLA有四种常用的方法

## F-SQLA
最推荐的还是使用这个扩展，这个今天稍晚，会读完F-SQLA的文档。
## 声明
SQLAlchemy 中的声明扩展是使用 SQLAlchemy 的最新方法，在一个地方定义表和模型然后到处使用。
首先当然是声明：
```python
from sqlalchemy import create_engine
from sqlalchemy.orm import scoped_session, sessionmaker
from sqlalchemy.ext.declarative import declarative_base

engine = create_engine('sqlite:////tmp/test.db', convert_unicode=True)
db_session = scoped_session(sessionmaker(autocommit=False,
                                         autoflush=False,
                                         bind=engine))
Base = declarative_base()
Base.query = db_session.query_property()

def init_db():
    # 在这里导入定义模型所需要的所有模块，这样它们就会正确的注册在
    # 元数据上。否则你就必须在调用 init_db() 之前导入它们。
    import yourapplication.models
    Base.metadata.create_all(bind=engine)
```
然后要让应用知道什么时候可以移除数据库会话。

```python
from yourapplication.database import db_session

@app.teardown_appcontext
def shutdown_session(exception=None):
    db_session.remove()
```

接下来，我们需要把模型做出来。

```python
from sqlalchemy import Column, Integer, String
from yourapplication.database import Base

class User(Base): #这里继承了Base
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True)
    name = Column(String(50), unique=True)
    email = Column(String(120), unique=True)

    def __init__(self, name=None, email=None):
        self.name = name
        self.email = email

    def __repr__(self):
        return '<User %r>' % (self.name)

```
创建数据库

```python
from yourapplication.database import init_db
init_db()
```

CRUD就很简单了。
```bash
>>> from yourapplication.database import db_session
>>> from yourapplication.models import User
>>> u = User('admin', 'admin@localhost')
>>> db_session.add(u)
>>> db_session.commit()

>>> User.query.all()
[<User u'admin'>]
>>> User.query.filter(User.name == 'admin').first()
<User u'admin'>
```

## 人工对象关系映射
相对来讲，我更熟悉也更习惯这种用法。整体而言，大同小异。

```python
'''
首先当然是建立session。
'''
from sqlalchemy import create_engine, MetaData
from sqlalchemy.orm import scoped_session, sessionmaker

engine = create_engine('sqlite:////tmp/test.db', convert_unicode=True)
metadata = MetaData()
db_session = scoped_session(sessionmaker(autocommit=False,
                                         autoflush=False,
                                         bind=engine))
def init_db():
    metadata.create_all(bind=engine)


'''
这一步是一样的。
'''
from yourapplication.database import db_session

@app.teardown_appcontext
def shutdown_session(exception=None):
    db_session.remove()

'''
不同之处在于这里。
'''
from sqlalchemy import Table, Column, Integer, String
from sqlalchemy.orm import mapper
from yourapplication.database import metadata, db_session

class User(object):
    query = db_session.query_property()

    def __init__(self, name=None, email=None):
        self.name = name
        self.email = email

    def __repr__(self):
        return '<User %r>' % (self.name)

users = Table('users', metadata,
    Column('id', Integer, primary_key=True),
    Column('name', String(50), unique=True),
    Column('email', String(120), unique=True)
)
mapper(User, users)
'''
CRUD就都一样了。
'''
```

## SQL 抽象层
最简单，也最直接。
```python
'''
只是用引擎就可以了。
'''
from sqlalchemy import create_engine, MetaData, Table

engine = create_engine('sqlite:////tmp/test.db', convert_unicode=True)
metadata = MetaData(bind=engine)

'''
声明表
'''
from sqlalchemy import Table

users = Table('users', metadata, autoload=True)

'''
CRUD
'''
con = engine.connect()
con.execute(users.insert(), name='admin', email='admin@localhost')

users.select(users.c.id == 1).execute().first()

r = users.select(users.c.id == 1).execute().first()
r['name']

 engine.execute('select * from users where id = :1', [1]).first()
```

总之，最好的办法还是使用扩展。因为扩展最简单，抽象的程度最高。