# 后端

后端主要用django-rest架构完成，作为了一个API服务器为前端提供数据，为此使用到以下**核心库**：

- django：基本框架库
- djangoresetframework：提供rest接口
- djangorestframework-simplejwt：提供jwt认证服务
- django-crontab：定时执行任务，用于跑天梯赛等

## 模型

作为django下开发数据库的数据库的基础，本着精简的原则设计了各种model，在backend/saiblo/models文件夹中定义了所有模型，下面在描述模型所在文件时均是相对该文件夹的相对路径。

注：如无特殊说明，模型将继承自`django.db.models.Model`，且默认包含了自增主键字段`id`。

### User

继承自`AbstrackUser`，作为网站的用户模型，在user.py中。

#### 字段

`AbstrackUser`本来包含以下字段：

- username：用户名，仅限字母与数字，唯一
- last_login：最近登陆时间，自动更新
- is_superuser：bool，是否是超级用户
- first_name & last_name：**未使用**
- email：邮箱号，唯一
- is_staff：bool，是否有django管理网站访问权限
- is_active：bool，用户是否已激活
- date_joined：注册时间，自动生成

此外，`User`额外添加了以下字段：

- name：真实姓名，支持汉字
- nickname：昵称，目前**未使用**
- biography：个人简介
- school：学校
- department：院系
- sex：性别，必须是[”男“，”女“，”保密“]之一
- email_md5：email的md5值，单独存储它以便于前端加载头像

### Group

网站的小组模型，在group.py中。

#### 字段

- name：小组名
- introduction：小组简介
- reason：小组创建理由
- size：小组人数，该字段会在小组save时自动变更
- creator：小组创建者，是一个指向`User`的多对一外键
- managers：小组管理员，是一个指向`User`的多对多外键
- members：小组普通成员，是一个指向`User`的多对多外键
- applicants：小组的申请者，是一个指向`User`的多对多外键，通过`Application`记录
- created_time：小组创建时间，自动生成
- is_permitted：bool，小组是否已被网站管理员批准创建，True为过审，False为驳回，None为未处理
- auto_pass：bool，小组是否自动处理申请(自动处理即自动通过)
- accept_application：bool，小组是否允许被申请
- visible：bool，小组是否可见，仅一些逻辑上的小组会处于不可见状态

#### 说明

1. accept_application设置为Flase时小组为”仅允许管理员邀请加入“，当它设置为True时auto_pass才有意义，此情况下auto_pass设置为True表示”允许任何人加入“，Flase表示”需要管理员审核后加入“。
2. creator、managers、members所包含的User互斥，它们数量的总和为小组人数(其中creator只有一个且不可更改)，类似于QQ群的管理结构。

### Application

成员加入小组的申请的模型，在application.py中。

#### 字段

- message：申请理由
- user：申请的用户，指向User的多对一外键
- group：申请的小组，指向Group的多对一外键
- time：申请/批准时间，自动更新
- is_permitted：bool，申请是否通过，True为通过，False为拒绝，None为未审核

#### 约束

- 按time降序排序
- (user, group)构成一组唯一元组以及索引，即一个用户对一个小组最多只能存在一个申请，仅当小组被解散或者用户被踢时会删除申请，拒绝后重新申请不会生成新的申请

### Notice

小组发布公告的模型，在notice.py中。

#### 字段

- title：公告标题
- content：公告内容，富文本
- pulisher：公告发布者，是一个指向`User`的多对一外键
- last_editor：公告最后编辑者，是一个指向`User`的多对一外键
- time：公告最后更新时间，自动更新
- group：公告所属小组，是一个指向`Group`的多对一外键

#### 约束

- 按time降序排序

### Game

游戏的模型，在game.py中。

#### 字段

- name：游戏名，字母和数字
- alias：别称，一般为中文全称
- introduction：游戏介绍，富文本
- creator：创建者，是一个指向`User`的多对一外键
- managers：管理员，是一个指向`User`的多对多外键
- game_file：游戏播放器等文件包
- create_time：创建时间，自动生成
- update_time：更新时间，自动更新
- is_permitted：bool，游戏是否过审，True为过审，False为驳回，None为未审核
- player_number：游戏支持的人数，字符串，里面包含游戏支持的可能人数，用空格隔开，如"2 4 6 8"
- has_replay_player：bool，游戏是否能看回放
- allow_watch：bool，游戏是否支持对局过程中观战
- support_online_game：bool，游戏是否支持在线对局
- quick_start：bool，游戏是否支持快速匹配人机模式
- allow_remote：bool，游戏是否支持远程算力

### Entity

智能体(或者俗称AI)的模型，在entity.py中。

#### 字段

- name：名称
- user：用户，是一个指向`User`的多对一外键
- game：游戏，是一个指向`Game`的多对一外键
- next_version：下一个版本号，用单增的方法处理一个AI的各个版本号
- created_time：创建时间，自动生成
- language：编程语言，从枚举类`Language` 中选择，这个类可以继续扩展以支持新的语言

### Code

一个智能体具体某个版本代码的模型，在code.py中。

#### 字段

- id：一个uuid作为主键
- code_file：代码文件
- time：更新时间，自动更新
- entity：所属智能体，是一个指向`Entity`的多对一外键
- version：版本号，它只针对某一个具体智能体唯一
- remark：注释
- compile_status：编译状态，评测机给出，从Code类常量COMPILE_STATUS取值，目前是['未编译', '编译中', '编译成功', '编译失败']
- compile_message：编译输出，评测机给出，主要用于查看报错信息

#### 约束

- 按time降序排序

### Contest

比赛的模型，在contest.py中。

#### 字段

- name：比赛名
- game：游戏，是一个指向`Game`的多对一外键
- group：小组，是一个指向`Group`的多对一外键
- begin_time：开始时间
- end_time：结束时间
- create_time：创建时间，自动生成
- introduction：简介，富文本
- promoter：发起人，是一个指向`User`的多对一外键
- format：赛制，从Contest类的常量成员`FORMAT`中取值，目前为['cycle', 'ladder', 'free', 'auto']

### Participation

用户参加比赛的记录信息的模型，在contest.py中。

#### 字段

- contest：比赛，是一个指向`Contest`的多对一外键
- user：用户，是一个指向`User`的多对一外键
- code：代码，是一个指向`Code`的多对一外键
- socre：天梯分
- joined_time：加入时间，自动生成

#### 约束

- 按score升序排序
- 对于一组('contest', 'user')其Participation唯一

#### 说明

1. 当用户加入比赛时会自动生成相应的Participation，退出(取消出战AI)时会删除该记录，此时天梯分将被清除
2. 用户参加比赛(指定比赛的出战AI)时只需要Pariticipation记录即可，而对游戏设置出战AI则需要用到下面的ActivateCode

### ActivateCode

表示游戏的出战AI的模型，在code.py中。

#### 字段

- user：用户，是一个指向`User`的多对一外键
- game：游戏，是一个指向`Game`的多对一外键
- code：代码，是一个指向`Code`的多对一外键
- time：更新时间，自动更新
- part：天梯记录信息，是一个指向`Participation`的一对一外键，可以为空

#### 约束

- 对于一组('user', 'game')其ActivateCode是唯一的

### Match

记录对局的模型，在match.py中。

#### 字段

- players：对局用户，是一个指向`User`的多对多外键
- contest：比赛，是一个指向`Contest`的多对一外键，可以为空，表示这是游戏对局而非比赛对局
- game：游戏，是一个指向`Game`的多对一外键
- state：状态，评测机给出，从Match常量成员STATE取值，目前为['准备中', '评测中', '评测成功', '评测失败']
- create_time：创建时间，自动生成
- file：对局文件，评测机给出
- message：评测字符串，评测机给出，包含了得分情况，需要后端根据通信格式解析

#### 约束

- 按create_time降序排序

### PlayerInfo

记录对局中各个选手得分情况的模型，在match.py中。

#### 字段

- code：代码，是一个指向`Code`的多对一外键，可以为空，表示这是玩家控制而非AI
- user：用户，是一个指向`User`的多对一外键
- match：对局，是一个指向`Match`的多对一外键
- rank：排名
- score：分数

### ContestInfo

用于记录一个对局关于比赛信息的模型，在match.py中。

疑似冗余，计划删除。

## 权限

权限管理使用了`rest_framework.permissions`部分的内容，在backend/saiblo/permissions文件夹中包含了所有权限设定，均继承自`rest_framework.permissions.BasePermission`。关于权限的使用属于django-rest基础知识，这里仅仅按文件列举出目前包含的权限以及语义。

### application.py

- `IsApplicant`：判断用户是否是申请发起人，疑似冗余，计划删除。
- `IsApplicationGroupManager`：判断用户是否是申请目标小组的管理员，疑似冗余，计划删除。

### code.py

- `IsEntityOwner`：判断用户是否是AI的拥有者。

### contest.py

- `IsActivateContest`：判断比赛是否可以被加入/更改AI，已经结束的比赛不能再设置AI，对于双败淘汰赛制开始后即不能设置AI。

### game.py

- `IsGameManger`：判断用户是否有游戏管理权限，包括管理员与创建者两种情况
- `IsGameCreator`：判断用户是否是游戏创建者

#### group.py

- `NotInGroup`：判断用户是否不在小组中
- `IsgGroupMember`：判断用户是否在小组中
- `IsGroupManger`：判断用户是否有小组管理权限，包括管理员与创建者两种情况
- `IsGroupCreator`：判断用户是否是小组创建者
- `HasHigherPermission`：判断用户是否比目标用户有更高的小组权限，创建者>管理员>普通成员

### user.py

- `IsAccountOwner`：判断用户是否是目标用户

