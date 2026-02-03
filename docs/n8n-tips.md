# 1. n8n Webhook 调试小 Tips 总结

在搭建或修改 webhook 响应逻辑时，别一开始就直接使用 `"Using 'Respond to Webhook' node"` 模式，否则容易卡在 “No Webhook node found” 或响应不返回的坑里。

## 推荐调试顺序（最省心）

1. 先把 Webhook 节点的 Response Mode 设为 **When Last Node Finishes**（或 **Immediately**）
2. 使用 Postman / 前端反复发送真实请求，确认：
   - body 数据能正确传入
   - 每个节点输出符合预期
   - 最后一个节点的 JSON 就是你想要返回的格式
3. 全部跑通、格式无误后，再改为 **Using 'Respond to Webhook' node**
4. 最后在流程末尾添加 **Respond to Webhook** 节点，精细设置响应体

> 💡 一句话口诀：
>
> **先让数据“自动流到底”看到结果，再切换成“手动控制返回”**

这个顺序能让你快速验证通路，少踩 80% 的响应相关坑。
