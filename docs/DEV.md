# SQLBot 开发模式部署说明（非容器）

本地直接运行后端与前端，不使用 Docker/Podman。

## 环境要求

- **Node.js** 18+（前端）
- **Python 3.11**（后端，推荐用 [uv](https://github.com/astral-sh/uv) 管理依赖）
- **PostgreSQL** 13+（后端数据库）
- （可选）Redis（不配置则使用内存缓存）

## 一、数据库

后端依赖 PostgreSQL，默认连接 `localhost:5432`，用户 `root`，密码 `Password123@pg`，数据库 `sqlbot`。

### 方式 A：Podman 跑 PostgreSQL（推荐）

项目根目录提供脚本，与上述默认配置一致：

```bash
# 启动（首次会拉镜像并建库）
./podman-pg.sh start

# 国内镜像加速
DOCKER_MIRROR=docker.1ms.run ./podman-pg.sh start

# 查看状态 / 停止 / 删除容器
./podman-pg.sh status
./podman-pg.sh stop
./podman-pg.sh rm
```

数据目录默认在 `./data/postgres`，删除容器后数据仍保留。

### 方式 B：本机安装 PostgreSQL

1. 安装并启动 PostgreSQL（如 `brew services start postgresql@14`）。
2. 创建库与用户（与上面默认一致即可）：

```bash
createuser -P root
createdb -O root sqlbot
```

## 二、配置

1. 在**项目根目录**复制环境变量示例并修改：

```bash
cp .env.example .env
# 按需编辑 .env（数据库连接、FRONTEND_HOST 等）
```

2. 前端开发环境已配置为请求本地后端：`frontend/.env.development` 中  
   `VITE_API_BASE_URL=http://localhost:8000/api/v1`，一般无需改。

## 三、依赖安装

```bash
# 后端（在项目根目录）
cd backend
uv sync
# 若无 uv：pip install -e . 或根据 pyproject.toml 安装

# 前端
cd frontend
npm install
```

## 四、启动（开发模式）

### 方式一：一键脚本（推荐）

在项目根目录执行：

```bash
chmod +x dev.sh
./dev.sh
```

- 后端：`http://localhost:8000`（默认 8000，可设 `BACKEND_PORT`）
- 前端：`http://localhost:8001`（默认 8001，可设 `FRONTEND_PORT`）

脚本会先**清理**占用 8000/8001 的旧进程及上次记录的 PID，再启动后端与前端；PID 写入 **`.dev/backend.pid`**、**`.dev/frontend.pid`**（可设 `PID_DIR` 自定义目录）。退出时 Ctrl+C 会停止后端与前端并删除 PID 文件。

### 方式二：分两个终端分别启动

**终端 1 - 后端：**

```bash
cd backend
uv run uvicorn main:app --host 0.0.0.0 --port 8000 --reload
# 或：python -m uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

**终端 2 - 前端：**

```bash
cd frontend
npm run dev
```

访问：前端 http://localhost:5173 ，后端 API http://localhost:8000/docs 。

## 五、默认账号

- 用户名：`admin`
- 密码：`admin`

（与容器部署一致，首次启动会通过 migration 初始化。）

## 六、可选：图表服务 g2-ssr

若需服务端图表渲染，可单独启动 g2-ssr（默认端口 3000）：

```bash
cd g2-ssr
npm install
node app.js
```

不启动时，图表可能仅在前端渲染，功能仍可正常开发调试。
