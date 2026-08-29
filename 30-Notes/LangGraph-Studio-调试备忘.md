---
tag: note/langgraph, note/debug
---

# LangGraph Studio 调试 claims_review_agent 备忘

> 记录 2026-08-29 为 Windows 环境搭建 LangGraph Studio 的过程，避免下次重踩坑。

## 关键结论
- Windows 无 Studio 桌面版（仅 macOS + Docker）。Windows 上唯一官方路径是
  `langgraph dev` 起本地 API server + 用 LangSmith 网页版 Studio。
- `langgraph-api` 这个版本（0.13.2）**没有打包本地前端**，所以 `http://localhost:2024/studio`
  和 `/ui` 会 404/500。可视化界面必须走：
  `https://smith.langchain.com/studio/?baseUrl=http://localhost:2025`

## 版本坑（重要）
- 项目 `rag` 环境的 `langgraph==0.2.76` 太旧，会把 `langgraph-api` 拖成缺 `feature_flags`
  的 0.2.1，现代 dev server 起不来（`ImportError: cannot import name 'feature_flags'`）。
- 解决：建独立影子 venv `D:/DevelopSoftware/.studio_venv`（--system-site-packages 复用
  rag 其余包），把 langgraph 全家桶升级成现代一致栈。rag 环境 0.2.76 原封未动。

## 启动命令（项目根目录）
```powershell
D:/DevelopSoftware/.studio_venv/Scripts/langgraph.exe dev --no-browser --port 2025
```

## 已暴露的图（langgraph.json）
- `claims_review`：顶层编排图（router → dispatch_workflow / dispatch_rag）
- `claims_workflow`：理赔 Workflow Agent（agent ↔ tools 循环）
- `claims_rag`：理赔 RAG Agent（agent ↔ search_knowledge 循环）

## 注意
- 改 `langgraph.json` 后 dev server **不会热重载**，需重启（且旧进程若跑在宿主机进程空间，
  沙箱 kill/taskkill 够不到，只能换端口或在任务管理器结束占用端口的 python）。
- 真正 invoke 节点跑检索需要 Milvus：`docker compose -f D:\Docker\docker-compose.yml up -d`
- Studio 只展开顶层节点；子图需各自暴露成 graph 才能单独调试。
