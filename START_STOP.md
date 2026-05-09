# DeepTutor Web 启动和停止手册

这份文档说明如何在本地启动、停止 DeepTutor 的 Web 版。Web 版由两个服务组成：

| 服务 | 默认端口 | 作用 |
| --- | --- | --- |
| 后端 FastAPI | `8001` | API、WebSocket、模型调用、知识库等 |
| 前端 Next.js | `3782` | 浏览器界面 |

> 注意：不要把真实 API Key 写进这个文档或提交到 Git。项目根目录的 `.env` 已被 `.gitignore` 忽略，适合放本机配置。

## 1. 首次安装

如果你运行 `source .venv/bin/activate` 时看到 `no such file or directory`，说明当前项目还没有创建虚拟环境。先执行本节，不要直接跳到启动步骤。

在项目根目录执行：

```bash
cd /Users/ben/Desktop/DeepTutor

python3 -m venv .venv
source .venv/bin/activate

python -m pip install --upgrade pip
python -m pip install -e ".[server]"

cd web
npm install
cd ..
```

如果你已经通过 `python scripts/start_tour.py` 完成过引导安装，并且 `.venv/bin/activate` 确实存在，可以跳过这一步。

## 2. 配置模型

复制示例配置：

```bash
cp .env.example .env
```

编辑 `.env`，至少填写这些字段：

```dotenv
BACKEND_PORT=8001
FRONTEND_PORT=3782

LLM_BINDING=dashscope
LLM_MODEL=qwen3.6-plus
LLM_API_KEY=你的_API_KEY
LLM_HOST=https://coding.dashscope.aliyuncs.com/v1
LLM_API_VERSION=

# 可选，但建议和 LLM_API_KEY 保持一致，便于 DashScope 相关兼容逻辑读取。
DASHSCOPE_API_KEY=你的_API_KEY
```

如果默认端口已被占用，可以把端口改成：

```dotenv
BACKEND_PORT=8002
FRONTEND_PORT=3783
```

`.env` 中的端口和模型配置优先级高于临时 shell 环境变量。换模型或换端口后，建议直接改 `.env` 并重启服务。

## 3. 推荐启动方式

每次启动前先进入项目目录并激活虚拟环境：

```bash
cd /Users/ben/Desktop/DeepTutor
source .venv/bin/activate
```

启动 Web：

```bash
python scripts/start_web.py
```

启动成功后，终端会打印前端地址，例如：

```text
Open http://localhost:3782 in your browser.
```

保持这个终端窗口打开。关闭终端或按 `Ctrl+C` 会停止服务。

## 4. 停止方式

如果服务是在前台通过 `python scripts/start_web.py` 启动的，在同一个终端按：

```text
Ctrl+C
```

如果上次异常退出，或者终端已经关闭，可以在项目根目录执行：

```bash
source .venv/bin/activate
python scripts/stop_web.py
```

这个脚本会读取 `scripts/start_web.py` 记录的进程状态并停止后端和前端。

## 5. 手动检查和停止端口占用

查看默认端口是否被占用：

```bash
lsof -n -P -iTCP:8001 -sTCP:LISTEN
lsof -n -P -iTCP:3782 -sTCP:LISTEN
```

如果你使用了备用端口，改查：

```bash
lsof -n -P -iTCP:8002 -sTCP:LISTEN
lsof -n -P -iTCP:3783 -sTCP:LISTEN
```

手动停止占用某个端口的进程：

```bash
kill $(lsof -tiTCP:8001 -sTCP:LISTEN)
kill $(lsof -tiTCP:3782 -sTCP:LISTEN)
```

如果使用备用端口：

```bash
kill $(lsof -tiTCP:8002 -sTCP:LISTEN)
kill $(lsof -tiTCP:3783 -sTCP:LISTEN)
```

如果普通 `kill` 后仍未停止，再确认进程确实属于 DeepTutor，然后使用：

```bash
kill -9 <PID>
```

## 6. 后台运行

日常开发推荐前台运行，因为日志最清楚，`Ctrl+C` 也最安全。

如果必须后台运行，建议使用 `tmux` 或 `screen`：

```bash
tmux new -s deeptutor
cd /Users/ben/Desktop/DeepTutor
source .venv/bin/activate
python scripts/start_web.py
```

退出 tmux 但保持服务运行：

```text
Ctrl+B 然后按 D
```

重新进入：

```bash
tmux attach -t deeptutor
```

停止时进入 tmux 会话后按 `Ctrl+C`。

## 7. 日志和状态确认

后端是否在线：

```bash
curl http://127.0.0.1:8001/
```

备用端口：

```bash
curl http://127.0.0.1:8002/
```

查看系统状态：

```bash
curl http://127.0.0.1:8001/api/v1/system/status
```

测试 LLM 连接：

```bash
curl -X POST http://127.0.0.1:8001/api/v1/system/test/llm
```

备用端口时把 `8001` 改成 `8002`。

如果通过标准启动脚本运行，日志会直接显示在启动终端中。运行过程中生成的用户数据、设置和日志通常在：

```text
data/user/
```

## 8. 常见问题

### `python: command not found`

用 `python3` 创建虚拟环境后，激活 `.venv`：

```bash
python3 -m venv .venv
source .venv/bin/activate
```

激活后再使用 `python`。

### `source: no such file or directory: .venv/bin/activate`

说明 `.venv` 还没有创建。回到第 1 节执行首次安装：

```bash
cd /Users/ben/Desktop/DeepTutor
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -e ".[server]"
```

### `ModuleNotFoundError: No module named 'yaml'`

说明你没有使用项目虚拟环境，或者虚拟环境里还没安装依赖。按下面顺序修：

```bash
cd /Users/ben/Desktop/DeepTutor
source .venv/bin/activate
python -m pip install -e ".[server]"
python scripts/start_web.py
```

### `npm not found`

安装 Node.js 和 npm。项目前端要求 Node.js `20.9+`。安装后重新执行：

```bash
cd web
npm install
cd ..
```

如果项目里已经有 `.tools/bin/npm` 和 `web/node_modules`，新版 `scripts/start_web.py` 会优先使用本地工具目录；普通启动命令不需要手动改 PATH。

### 端口已经被占用

先确认占用者：

```bash
lsof -n -P -iTCP:8001 -sTCP:LISTEN
lsof -n -P -iTCP:3782 -sTCP:LISTEN
```

你可以停止旧进程，也可以在 `.env` 中把端口改成 `8002` 和 `3783` 后重启。

### `Module not found: Can't resolve 'cytoscape'`

这是前端依赖没有完整安装，或当前项目误用了其他目录的 `node_modules`。在当前项目里重新安装前端依赖：

```bash
cd /Users/ben/Desktop/DeepTutor/web
npm install
```

安装完成后，回到项目根目录重启 Web：

```bash
cd /Users/ben/Desktop/DeepTutor
python scripts/stop_web.py
python scripts/start_web.py
```

### LLM 显示未配置或还是旧模型

检查 `.env`：

```bash
grep '^LLM_' .env
grep '^DASHSCOPE_API_KEY' .env
```

确认 `LLM_BINDING=dashscope`、`LLM_MODEL=qwen3.6-plus`、`LLM_HOST=https://coding.dashscope.aliyuncs.com/v1`。修改后重启 Web 服务。

### 知识库或 RAG 不可用

普通聊天只需要 LLM 配置。知识库/RAG 还需要额外配置 embedding：

```dotenv
EMBEDDING_BINDING=...
EMBEDDING_MODEL=...
EMBEDDING_API_KEY=...
EMBEDDING_HOST=...
```

配置后重启服务，并在 Web 设置页测试 embedding 连接。
