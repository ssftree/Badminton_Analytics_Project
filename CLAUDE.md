# Badminton Analysis — 项目架构说明

## 整体架构

前后端分离，三层结构：

```
mobile/          (Expo / React Native — iOS/Android App)
    ↕ HTTP REST
backend/         (FastAPI + Python — 分析 API 服务)
    ↕ subprocess
*.py (root)      (CV/ML 分析脚本 — YOLO + 姿态估计)
```

---

## 各层说明

### 1. 移动端 `mobile/`

- **框架**: Expo (React Native)，无导航库，单屏 App
- **入口**: `App.js` → `src/Result.js`（结果展示页）
- **API 通信**: `src/api.js`（轻量封装，XHR 支持上传进度）
- **连接方式**: 局域网直连，后端地址在 `src/config.js` 手动配置
- **关键依赖**: `expo-image-picker`（选视频）、`expo-video`（播放标注视频）

### 2. 后端 API `backend/`

- **框架**: FastAPI + uvicorn，Python 3.12
- **入口**: `server.py`，运行 `uvicorn server:app --host 0.0.0.0 --port 8000`
- **GPU 设备**: 默认 `mps`（Apple Silicon），可用环境变量 `BAD_DEVICE=cuda|cpu` 覆盖
- **Job 管理**: 内存 dict（`JOBS`），重启后丢失；视频文件持久存 `backend/jobs/{job_id}/`
- **并发**: 单线程池（`ThreadPoolExecutor(max_workers=1)`），GPU 任务串行执行

**API 端点**:
| 端点 | 方法 | 说明 |
|------|------|------|
| `/jobs` | POST | 上传 mp4/mov，返回 `job_id` |
| `/jobs/{id}` | GET | 查询状态、进度、阶段 |
| `/jobs/{id}/result` | GET | 获取完整分析 JSON |
| `/jobs/{id}/video` | GET | 下载标注视频（支持 HTTP Range） |
| `/jobs/{id}/chart/{name}` | GET | 获取图表 PNG（swing_timeline / feature_dist） |

**核心模块**:
- `pipeline.py` — 以子进程方式调用根目录分析脚本，聚合输出为统一 summary JSON
- `advice.py` — 基于姿态指标生成启发式教练建议卡片
- `requirements.txt` — 依赖：fastapi、uvicorn、ultralytics、opencv、scikit-learn、matplotlib

### 3. 分析脚本（项目根目录）

两条独立管线，由 `pipeline.py` 以子进程调用：

| 脚本 | 功能 |
|------|------|
| `color_id_weixin.py` | 基于球衣颜色识别球员身份，生成标注视频 |
| `pose_analytics_weixin.py` | 姿态估计，输出挥拍时序图 + 特征分布图 |

**模型文件**:
- `yolov8s-pose.pt` — YOLOv8 姿态估计模型
- `weights/best.pt` / `weights/last.pt` — 自训练羽毛球检测模型（YOLOv11）

### 4. Web Frontend `web_app/`

独立浏览器工作台（vanilla HTML/CSS/JS），默认连接 `http://localhost:8000`
的后端 API，支持上传视频、轮询任务进度，并展示标注视频、球员指标、图表与
教练建议。没有后端时仍可读取 `data.js` 与 `assets/` 展示本地 demo 结果。

---

## Summary JSON 结构

后端所有接口的核心数据格式：

```json
{
  "overview": { "duration_s": 60.0, "total_swings": 42, ... },
  "players": [
    {
      "identity": "红色",
      "swings_per_min": 12.5,
      "mean_arm_extension": 1.8,
      "overhead_pct": 0.65,
      "mean_stance_width": 0.9
    }
  ],
  "media": {
    "video": "annotated.mp4",
    "swing_timeline": "swing_timeline.png",
    "feature_dist": "feature_dist.png"
  },
  "advice": [
    { "title": "挥拍节奏", "body": "..." }
  ]
}
```

---

## 开发启动

```bash
# 后端
cd backend
source .venv/bin/activate
uvicorn server:app --host 0.0.0.0 --port 8000 --reload

# 移动端（需先配置 src/config.js 中的后端 IP）
cd mobile
npx expo start

# Web 前端
cd web_app
python -m http.server 5173
```

## 开发验证要求

- 修改 `web_app/` 后，必须启动 Web 前端并截图确认页面仍可正常渲染，相关功能没有被改坏。
- 默认 Web 启动命令：`cd web_app && python -m http.server 5173`，默认访问 `http://localhost:5173`。
- 如果 Web 改动依赖后端接口或上传/轮询流程，需要同时启动 Backend，验证真实联通流程，而不是只看本地 demo 数据。
- 修改 `backend/` 或分析管线后，必须使用真实 `.mp4` 文件测试后端，可以通过 API 上传或 Web 上传流程完成。
- 优先使用仓库里已有的小型 MP4 样例；如果没有合适文件，先向用户确认要使用哪个视频。
- 收尾时说明实际验证过的 Web URL、截图方式或截图路径、MP4 文件名，以及运行过的 backend/API 检查。

---

## 已知限制 / 待改进

- Job 状态主要存内存；完成后的 `summary.json` 可从磁盘恢复，但生产仍建议用 SQLite / Redis
- 后端 IP 在移动端 `src/config.js` 硬编码，需手动改
- 分析管线以子进程调用，进度解析依赖脚本 stdout 格式（`fi/N`）
- `web_app/` 是本地静态服务，公网部署前需补鉴权、文件大小限制和 HTTPS

<!-- pr-workflow-hook:v1 -->
## PR Workflow

- For every GitHub PR, review both `AGENTS.md` and `CLAUDE.md` before opening or updating the PR.
- If the change affects setup, commands, architecture, workflows, conventions, or agent expectations, update both files in the same PR.
- Preserve existing project-specific guidance; add concise notes instead of replacing unrelated content.
<!-- /pr-workflow-hook:v1 -->
