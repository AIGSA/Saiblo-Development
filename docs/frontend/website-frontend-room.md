# 网站前端-房间系统

[TOC]

## room/\_id.vue

| 参数 | 值                        |
| ---- | ------------------------- |
| 地址 | `/room/<room id>`         |
| 用途 | 进入编号为 room id 的房间 |
| 权限 | 登录用户                  |
| 预览 | ![](imgs\groups_id.PNG)   |

| 属性            | 解释                                                                                 |
| --------------- | ------------------------------------------------------------------------------------ |
| is_web_player   | 说明当前用户是不是使用网页播放器                                                     |
| play_web_player | 说明是不是需要播放网页播放器                                                         |
| room_game_id    | 指明当前房间游玩的游戏编号                                                           |
| web_socket      | 与评测端交互用的 Websocket                                                           |
| server          | 存储与评测端的连接的一些信息，open 表示是否开启连接，failures 表示失败重连的次数等等 |
| look            | 存储一个用以查看某些信息的小窗口的参数                                               |
| room            | 存储整个房间的信息，例如玩家 players，游戏 game，房主 host，观战用户 users 等等      |
| room_editor     | 用于存储修改的房间信息                                                               |
| chat            | 聊天室数据                                                                           |
| my_entities     | 存储用户自己在当前房间中可用的 AI 信息                                               |
| code_selector   | 存储用户选择的 AI 信息，用以将 AI 加入到房间中                                       |
| iframe          | 网页播放器的参数                                                                     |
| configs         | 房间内游戏参数的配置                                                                 |

| 函数                 | 参数                    | 返回值 | 解释                                                         |
| -------------------- | ----------------------- | ------ | ------------------------------------------------------------ |
| tryAutoStart         | null                    | null   | 尝试自动启动一局游戏，为快速开始人机对局所使用的函数         |
| haveALook            | message, title, content | null   | 用一个小窗口查看目标信息                                     |
| clearWebSocket       | null                    | null   | 清除与评测端连接的 Websocket                                 |
| initWebSocket        | null                    | null   | 初始化与评测端连接的 Websocket                               |
| webSocketOnOpen      | null                    | null   | 与评测端连接后所调用的函数                                   |
| webSocketOnMessage   | event                   | null   | 收到由评测端发送的信息时调用的处理函数                       |
| sendMessage          | data                    | null   | 向评测端发送数据                                             |
| getUserByUsername    | users, username         | user   | 从 users 中通过 username 找到目标 user 并返回信息            |
| getGameById          | id                      | game   | 从 games 中通过 id 找到对应的 game 并返回信息                |
| getEntityById        | entities, id            | entity | 从 entities 中通过 id 找到目标 entity 并返回信息             |
| getCodeById          | codes, id               | code   | 从 codes 中通过 id 找到目标 code 并返回信息                  |
| loadStatus           | data                    | null   | 从评测端收到房间信息后加载房间信息                           |
| joinSeat             | index                   | null   | 让当前用户加入房间的某一个座位上                             |
| quitSeat             | index                   | null   | 让当前用户退出他所占据的座位                                 |
| saveGameConfigs      | null                    | null   | 房主保存对游戏的配置信息，将其发送给评测端                   |
| initCodeSelector     | seat                    | null   | 初始化 seat 位置上的 AI 选择器，以便用户选择一个 AI 进行加入 |
| selectCode           | null                    | null   | 用户确认选择一个 AI，并将其加入到选择的座位上                |
| startGame            | null                    | null   | 房主开始游戏                                                 |
| postMessage          | null                    | null   | 发送聊天信息                                                 |
| receiveMessage       | data                    | null   | 收取聊天信息进行显示                                         |
| getMyEntities        | game_id                 | null   | 获取用户在游戏编号为 game_id 的所有可用 AI 以供用户选择      |
| getMyEntity          | entity_id               | entity | 通过 entity_id 获取用户的 entity                             |
| getMyCode            | entity, code_id         | code   | 通过 code_id 来获取 entity 中的某份 code                     |
| onLoad               | null                    | null   | 网页播放器加载好后调用的初始化函数                           |
| initPlayer           | null                    | null   | 初始化网页播放器                                             |
| iframePostMessage    | data                    | null   | 向网页播放器发送信息请求其初始化，或者是重新加载等等         |
| iframeReceiveMessage | event                   | null   | 收到由网页播放器发回的信息，对其进行处理                     |
