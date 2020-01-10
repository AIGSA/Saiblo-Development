# 网站后端与测评端

## 概要

该文档定义了网站后端与测评端交互的接口。原则上，对于测评任务的分配，双方采用 Websocket 协议通信。对于测评端向服务器端的数据获取以及发送，采用 HTTP 协议。

## 定义

网站后端将 Websocket 服务器启动在 `ws://HOSTNAME:PORT/ws/` 上
网站后端将 HTTP API 启动在 `http://HOSTNAME:PORT/judger/` 上

## 基本流程

1. 网站后端启动 Websocket Server / HTTP Server
2. 测评端通过 Websocket 连接至网站后端
3. 测评端发送初始化指令，此时网站后端认为测评端上线
4. 网站后端利用 Websocket 和 HTTP API 与测评端进行通信
5. 测评端断开 Websocket，此时网站后端认为测评端下线

## Websocket 通信

### 初始化

#### 测评端发送

```json
{
    type: 0,
    data: {
 		description,
    	address
	}
}
```

**data.description : string : required**

描述测评端的信息，用于前端显示

**data.address : string : optional**

测评端的地址，形如 `1.1.1.1:5000`，必须外网可访问。

如果提供地址，该测评端将被认为提供了一个完整的房间系统服务，会在网站前端的房间服务器列表中进行显示。

如果未提供地址，则网站认为测评端不提供房间系统服务。

### 对局测评任务

#### 网站后端发送

```json
{
    "type": 1
    "data": {
    	match_id,
		game_id,
        player_number,
        "players": [
            {
                type,
                player_id
            }
        ],
	}
}
```

#### 评测端回复

```json
{
    "type": 1
    "data": {
    	match_id,
		success,
		msg,
	}
}
```

表示是否成功接受到这个评测任务

### AI 编译任务

#### 网站后端发送

```json
{
    "type": 2
    "data": {
		code_id
	}
}
```

**code_token : string : required **

表示待编译的代码的 token

#### 评测端回复

```json
{
    "type": 2
    "data": {
		code_id,
		success,
		msg,
	}
}
```

表示是否成功接受到这个编译任务

## HTTP API

### URL: codes/:code_id/

#### GET

只读接口，提供具体代码数据。

**响应**

- 200：获取成功

**返回**

```python
{
    'entity': 							# info of entity
    {
        'user': 'moon', 				# user's name
        'name': 'test1',				# entity's name
        'game': 'love', 				# game's name
        'created_time': 'Tue, 22 Oct 2019 10:04:38 +0800',
        'id': 1, 						# entity's id
        'versions': 3,					# entity's versions
        'update_time': 'Fri, 25 Oct 2019 13:27:17 +0800',
    	'language': 'C++'
    },
    'remark': 'test', 					# code remark
    'version': 5, 						# code version id
    'url': '/judger/codes/1/download/',	# code download url
    'compile_status': '未编译',
    'compile_message': ''
}
```

##### 参数

* `language`选项 - 前端显示信息
  * 'cmake'：c++ with CMake
  * 'make': c++ with Makefile
  * 'cpp': g++
  * 'c': gcc
  * 'python': python single file
  * 'python_zip': python zip file

#### PUT

评测端代码编译完成后，提交编译信息

**数据**

- compile_status: 字符串，表示新的编译状态，取值范围是 `['编译成功', '编译失败']`
- compile_message: 字符串，表示编译信息

**响应**

- 200：修改成功

### URL: codes/:code_id/download/

#### GET

下载目标代码。

**响应**

- 200：获取成功

**返回**

下载内容，文件名为`<code_id>.zip`。

### URL: matches/:match_id/

#### PUT

评测端对局评测完成后，返回评测信息

**数据（Form 格式）**

请求的 Header 应为：`"Content-Type": "multipart/form-data; boundary=BoUnDaRyStRiNg"`

- file: 文件，表示评测文件
- message: 字符串，表示评测结果
- state: 字符串， 表示评测状态，取值范围是 `['准备中', '评测中', '评测成功', '评测失败']`
- scores: 字典，表示各个位置上玩家的得分，如 `{"0": 100, "1", 200}`

**返回**

- 200：修改成功

