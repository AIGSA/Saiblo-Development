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

### 6 URL: users/\<username\>/update-password/

#### 6.1 PUT方法

修改用户名为username的用户密码。

##### 6.1.1 权限

- 本人

##### 6.1.2 数据

```json
{
    old_password: String, //表示原来的密码
    password: String //表示新密码
}
```

##### 6.1.3 响应

- 200：修改成功
- 400：数据不符合期望(前端应避免)或密码错误

## 小组模块

小组用于组织一些用户进行讨论、比赛，由管理者进行管理。小组模块的主要功能有**创建**、**查看**、**修改**小组信息；**审批**、**删除**小组人员或者将小组成员**提升为管理者**、**降为普通成员**；**添加**、**修改**、**删除**比赛；用户**申请**、**取消入组**。

### 1 URL: groups/

#### 1.1 GET方法

获取所有过审的小组信息，便于用户查看。

##### 1.1.1 权限

- 任何人

##### 1.1.2 响应

- 200：获取成功

##### 1.1.3 返回

```python
[
    {
        'name': '普及组',
        'introduction': '在良辰的地盘，良辰有一百种办法 让你待不下去 。',
        'reason': '良辰不介意陪你玩玩。',
        'creator': 'moon',
        'auto_pass': False,
        'accept_application': True,
        'id': 1,
        'is_permitted': True,
        'managers': ['sky', ],
        'created_time': 'Mon, 14 Oct 2019 15:11:43 +0800',
        'relation': 'created'
    },
    ...
]
```

#### 1.2 POST方法

用户申请新建小组。

##### 1.2.1 权限

- 已登录用户，即用户状态为“已登录”。

##### 1.2.2 数据

```json
{
    name: String, //表示小组名
    introduction: String, //表示简介
    reason: String //表示申请理由
}
```

##### 1.2.3 响应

- 201：创建成功
- 400：数据不符合期望（前端应避免）

### 2 URL: groups/applying/

#### 2.1 GET方法

获取所有申请创建的小组列表，便于超级用户对小组申请进行审批。

##### 2.1.1 权限

- 超级用户

##### 2.1.2 响应

- 200：获取成功

##### 2.1.3 返回

```python
[
    {
        'name': '提高组',
         'introduction': '我赵日天第一个不服！',
         'reason': '给我赵日天一个面子！',
         'creator': 'sky',
         'auto_pass': False,
         'accept_application': True,
         'id': 2,
         'is_permitted': None,
         'managers': ['sky', ],
         'created_time': 'Mon, 14 Oct 2019 15:12:06 +0800'
    },
    ...
]
```

### 3 URL: groups/\<group_id\>/

#### 3.1 GET方法

返回目标id的小组信息。

##### 3.1.1 权限

- 任何人

##### 3.1.2 响应

- 200：获取成功

##### 3.1.3 返回

```python
{
    'name': '普及组',
    'introduction': '在良辰的地盘，良辰有一百种办法 让你待不下去 。',
    'reason': '良辰不介意陪你玩玩。',
    'creator': 'moon',
    'auto_pass': False,
    'accept_application': True,
    'id': 1, 'is_permitted': True,
    'managers': ['sky'],
    'created_time': 'Mon, 14 Oct 2019 15:11:43 +0800',
    'relation': 'idle'
}
```

#### 3.2 PUT方法

修改目标id的小组信息，如果过审失败则可以靠此重新申请。

##### 3.2.1 权限

- 创建者
- 小组管理员

##### 3.2.1 数据

```json
{
    name: String, //表示小组名
    introduction: String, //表示简介
    auto_pass: Boolean, //决定用户申请后是否会自动通过
    accept_application: Boolean //决定是否允许用户主动申请加入
}
```

注：`accept_application`设置为`False`即为只允许管理员邀请加入，设置为`True`后`auto_pass`设为`False`即为管理员验证加入，设为`True`即为允许任何人加入。

##### 3.2.2 响应

- 200：修改成功

#### 3.3 PATCH方法

超级用户审批小组申请。

##### 3.3.1 权限

- 超级用户

##### 3.3.2 数据

```json
{
    permission: Boolean, //表示通过与否
}
```

##### 3.3.3 响应

- 200：审批成功
- 400：数据不符合期望（前端应避免）

#### 3.4 DELETE方法

创造者可以删除目标小组，无论它是否过审。

##### 3.4.1 权限

- 创建者

##### 3.4.2 响应

- 204：删除成功

### 4 URL: users/\<username>/applications/

#### 4.1 GET方法

用户获取自己的所有入组申请。

##### 4.1.1 权限

- 本人

##### 4.1.2 响应

- 200：获取成功

##### 4.1.3 返回

```python
[
    {
        'message': '我日天想会会你。',
        'id': 1,
        'user': {
            'name': '赵日天',
            'biography': '',
            'school': '',
            'department': '',
            'sex': '保密',
            'username': 'sky',
            'email': 'sky@saiblo.com',
            'email_md5': 'f3be03c754c5daf465a339ea9b537d7b',
            'date_joined': 'Mon, 14 Oct 2019 15:10:52 +0800',
            'id': 3,
            'is_active': True
        },
        'time': 'Mon, 14 Oct 2019 23:35:29 +0800',
        'group': 1,
        'is_permitted': None
    },
    ...
]
```

### 5 URL: users/\<username>/applications/\<application_id>/

#### GET

用户获取入组申请详情。

**权限**

- 本人

**响应**

- 200：获取成功

**返回**

```python
{
    'message': '我日天想会会你。',
    'id': 1, 
    'user': {
        'name': '赵日天', 
        'biography': '', 
        'school': '',
        'department': '', 
        'sex': '保密', 
        'username': 'sky', 
        'email': 'sky@saiblo.com',
        'email_md5': 'f3be03c754c5daf465a339ea9b537d7b', 
        'date_joined': 'Mon, 14 Oct 2019 15:10:52 +0800',
        'id': 3, 
        'is_active': True
    }, 
    'time': 'Mon, 14 Oct 2019 23:35:29 +0800', 
    'group': 1,
    'is_permitted': None
}
```

#### PUT


用户修改入组申请内容，在被拒绝后调用它会重新发起申请。

**权限**

- 本人

**数据**

- message: 字符串，表示申请理由

**响应**

- 200：修改成功
- 400：数据不符合期望（前端应避免）

#### PATCH

同PUT

#### DELETE

用户撤销或删除未通过的申请。

**权限**

- 本人

**响应**

- 204：删除成功

### groups/\<group_id\>/applications/

#### GET

返回目标id的全部申请。

**权限**

- 管理员

**响应**

- 200：获取成功

#### POST

用户发起向目标小组的申请，要求小组已过审。

**权限**

- 已登录
- 非成员

**数据**

- message: 字符串，表示申请理由

**响应**

- 201：申请成功
- 400：数据不符合期望（前端应避免）或用户已在组中
- 403：小组禁制任何人加入
- 404：小组未过审

### groups/\<group_id\>/applications/\<application_id\>/

#### GET

获取目标组的目标申请信息。

**权限**

- 管理员

**响应**

- 200：获取成功
- 404：小组/申请不存在或申请不属于该小组

#### PUT

审批目标申请，需要小组管理员以上权限。

**权限**

- 管理员

**数据**

- permit：布尔值，表示是否通过申请

**响应**

- 400：数据不符合期望（前端应避免）
- 200：审批成功

**PATCH**

同PUT

#### DELETE

删除(已被通过的冗余)目标申请。

**权限**

- 管理员

**响应**

- 204：清除成功

### groups/\<group_id>/users/

**GET**

获取该组内所有成员（不包括创建者）的信息。

**权限**

- 管理员

**响应**

- 200：获取成功

#### POST

向组内添加成员（注：现设定不需要经过同意）。

**权限**

- 管理员

**数据**

- username: 字符串，表示目标成员用户名

**响应**

- 201：添加成功
- 400：数据不符合期望（前端应避免）或用户不存在或用户已经在组中

### groups/\<group_id>/users/\<username>/

#### PUT

修改一个组员的管理员权限。

**权限**

- 创建者

**数据**

- management: bool，表示是否设为管理员

**响应**

- 201：修改成功
- 400：数据不符合期望（前端应避免）

#### PATCH

同PUT。

#### DELETE

踢出一个组员或自己退组，需注意创建者无法通过此接口退群，否则会404，如果需要解散群请delete小组。

**权限**

- 更高权限或本人

**响应**

- 204：踢出成功

### groups/\<group_id\>/notices/

#### GET

查看目标id小组所有公告，需要是小组成员(这一点可修改)。

**权限**

- 组成员

**响应**

- 200：获取成功，返回json数据
- 404：小组未过审

**返回**

```python
[
    {
        'title': '测试公告',
        'content': '公告测试',
        'group': 1, 
        'last_editor': 'moon', 
        'id': 1, 
        'time': 'Mon, 14 Oct 2019 23:34:31 +0800',
        'publisher': 'moon'
    },
]
```

#### POST

发布新公告。

**权限**

- 需要有小组管理员以上权限

**数据**

- title: 字符串，表示公告标题
- content: 字符串，表示公告内容

**响应**

- 201：发布成功
- 400：数据不符合期望（前端应避免）
- 404：小组未过审

### groups/\<group_id\>/notices/\<notice_id>/

#### GET

获取目标id小组目标id公告内容。

**权限**

- 组成员

**响应**

- 200：获取成功

**返回**

```python
{
    'title': '测试公告',
    'content': '公告测试',
    'group': 1, 
    'last_editor': 'moon', 
    'id': 1, 
    'time': 'Mon, 14 Oct 2019 23:34:31 +0800',
    'publisher': 'moon'
}
```

#### PUT

修改目标id小组目标id公告内容。

**权限**

- 管理员

**数据**

- title: 字符串，表示修改后标题
- content: 字符串，表示修改后内容

**响应**

- 200：修改成功
- 400：数据不符合期望（前端应避免）

#### PATCH

同PUT

#### DEELTE

删除目标id小组的目标id公告

**权限**

- 管理员

**响应**

- 204：删除成功

