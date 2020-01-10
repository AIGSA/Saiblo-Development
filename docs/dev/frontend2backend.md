# 后端与前端

整个 ***Saiblo*** 网站包括三个部分，即**前端**、**后端**与**评测端**。这个文档主要讲的是**后端**与**前端**的交互。后端与前端是网站的重要组成部分，前端需要从后端拉取数据并进行显示，用户也需要从前端输入数据录入后端。为此，我们设计了一系列的前后端交互的 API ，以下将分模块来介绍后端的各个模块的 API 及其功能。

前端要使用这些 API ，一般是通过`axios.post[put/get/...](URL, data)`来进行操作的。例如`axios.post('token/', {username: a, password: b})`。

以下对介绍 API 时所列举的各项**属性**进行说明。

- **post/put/get/...方法**：指明使用`axios`时所使用的方法。
- **权限**：指明使用这个 API 时，用户所需要具有的特征和属性，即用户权限。便于前端对用户操作进行管理限制，同时后端也会判断用户是否能够使用此 API。
- **数据**：指明使用这个 API 时，前端所需要向后端发送的信息。
- **响应**：指明使用这个 API 时，后端会向前端发回的返回代码以及返回代码的意义。便于前端对这些响应代码进行区分与处理。
- **返回**：指明如果正确调用了这个 API，并且正常返回时，后端返回给前端的数据。

## 用户模块

用户模块的功能是控制用户的**注册**、**登陆**、**注销**；或是**获取**、**修改**用户的信息。

### 1 URL: token/

#### 1.1 POST方法

**用于用户登陆**。验证身份信息，通过则获取新发放的`token`。用户的所有操作可以通过`token`进行身份验证。

##### 1.1.1 权限

- 任何人

##### 1.1.2 数据

```json
{
    username: String, //表示用户名
    password: String //表示密码
}
```

##### 1.1.3 响应

- 200：登录成功，获取到`token={"access": \<access>, "refresh": \<refresh>}`
- 401：用户名或密码错误

##### 1.1.4 返回

```python
{
    'refresh': # refresh token
    'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0b2tlbl90eXBlIjoicmVmcmVzaCIsImV4cCI6MTU3MTk4MDk5OCwianRpIjoiM2UwNzU0YWU2NmQ2NDZkYWJmZDc4OWE4YjQ1YjNjOGYiLCJ1c2VyX2lkIjoyfQ.YIEn2aJ9zkzjgpw0aN03VIHCZAmgpvohMkMAmMeQi9U',
    'access': # access token
    'eyJ0eXA8iOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIiwiZXhwIjoxNTcxODk4MTk4LCJqdGkiOiIwODZiMGIxMDVkMjQ0NGVhODNlMTg0NTIyYzc1YzEwZiIsInVzZXJfaWQiOjJ9.RVxrGW5b24JpCnLSJhkyONixMllJZOIe4Hj86TpCBp',
}
```

### 2 URL: refresh/

#### 2.1 POST方法

更新用户的`token`。

##### 2.1.1 权限

- 已登录用户，即用户状态为“已登录”。

##### 2.1.2 数据

```json
{
    refresh：String //登录时所获取的refresh token
}
```

##### 2.1.3 响应

- 200：获取到新的token

##### 2.1.4 返回

```python
{
    'access': # new access token
    'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIiwiZXhwIjoxNTcxODk4NDA3LCJqdGkiOiJmMjE1Y2ZiMzhjMTA0MDNlYTQ2ZTkwMjNhNGRiZDA2MSIsInVzZXJfaWQiOjJ9.QS4-AKNhnEEzlzCKJ2clFsjHzb5hSAp4Uk1waLyraRg'
}
```

### 3 URL: profile/

#### 3.1 GET方法

获取当前登录用户的个人信息，可用于判断是否登录。

##### 3.1.1 权限

- 已登录用户，即用户状态为“已登录”。

##### 3.1.2 响应

- 200：获取成功

##### 3.1.3 返回

注：此处计划将来加入一些仅自己可见的私人信息项，现在暂且没加入。

```python
{
    'user': {
        'name': '叶良辰',
        'biography': '后会有期',
        'school': '',
        'department': '',
        'sex': '男',
        'username': 'moon',
        'email': 'moon@saiblo.com',
        'email_md5': '496f4e4ae237bc3107d31400e5b61257',
        'date_joined': 'Mon, 14 Oct 2019 14:57:44 +0800',
        'id': 2,
        'is_active': True
    }
}
```

### 4 URL: users/

#### 4.1 GET方法

返回所有注册过的用户信息。

##### 4.1.1 权限

- 任何人

##### 4.1.2 响应

- 200：获取成功

##### 4.1.3 返回

```python
[
    {
        'name': '网管',
        'biography': '',
        'school': '',
        'department': '',
        'sex': '保密',
        'username': 'admin',
        'email': 'admin@saiblo.com',
        'email_md5': 'c6c7721cf2ddbb4f8576620e394434d2',
        'date_joined': 'Mon, 14 Oct 2019 14:56:36 +0800',
        'id': 1,
        'is_active': True
    },
    ...
]
```

#### 4.2 POST方法

注册一个新的用户。

##### 4.2.1 权限

- 任何人

##### 4.2.2 数据

```json
{
    username: String, //表示用户名
    password: String, //表示密码
    email: String //表示邮箱
}
```

##### 4.2.3 响应

- 201：注册成功
- 400：数据不符合期望(前端应避免)或邮箱/用户名已存在

### 5 URL: users/\<username\>/

#### 5.1 GET方法

获取用户名为username的用户信息。

##### 5.1.1 权限

- 任何人

##### 5.1.2 响应

- 200：获取成功

##### 5.1.3 返回

```python
{
    'name': '叶良辰',
    'biography': '后会有期',
    'school': '',
    'department': '',
    'sex': '男',
    'username': 'moon',
    'email': 'moon@saiblo.com',
    'email_md5': '496f4e4ae237bc3107d31400e5b61257',
    'date_joined': 'Mon, 14 Oct 2019 14:57:44 +0800',
    'id': 2,
    'is_active': True
}
```

#### 5.2 PUT方法

修改用户个人信息(不包括密码)。

##### 5.2.1 权限

- 本人

##### 5.2.2 数据

```json
{
    name: String,//表示姓名
    biography: String, //表示个人简介
    school: String, //表示所在学校
    department: String, //表示院系
    sex: String //必须是“男”、“女”、“保密”三选一
}
```

##### 5.2.3 响应

- 200：修改成功
- 400：数据不符合期望(前端应避免)

#### 5.3 PATCH方法

与`put`方法作用相同。

#### 5.4 DELETE方法

删除该账号。

##### 5.4.1 权限

- 本人

##### 5.4.2 响应

- 204：删除成功
