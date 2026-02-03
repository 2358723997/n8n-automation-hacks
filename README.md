# 🚀 n8n Automation Hacks

> 专注于自托管自动化工具 n8n 的实战沉淀。解决 Mac/Docker 环境下的权限地狱，分享高级工作流。

---
 ## 📂 核心指南 (Guides)

* **[🔥 必读] [n8n + Runner + Mac 部署踩坑完整指南](./docs/n8n-mac-guide.md)**
  * *解决 Read/Write 权限报错、临时目录 Socket 及代理拦截问题。*
* **[💡 实用] [n8n Webhook 调试小 Tips 总结](./docs/n8n-tips.md)**
  * *先让数据“自动流到底”，再切换成“手动控制返回”——避坑 80% 的 webhook 响应问题。*

---

## 🛠️ 快速开始

请在终端（Terminal）逐行运行以下命令：

### 1. 克隆仓库
```bash
git clone [https://github.com/2358723997/n8n-automation-hacks.git](https://github.com/2358723997/n8n-automation-hacks.git)

2. 进入目录
cd n8n-automation-hacks

3. 环境预处理 (仅 Mac 用户)
mkdir -p ./files/tmp ./n8n_data && chmod -R 777 ./files ./n8n_data

4. 启动服务
docker-compose up -d
```

🗺️ 路线图 (Roadmap)
 * [x] n8n Mac Runner 权限填坑指南
 * [ ] 自动化数据库备份工作流模版
 * [ ] AI 自动化 Agent 落地实战
🤝 交流与反馈
 * 提交 Issue: 欢迎提交 New Issue
 * License: MIT
