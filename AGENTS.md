# AGENTS.md — AI 行为规则

## 项目概述

Corfu 是一款 iOS 应用，通过拍照或上传图片，由 AI 分析画面情绪与场景，自动推荐匹配的音乐。

## 目录约定

| 目录 | 职责 |
|------|------|
| `ios_app/` | iOS 主工程，Swift/SwiftUI |
| `services/vision/` | 图像理解逻辑（调用视觉模型） |
| `services/recommendation/` | 音乐推荐逻辑 |
| `services/prompt/` | Prompt 模板，禁止硬编码在业务代码中 |
| `docs/` | 需求、架构、接口文档 |
| `scripts/` | 自动化脚本，不含业务逻辑 |

## AI 协作规则

1. **修改前先读文档**：理解 `docs/requirements.md` 再动手。
2. **Prompt 集中管理**：所有 prompt 只存放在 `services/prompt/`，不散落在其他文件。
3. **不越权修改**：未经明确指令，不改动 `docs/` 下的需求与架构文档。
4. **小步提交**：每次只改一个功能点，保持 diff 可读。
5. **不生成占位代码**：不写 `TODO`、空函数体、假数据来"假装完成"。
