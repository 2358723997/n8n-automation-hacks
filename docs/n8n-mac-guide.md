# 🚀 n8n + Runner + Mac 部署踩坑完整指南

> **Status**: ✅ Production Ready | **Platform**: Docker Desktop for Mac | **Author**: 狮子玄

在使用 Mac 部署 n8n 分布式 Runner 时，最头疼的就是文件读写权限报错。本文总结了从权限、路径到代理的完整避坑方案。

---

## 📖 目录
- [1. 背景与目标](#1-背景与目标)
- [2. 常见痛点 (Issues)](#2-常见痛点-issues)
- [3. 核心原因分析](#3-核心原因分析)
- [4. 终极解决方案](#4-终极解决方案)
- [5. Docker Compose 配置模板](#5-docker-compose-配置模板)
- [6. Mac 环境预处理](#6-mac-环境预处理)
- [7. 总结与 Tips](#7-总结与-tips)

---

## 1. 背景与目标
为了提升性能，官方推荐使用 **n8n Runners** 分布式执行任务。在 Mac 上部署时，我们需要确保：
* Read/Write File 节点可以无障碍读写。
* Runner 节点能实时同步主容器生成的文件。
* 彻底解决 Mac Bind Mount 的权限地狱。

---

## 2. 常见痛点 (Issues)
如果你遇到以下报错，看这篇文章就够了：
* ❌ `NodeApiError: The file "/files/xxx" is not writable`
* ❌ `EACCES: permission denied, open '/tmp/xxx'`
* ❌ Runner 找不到主容器写入的文件（路径不一致）。

---

## 3. 核心原因分析

> [!IMPORTANT]
> **主要坑点有三：**
> 1. **安全沙箱**：n8n 默认限制文件访问路径，需环境变量授权。
> 2. **Mac /tmp 特性**：Mac 宿主机的 `/tmp` 是特殊映射，不能直接挂载给 Docker 用作 Socket 交换。
> 3. **UID/GID 冲突**：Docker 容器内的 `root` 与 Mac 宿主机用户的权限映射不匹配。

---

## 4. 终极解决方案

1. **统一路径挂载**：主容器和 Runner 必须挂载同一个 Mac 目录。
2. **强制覆盖临时目录**：通过 `N8N_TEMP_DIR` 环境变量重定向临时文件，避开系统 `/tmp`。
3. **环境变量授权**：设置 `N8N_RESTRICT_FILE_ACCESS_TO` 指向你的挂载点。
4. **代理绕过**：在 `NO_PROXY` 中加入容器名，防止 Runner 通信被 Clash 等拦截。

---

## 5. Docker Compose 配置模板

```yaml
version: "3.8"

services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n_demo
    user: root
    privileged: true
    environment:
      - N8N_RUNNERS_ENABLED=true
      - N8N_RUNNERS_MODE=external
      - N8N_RUNNERS_BROKER_PORT=5679
      - N8N_RUNNERS_AUTH_TOKEN=secure_token_123
      # ⚠️ 关键：限制文件访问目录
      - N8N_RESTRICT_FILE_ACCESS_TO=/n8n_files
      - N8N_TEMP_DIR=/tmp
      # ⚠️ 关键：防止代理拦截
      - NO_PROXY=localhost,127.0.0.1,n8n,n8n-runners,host.docker.internal
    ports:
      - "5678:5678"
      - "5679:5679"
    volumes:
      # 请将 /Users/yourname/ 修改为你真实的 Mac 路径
      - /Users/yourname/n8n-demo/n8n_data:/n8n_data
      - /Users/yourname/n8n-demo/files:/n8n_files
      - /Users/yourname/n8n-demo/files/tmp:/tmp
    networks:
      - n8n-network

  task-runners:
    image: n8nio/runners:latest
    container_name: n8n-runners
    user: root
    environment:
      - N8N_RUNNERS_TASK_BROKER_URI=http://n8n:5679
      - N8N_RUNNERS_AUTH_TOKEN=secure_token_123
      - N8N_TEMP_DIR=/tmp
    volumes:
      - /Users/yourname/n8n-demo/files:/n8n_files
      - /Users/yourname/n8n-demo/files/tmp:/tmp
    networks:
      - n8n-network

networks:
  n8n-network:
    driver: bridge
```
## 6. Mac 环境预处理
在执行 docker-compose up 之前，请务必在终端执行以下命令：
### 1. 创建本地目录
mkdir -p ~/n8n-demo/n8n_data
mkdir -p ~/n8n-demo/files/tmp

### 2. 赋予最高权限（解决 Mac Bind Mount 权限问题）
chmod -R 777 ~/n8n-demo

> [!CAUTION]
> 如果不手动创建 files/tmp 并挂载到 /tmp，n8n 在执行某些二进制组件（如 Python Runner）时可能会因权限不足崩溃。
> 
## 7. 总结与 Tips
 * 生产环境：推荐使用 Docker Managed Volumes 提升性能。
 * 调试技巧：如果文件写不进去，先用 docker exec -it n8n_demo touch /n8n_files/test.txt 测试容器内写权限。
 * 更新提醒：n8n 版本更新较快，建议 Runner 和主镜像版本号保持一致。
   
Copyright © 2026 . Licensed under the MIT License.
