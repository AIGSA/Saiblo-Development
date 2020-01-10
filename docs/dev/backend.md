# 后端

后端主要用Django REST架构完成，作为了一个API服务器为前端提供数据，为此使用到以下**核心库**：

- Django：基本框架库
- Django REST Framework：提供REST接口
- Simple JWT：提供JWT认证服务
- Django Crontab：定时执行任务，用于跑天梯赛等

## 模型

作为Django下开发数据库的数据库的基础，本着精简的原则设计了各种model，在backend/saiblo/models文件夹中定义了所有模型，下面在描述模型所在文件时均是相对该文件夹的相对路径。

注：如无特殊说明，模型将继承自`Django.db.models.Model`，且默认包含了自增主键字段`id`。

### User

继承自`AbstrackUser`，作为网站的用户模型，在user.py中。

#### 字段

`AbstrackUser`本来包含以下字段：

- username：用户名，仅限字母与数字，唯一
- last_login：最近登陆时间，自动更新
- is_superuser：bool，是否是超级用户
- first_name & last_name：**未使用**
- email：邮箱号，唯一
- is_staff：bool，是否有Django管理网站访问权限
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

#### 说明

1. 后端支持比赛随时修改时间
2. 为了自洽性，将游戏的天梯设置为隐性的比赛天梯，赛制为`auto`，因此后文`ActiveCode`在设置时会生成对应的`Paritipation`

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

权限管理使用了`rest_framework.permissions`部分的内容，在backend/saiblo/permissions文件夹中包含了所有权限设定，均继承自`rest_framework.permissions.BasePermission`。关于权限的使用属于Django REST基础知识，这里仅仅按文件列举出目前包含的权限以及语义。

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

## 视图

视图即后端所提供的API，这部分文件全部在backend/saiblo/views中，写法由Django REST框架决定，遵循REST规范，此处将它们罗列一遍毫无意义。

需要注意的是用到的基类可能是混合的，对于一个完整的REST API，视图类继承自`rest_framework.viewsets.ModelViewSet`，但是对于一些孤立的API，可能继承自`APIView`，对于那些不支持全部API，只需要部分功能的类，我采用了多继承的方法，即继承`viewsets.GenericViewSet`与多个`viewsets.mixins.SomeModeMixin`，其中的`Some`为某个具体操作，这样一来可以达到自行组合API的效果。

## 序列化

为了将模型数据序列化成为json数据，用到了serializers，所有的serializer均保存在backend/saiblo/serializers文件夹中，且继承自`rest_framework.serializers.ModelSerializer`，用到了嵌套序列化等复杂结构，使用了视图参数等不优雅的写法，总的来说仍然是遵循着Django REST的支持完成，此处也没有必要罗列一遍。

需要强调的是要注意时间的序列化，前端在开发过程中出现了RFC_2822与RFC_3339标准的混用，后端目前的设定是输入格式为两种标准均支持，输出格式为RFC_2822，这些格式字符串均定义在backend/saiblo/utils/string.py中，维护时建议遵循原有风格。

## 信号

由于信号功能在开发到一半时才学会，所以设计上可能不能保证绝对合理，某些可以采用信号完成的内容在别处完成了，并且信号的设计可能也不优雅，如果有需要可以加强对信号的集成。

信号全部定义在backend/saiblo/signals文件夹中，确切地说与其叫“信号”不如叫“槽”更合适，官方文档推荐把这些槽函数放置于signals文件夹中，我按照该建议完成，但是可以理解成这是一个slots文件夹。

在该文件夹中，signal.py定义了人为新增的信号，包含以下信号：

1. match_prepared：天梯生成的自动对局准备完毕时触发
2. fighter_changed：天梯AI变更时触发
3. activate_code_changed：游戏出战AI变更时触发，疑似冗余
4. ladder_match_judged：天梯对局被评测完成时触发

下面是所有的槽函数。

### auto_deal

- 文件：application.py
- 信号：post_save
- 来源：Application
- 效果：如果是created则处理自动处理申请(不允许任何人加入或自动通过申请)的逻辑，如果不是则处理通过申请的逻辑。

### remove_user

- 文件：application.py
- 信号：post_delete
- 来源：Application
- 效果：在删除申请时移除对应的用户

### add_compile_task

- 文件：code.py
- 信号：post_save
- 来源：Code
- 效果：处理向评测机发起代码编译任务的逻辑

### update_game_ladder

- 文件：code.py
- 信号：post_save
- 来源：ActiveCode
- 效果：在用户添加出战AI时新增比赛AI时生成对应的`Participation`，处理用户出战AI修改

### delete_game_ladder

- 文件：code.py
- 信号：post_delete
- 来源：ActiveCode
- 效果：在游戏退出游戏天梯时删除相应的`Participation`

### deal_code_change

- 文件：contest.py
- 信号：fighter_changed
- 来源：Participation
- 效果：在用户更换天梯AI时给出天梯惩罚分，目前是减100

### add_game_contest

- 文件：game.py
- 信号：post_save
- 来源：Game
- 效果：在新建Game时添加对应的隐性比赛

### update_ladder

- 文件：group.py
- 信号：m2m_changed
- 来源：Group.members.through
- 效果：在小组离开用户时，将他在该小组的所有比赛的信息删除

### deal_application

- 文件：group.py
- 信号：post_save
- 来源：Group
- 效果：在创建小组时为创建者生成隐性申请，在设置为不允许任何人加入时删除所有申请，在设置成自动通过申请时通过所有现有申请

### send_ladder_match

- 文件：match.py
- 信号：match_prepared
- 来源：Match
- 效果：在新的对局生成时将它传给评测机

### update_ladder

- 文件：match.py
- 信号：ladder_match_judged
- 来源：Match
- 效果：在评测完对局后处理对局选手天体分数的变更

## 定时任务

为了实现后端定时发起天梯对局的效果，用到了Django-crontab设置定时任务，任务的执行函数在backend/saiblo/tasks文件夹中，目前该文件夹只有ladder.py，包含以下内容。

### NAME_TO_MATCH_CREATOR

一个字典常量，为不同赛制绑定相应对局生成函数，这里理应允许赛事方自行制定赛制并接入。

### auto_match

被定时调用的函数，每隔一个固定时间会在这里生成天体对局，时间间隔设置在backend/app/settings.py的`CRONJOBS`列表中，具体用法参加Django-crontab文档。

### randon_create_match

暂时使用的随机生成对局的函数，会自动在比赛达到所需要人数下限后生成对局，随机在参赛AI中进行抽取。

