## 部署方式

将 Git 仓库 clone 到本地

```bash
git clone https://git.net9.org/Sengxian/Saiblo
cd Saiblo
```

### Docker 部署（不推荐）

使用 Dockerfile 构建镜像并自动化部署

```bash
docker build -t saiblo .
docker run -d -p 127.0.0.1:3000:80 --name saiblo saiblo
docker start saiblo
```

网站将会在 `127.0.0.1:3000` 上启动，你可以修改为 `0.0.0.0:80` 监听来自所有地址的链接。

这种方法能迅速启动一个 Saiblo 实例，但不推荐用于生产环境。

### 本机部署

#### 依赖条件

需要在系统中安装以下依赖

- Node.js
- Python3
- Redis

其中 Redis 将作为后端的缓存服务器，需要自行配置并运行，后端默认 Redis 地址为  `127.0.0.1:6379`（可以在 `backend/app/settings.py` 的 `CACHES`  处进行更改）。

#### 安装依赖

安装前后端所需的依赖包，推荐使用 virtualenv 来管理 Python 环境。

```bash
cd frontend
npm install
cd ../backend
pip install -r requirements.txt
```

#### 初始化数据库

```bash
cd backend
python manage.py migrate
python manage.py loadtestdata # 如果有需要，可以载入这套测试数据，包含两个游戏以及三个小组
```

#### 启动前端

```bash
cd frontend
npm run build # 编译前端
export NUXT_HOST=0.0.0.0 # 指定监听地址
export NUXT_PORT=80 # 指定端口
npm run start # 启动前端
```

#### 启动后端

```bash
cd backend
uvicorn app.asgi:application --workers 1 --port 8000 --host 127.0.0.1
# 以 1 个 worker 启动后端，如有需要可以增加 worker 数量，但需要换用 MySQL 等并发友好的数据库，否则会在访问过快时死锁
```

#### 可持久化运行

推荐使用 `supervisor` 配置服务，也可使用 `tmux` 来可持久化运行前后端

