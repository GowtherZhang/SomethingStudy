[TOC]

# 1. 背景

平时处理数据时，将大批量的数据处理导入数据库常常会花费大量的时间，希望通过 SqlAlchemy 提高传表效率，对 SqlAlchemy 进行系统性的学习。

# 2. SqlAlchemy 简介

* 是什么？

  SqlAlchemy 是链接数据库的一种方式，经过驱动（pymysql 包）来连接数据库。它是 python 中使用最普遍的对象关系映射（Object Relation Mapper）工具之一，可以直接使用编程语言的模型来操作对象，而不是通过 Mysql 语句。

  SQLAlchemy 分为两个主要部分：ORM 对象映射和核心的 SQLexpression。其中，ORM对象映射负责将Python类映射到关系型数据库中的表，而核心的 SQLexpression 则提供了一种使用SQL表达式来操作关系型数据库的方法 。

  所以在运行时，ORM 把表映射成类，把行作为实例，把字段作为属性；在执行对象操作时，会自动把操作转化为数据库的原生语句。

* 优点

​	易用性：减少 sql 语句的使用，使得代码模型更直观、清晰

​	性能损耗小：设计灵活、可移植性强

* 缺点：

  在处理多表查询时，ORM 的语句会变得复杂

# 3. SqlAlchemy 的安装与连接

## 3.1 安装 

由于 mysql 不支持 Python3，所以需要先安装驱动，即 PyMysql 与 SqlAlchemy 交互。

```pyton
pip install pymysql
pip install sqlalchemy
```

## 3.2 连接数据库

通过，sqlalchemy.create_engine 连接数据库，引擎参数如下：

```python
engine = create_engine("数据库类型+数据库驱动://数据库用户名:数据库密码@IP地址:端口号/数据库?编码...", 其它参数)
```

以连接本地数据库为例，其中 ：

* echo：用来设置数据库日志，为 True 时，可以看到所有数据库的操作记录
* create_engine：返回的是一个 engine 实例，代表了操作数据库的核心接口，用来处理数据库和数据库的 API。第一次调用 create_eigine() 时，只是创建了连接数据库的引擎，而不会连接数据库，只有在利用引擎执行查询的时候，才会连接数据库，这样可以节省资源。

```python
engine = create_engine("mysql+pymysql://root:123456@localhost:3306/database", echo=True)
```

## 3.3 映射声明

当使用 ORM 时，其配置过程主要分为两部分：一部分用来处理数据库表的数据，另一部分是将 Python 的类映射到这些表中，这个过程在 SqlAlchemy  中一起完成，过程称之为 Declarative。

使用 Declarative 参与的 ORM 映射的类需要被指定为一个基类的子类，这个基类包含了 ORM 映射中相关类和表的信息，这个基类被称之为 declarative base class，可以通过 declarative_base 来创建。

```python
from sqlalchemy.ext.declarative import declarative_bae
Base = declarative_base()
```

# 4. SqlAlchemy 常用的数据类型

| 类型     | 含义         | Note                                                       |
| -------- | ------------ | ---------------------------------------------------------- |
| Integer  | 整数         | 即数据库中的Int                                            |
| Float    | 浮点数       |                                                            |
| String   |              |                                                            |
| Double   |              |                                                            |
| Boolean  |              |                                                            |
| Decimal  | 定点类型     | 需要传入两个参数，参数 1 决定总位数，参数 2 决定小数的位数 |
| Enum     | 枚举类型     |                                                            |
| Date     | 日期类型     | 年月日                                                     |
| DateTime | 日期时间类型 | 年月日时分秒                                               |
| Time     | 时间类型     | 时分秒                                                     |
| Text     | 文本类型     | 最多存储的数据长度为 64K                                   |
| LongText | 长文本       |                                                            |

# 5. 创建类

基于基类，使用常见的数据类型创建一个自定义的类。

## 5.1 创建一个用户类

User 类继承了基类；

\__tablename__ 指明了表的名字；

通过 Column 设置了列的属性 ；

通过以上对类的说明，将类映射成能够读写数据库的表和列。

## 5.2 类返回字符串

函数 repr 定义该类返回的字符串内容。

```python
class User(Base):
    __tablename__ = "user"
    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(20), default=None, nullable=False, comment='用户姓名', unique=False)
    phone = Column(String, default=None, nullable=True, comment="用户电话")
    weight = Column(Float, default=50, nullable=False, comment="用户体重" )

    def _repr_(self):
        Name = self.name
        Phone = self.phone
        Weight = self.weight
        return f"User: name: {Name}, phone: {Phone}, weight: {Weight}"
```

# 6. 创建模式

## 6.1 查看表的类型

```python
User.__table__

# 结果如下：
TABLE (
	'user',
	MetaData (),
	COLUMN ( 'id', INTEGER (), TABLE =< USER >, primary_key = TRUE, nullable = FALSE ),
	COLUMN ( 'name', String ( length = 20 ), TABLE =< USER >, nullable = FALSE, COMMENT = '用户姓名' ),
	COLUMN ( 'phone', String (), TABLE =< USER >, COMMENT = '用户电话' ),
	COLUMN ( 'weight', FLOAT (), TABLE =< USER >, nullable = FALSE, DEFAULT = ScalarElementColumnDefault ( 50 ), COMMENT = '用户体重' ),
SCHEMA = NONE)
```

## 6.2 创建表

Table 对象是一个更大家庭——metadata，metadata 是与数据打交道的一个接口，创建表需要 metadata 发出 create table 的命令，完整的创建表的脚本如下：

```python
class User(Base):
    __tablename__ = "user"
    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(20), default=None, nullable=False, comment='用户姓名', unique=False)
    phone = Column(String, default=None, nullable=True, comment="用户电话")
    weight = Column(Float, default=50, nullable=False, comment="用户体重" )

    def _repr_(self):
	    Name = self.name
        Phone = self.phone
        Weight = self.weight
        return f"User: name: {Name}, phone: {Phone}, weight: {Weight}"
   
Base.metadata.create_all() #通过此语句创建表
```

创建后终端显示如下图：![image-20230910151520890](C:\Users\18098\AppData\Roaming\Typora\typora-user-images\image-20230910151520890.png)

## 6.3 创建实例

使用 User 类创建一个新的实例

```python
NewUser = User(name='Gowther', phone='123456', weight=55.55)
```

## 6.4 创建会话

要真正使用类对象操作数据库，还需要一个 Session 对象，ORM 对数据库的入口即 Session，Session 实例的创建如下：

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
engine = create_engine("mysql+pymysql://root:123456@localhost:3306/spiders", echo=True)
Session = sessionmaker(bind=engine)
session = Session()
```

sqlalchemy 中的 Session 提供了各种操作数据库的方法，包括：add(), query(), add_all(), commit(), delete(), flush(), rollback(), close()等。

# 7. SqlAlchemy 对 Mysql 数据库的基本操作

## 7.1 添加对象

sqlalchemy 可以通过 add(), add_all() 方法将数据添加到数据库，当方法为 add_all()，参数为一个列表。

```python
# add() 添加单个实例
NewUser = User(name='Gowther', phone='123456', weight=55.55)
session.add(NewUser)
session.commit()

# add_all() 添加多个实例
SomeUsers = [User(name='Holf',phone='11111', weight=50.01), User(name='Julie', phone='22222222', weight=39.99)]
session.add_all(SomeUsers)
session.commit()
```

操作结果截图如下：

![image-20230910153506201](C:\Users\18098\AppData\Roaming\Typora\typora-user-images\image-20230910153506201.png)

![image-20230910154245493](C:\Users\18098\AppData\Roaming\Typora\typora-user-images\image-20230910154245493.png)

## 7.2 查询对象	

* 整表查询

sqlalchemy 中使用 query() 方法查询数据，返回函数__repr__中返回的值，脚本与操作截图如下： 

```python
result = session.query(User).all()

print(f"{'name':15} | {'phone':15} | {'weight':15}")

for i in result:
    print(f"{i.name:15} | {i.phone:15} | {i.weight:<15}")
```

![image-20230910163203055](C:\Users\18098\AppData\Roaming\Typora\typora-user-images\image-20230910163203055.png)

* 条件查询

  sqlalchemy 使用 filter 进行条件查询，以下为脚本与操作截图：

  ```python
  # 设置一个不显示操作日志的接口，让输出更整洁
  engine2 = create_engine("mysql+pymysql://root:123456@localhost:3306/spiders", echo=False)
  Session2 = sessionmaker(bind=engine2)
  session2 = Session2()
  
  # 添加filter条件
  result = session2.query(User).filter(User.name!='Gowther')
  print(f"{'name':^15} | {'phone':^15} | {'weight':^15}")
  for i in result:
      print(f"{i.name:^15} | {i.phone:^15} | {i.weight:^15}")
  ```

  ![image-20230910164220129](C:\Users\18098\AppData\Roaming\Typora\typora-user-images\image-20230910164220129.png)

## 7.3 更新对象

sqlalchemy 中实现 update 的过程是，先通过 query 查询要更新的目标，并将目标映射到类的属性字段上，然后对属性字段进行赋值更新，最后使用 add(), commit() 方法实现更新。

```python
result = session.query(User).filter(User.name=='Gowther').first()
result.weight = 888
session.add(result)
session.commit()
```

但是要注意的是如果有两个会话连接数据库时，Session1 对数据库提交的更改，使用 Session2 进行查询，结果并不能得到该变化。这是因为 SQLAlchemy 的 Session 对象是一个事务（Transaction) 级别的，他跟数据库连接是绑定的，当你使用 Session2 进行查询时，他创建了一个新的数据库连接，不会受之前 Session1 对数据库进行的更新操作的影响。如果希望 Session2 也能看到 Session 的修改，可以在 Session1 中提交事务后再创建 Session2。示例如下：

```python
result = session.query(User).filter(User.name=='Gowther').first()
result.weight = 888
session.add(result)
session.commit()

result = session.query(User).filter(User.name=='Gowther').all()
for i in result:
    print(i.name, i.weight)
    
#输出
Gowther 888.0
Gowther 55.55
Gowther 55.55
Gowther 55.55
Gowther 55.55

result = session2.query(User).filter(User.name=='Gowther').all()
for i in result:
    print(i.name, i.weight)
    
#输出
Gowther 999.0
Gowther 55.55
Gowther 55.55
Gowther 55.55
Gowther 55.55
```

## 7.4 删除对象

SqlAlchemy 中使用 delete 的方法删除对象。

```python
result = session.query(User).filter(User.name=='Gowther').first()
session.delete(result)
session.commit()

result = session.query(User).filter(User.name=='Gowther').all()
for i in result:
    print(f'name: {i.name}, phone: {i.phone}, weight: {i.weight}')
    
#输出
Gowther 55.55
Gowther 55.55
Gowther 55.55
Gowther 55.55
```

## 7.5 高级查询

* and 操作符

  共有 3 种方法，传递多个关键词给 filter，使用多个 filter 函数，或者使用 and 函数传递多个表达式给 filter 方法。第一种更为简洁，而第三种更明确的显示了条件之间的逻辑关系。

  ```python
  # and 运算符
  result1 = session.query(User).filter(User.id > 10 , User.name == 'Gowther').first()
  print(result1.id, result1.name)
  
  result2 = session.query(User).filter(User.id > 10).filter(User.name == 'Gowther').first()
  print(result1.id, result1.name)
  
  from sqlalchemy import and_
  result3 = session.query(User).filter(and_(User.id > 10, User.name =='Gowther')).first()
  print(result3.id, result3.name)
  ```

* or 操作符

  使用 or 函数，将多个条件表达式传递给 filter 方法

  ````python
  from sqlalchemy import or_
  result4 = session.query(User).filter(or_(User.id > 10, User.name =='Gowther')).first()
  print(result4.id, result4.name)
  ````

* Like 操作符

  与 sql 中一致，直接使用 like 函数匹配即可，但是注意，like 后跟的匹配表达式要带括号

  ```python
  result5 = session.query(User).filter(User.name.like("%wt%")).first()
  print(result5.id, result5.name)
  ```

* in 操作符

  使用 in_ 函数进行匹配，注意使用 in_ 函数的时候，参数应该是一个列表

  ```python
  result6 = session.query(User).filter(User.name.in_(['Gowther', 'Holf'])).all()
  for i in result6:
      print(i.id, i.name)
  ```

* not in 操作符

  在 in 操作符的基础上加上运算符 "~"

  ```python
  result7 = session.query(User).filter(~User.name.in_(['Gowther', 'Holf'])).all()
  for i in result7:
      print(i.id, i.name)
  ```

# 8. 嵌入使用 Sql 语句

Sqlalchemy 中有 Text 方法，可以通过使用 text(SQL语句) 嵌入使用 SQL 语句。

## 8.1 在查询中嵌入 SQL 语句

```python
from sqlalchemy import text

result = session.query(User).filter(text("name = 'Gowther'")).first()
print(result.name, result.weight)

result = session.query(User).filter(text("name like '%he%'")).first()
print(result.name, result.weight)
```

## 8.2 通过 Engine 对象执行 SQL语句

```python
sql = 'select * from user where id in (1, 3, 5, 7, 9);'

with engine.connect() as conn:
    result = conn.execute(text(sql))

for i in result:
    print(i.name, i.weight)
```

## 8.3 保存/更新用户信息示例



