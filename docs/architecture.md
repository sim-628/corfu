# 架构设计

> 待补充。参考 `requirements.md` 确定技术选型后填写。

## 草稿

- iOS 客户端（SwiftUI）→ 本地拍照 / 相册选图
- 图像上传 → `services/vision/` 调用视觉模型解析场景与情绪
- 情绪标签 → `services/recommendation/` 匹配音乐
- 结果返回客户端播放
