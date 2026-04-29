# Photo-to-Music iOS MVP 需求文档

## 1. 产品概述

本产品是一款 iOS App。用户上传或拍摄一张照片后，系统通过 AI 分析照片中的场景、情绪、色彩、时间感和空间感，然后推荐一首最适合当前照片氛围的 Apple Music 歌曲。

MVP 阶段只解决一个核心问题：

> 用户给一张照片，App 推荐一首可在 Apple Music 中播放或打开的歌曲。

本阶段不做 Android、不做 Web、不做社交 feed、不做视频配乐、不做自动生成原创音乐。

---

## 2. MVP 目标

### 2.1 核心目标

用户完成以下流程：

1. 打开 App。
2. 从相册选择一张照片，或使用相机拍摄一张照片。
3. App 上传压缩后的照片到后端。
4. 后端调用 AI 分析照片。
5. 系统生成音乐检索条件。
6. 系统搜索 Apple Music 候选歌曲。
7. 系统排序并返回最适合的一首歌。
8. App 展示歌曲封面、歌名、歌手、推荐理由和播放入口。

### 2.2 成功标准

MVP 可用标准：

- 用户从选图到看到推荐结果，目标耗时小于 10 秒。
- 推荐结果必须来自真实候选歌曲，不允许 AI 编造歌名。
- 至少支持 Apple Music 跳转。
- 已授权 Apple Music 的用户可以在 App 内尝试播放。
- 无 Apple Music 授权时，仍然能看到推荐结果和打开 Apple Music 的入口。
- 推荐失败时必须有 fallback 结果，不能直接空白。

---

## 3. 用户画像

### 3.1 主要用户

喜欢用照片记录场景、旅行、日落、城市、咖啡馆、海边、夜景的人。

### 3.2 典型场景

- 用户拍了一张海边日落照片，想知道现在适合听什么歌。
- 用户拍了一张城市夜景照片，想找一首适合发朋友圈、小红书或 Instagram 的背景音乐。
- 用户想通过照片获得一种“被理解”的音乐推荐体验。

---

## 4. 核心用户流程

```text
Launch App
↓
Home Screen
↓
Choose Photo / Take Photo
↓
Photo Preview
↓
Tap “Find a Song”
↓
Loading: Analysing the scene...
↓
Loading: Finding the right song...
↓
Result Screen
↓
Show one recommended Apple Music song
↓
User can:
  - Play / Open in Apple Music
  - Try another song
  - Like / Dislike
  - Save to History
```

---

## 5. 功能优先级

## P0：MVP 必须实现

### P0-1. 首页

首页需要包含：

- App 名称。
- 简短说明：`Upload a photo and get one song that matches the mood.`
- `Choose from Photos` 按钮。
- `Take Photo` 按钮。
- 最近一次推荐入口，可选。

验收标准：

- 用户可以从首页进入相册选择流程。
- 用户可以从首页进入拍照流程。
- 未授予照片权限时，系统应展示合理提示。

---

### P0-2. 图片输入

支持两种图片输入方式：

1. 从相册选择图片。
2. 使用相机拍摄图片。

iOS 实现建议：

- 使用 SwiftUI。
- 相册选择使用 `PhotosPicker`。
- 相机可使用 UIKit bridge 或系统相机封装。
- 只支持单张图片。
- 图片上传前需要压缩和 resize。

图片处理要求：

- 最长边压缩到 1024px 或 1280px。
- JPEG 压缩质量建议 0.75–0.85。
- 不在本地永久保存原图，除非用户明确保存历史记录。
- 上传后端时只上传压缩图。

验收标准：

- 用户能选择一张图片并进入预览页。
- App 可以显示用户选择的图片。
- 图片过大时不会导致 App 卡死。
- 上传的是压缩后的图片，而不是原图。

---

### P0-3. 图片预览页

用户选择照片后进入预览页。

页面元素：

- 图片预览。
- `Find a Song` 按钮。
- `Choose Another Photo` 按钮。
- 简短隐私提示：`Photo is used only to generate a music recommendation.`

验收标准：

- 点击 `Find a Song` 后开始推荐流程。
- 点击 `Choose Another Photo` 可重新选择。
- 推荐过程中按钮应进入 loading / disabled 状态，避免重复提交。

---

### P0-4. AI 场景分析

后端负责调用视觉 AI 模型。

输入：

- 压缩后的图片。

输出必须是结构化 JSON。

```json
{
  "scene_type": "string",
  "time_of_day": "string",
  "visual_elements": ["string"],
  "dominant_colours": ["string"],
  "mood_tags": ["string"],
  "energy_level": 0.0,
  "emotional_intensity": 0.0,
  "motion_level": 0.0,
  "human_presence": true,
  "music_unsuitable_tags": ["string"]
}
```

字段说明：

| 字段 | 说明 |
|---|---|
| `scene_type` | 照片场景类型，例如 `seaside landscape`、`city night`、`cafe interior` |
| `time_of_day` | `morning`、`afternoon`、`sunset`、`night`、`unknown` |
| `visual_elements` | 照片中主要元素 |
| `dominant_colours` | 主色调 |
| `mood_tags` | 情绪标签 |
| `energy_level` | 音乐能量需求，0 到 1 |
| `emotional_intensity` | 情绪强度，0 到 1 |
| `motion_level` | 画面动态感，0 到 1 |
| `human_presence` | 是否有人 |
| `music_unsuitable_tags` | 不适合的音乐方向 |

约束：

- 此 Agent 不允许推荐具体歌曲。
- 此 Agent 不允许输出散文式描述。
- 输出必须是可解析 JSON。

---

### P0-5. 视觉转音乐需求

后端根据场景 JSON 生成音乐检索条件。

输出字段：

```json
{
  "target_moods": ["string"],
  "preferred_genres": ["string"],
  "avoid_genres": ["string"],
  "tempo_range_bpm": [60, 90],
  "vocal_preference": "string",
  "instrumentation": ["string"],
  "english_search_queries": ["string"],
  "ranking_notes": "string"
}
```

示例：

```json
{
  "target_moods": ["calm", "cinematic", "melancholic", "expansive"],
  "preferred_genres": ["ambient", "post-rock", "modern classical"],
  "avoid_genres": ["EDM", "trap", "hard rock", "party pop"],
  "tempo_range_bpm": [55, 90],
  "vocal_preference": "instrumental or sparse vocal",
  "instrumentation": ["soft synth pad", "piano", "strings", "reverb guitar"],
  "english_search_queries": [
    "cinematic ambient sunset sea",
    "melancholic post rock ocean",
    "peaceful modern classical evening"
  ],
  "ranking_notes": "Prefer spacious, slow-building, low-energy tracks."
}
```

约束：

- 不允许编造歌名。
- 只生成搜索条件。
- Search query 应适合 Apple Music 目录搜索。

---

### P0-6. Apple Music 搜索

后端或 iOS 端需要根据音乐检索条件搜索 Apple Music 候选歌曲。

建议：

- MVP 阶段优先由后端统一处理 Apple Music 搜索，便于隐藏 API key、记录候选结果、调试排序。
- iOS 端只负责显示最终结果和发起播放 / 跳转。
- 搜索结果应至少返回 10–30 首候选歌曲。
- 候选歌曲必须包含 Apple Music 可用 ID 或 URL。

候选歌曲数据结构：

```json
{
  "id": "string",
  "title": "string",
  "artist": "string",
  "album": "string",
  "artwork_url": "string",
  "apple_music_url": "string",
  "genre": "string",
  "duration_ms": 0
}
```

约束：

- 推荐结果必须来自候选歌曲列表。
- 如果 Apple Music 搜索失败，进入 fallback 曲库。
- 不允许让 AI 直接输出一个未验证存在的歌名。

---

### P0-7. 候选歌曲排序

后端使用 Ranking Agent 从候选歌曲中选择一首。

输入：

- 图片场景 JSON。
- 音乐需求 JSON。
- Apple Music 候选歌曲列表。

输出：

```json
{
  "selected_song": {
    "id": "string",
    "title": "string",
    "artist": "string",
    "album": "string",
    "artwork_url": "string",
    "apple_music_url": "string",
    "genre": "string",
    "duration_ms": 0
  },
  "score_breakdown": {
    "mood_match": 0,
    "scene_match": 0,
    "energy_match": 0,
    "genre_match": 0,
    "user_preference_match": 0,
    "overused_penalty": 0
  },
  "reason": "string",
  "fallback_used": false
}
```

排序规则：

```text
final_score =
0.30 * mood_match +
0.25 * scene_match +
0.20 * energy_match +
0.15 * genre_match +
0.10 * user_preference_match -
0.10 * overused_penalty
```

约束：

- Ranking Agent 只能从候选列表里选。
- 推荐理由应短，不超过 40 个英文词，或不超过 80 个中文字符。
- 如果候选都不合适，返回 `fallback_used = true`。

---

### P0-8. 结果页

结果页展示一首歌。

页面元素：

- 用户上传的照片预览。
- 歌曲封面。
- 歌名。
- 歌手。
- 专辑名，可选。
- 推荐理由。
- `Play` 或 `Open in Apple Music` 按钮。
- `Try Another Song` 按钮。
- `Back to Home` 按钮。

验收标准：

- 用户可以看到明确的一首推荐歌曲。
- 用户可以跳转到 Apple Music。
- 如果有 MusicKit 授权，可以尝试 App 内播放。
- 如果没有授权，显示授权入口或 Apple Music 跳转入口。
- 点击 `Try Another Song` 时，不重新分析图片，只基于已有候选列表选择下一首。

---

### P0-9. Loading 状态

推荐流程需要有清晰的状态反馈。

状态文案：

```text
Analysing the scene...
Reading the mood...
Finding a song...
Almost there...
```

验收标准：

- 网络请求期间不能空白。
- 请求超过 10 秒时显示较长等待提示。
- 请求失败时显示可重试按钮。

---

### P0-10. 失败兜底

必须处理以下错误：

- 图片上传失败。
- AI 分析失败。
- JSON 解析失败。
- Apple Music 搜索失败。
- 没有候选歌曲。
- Apple Music 授权失败。
- 网络断开。
- 后端超时。

错误处理要求：

- 用户看到友好的错误提示。
- 提供 `Try Again` 按钮。
- Apple Music 搜索失败时使用 fallback 曲库。
- 不要展示原始技术错误给普通用户。

---

### P0-11. 本地历史记录

MVP 可以做简单本地历史记录。

保存内容：

```json
{
  "id": "uuid",
  "created_at": "datetime",
  "local_photo_thumbnail": "local path or data",
  "song_title": "string",
  "artist": "string",
  "artwork_url": "string",
  "apple_music_url": "string",
  "reason": "string"
}
```

约束：

- 只保存缩略图，不保存原始大图。
- 用户可以清空历史记录。
- 历史记录优先存在本地，不需要账号系统。

---

## P1：MVP 后增强功能

这些功能不阻塞第一版，但建议在 v0.2 加入。

### P1-1. 用户偏好设置

用户可以选择：

- 语言偏好：English / Chinese / No preference。
- 人声偏好：Instrumental / Vocal / No preference。
- 情绪偏好：Calm / Cinematic / Happy / Sad / Energetic / No preference。
- 熟悉度偏好：Popular / Balanced / Hidden gems。

这些偏好会进入 Ranking Agent。

---

### P1-2. 喜欢 / 不喜欢反馈

结果页加入：

- Like。
- Dislike。

保存反馈：

```json
{
  "recommendation_id": "uuid",
  "song_id": "string",
  "feedback": "like | dislike",
  "created_at": "datetime"
}
```

MVP 后续可用这些数据优化排序。

---

### P1-3. 分享卡片

用户可以生成分享图：

- 原图。
- 歌曲封面。
- 歌名。
- 歌手。
- 简短推荐语。

暂时只生成静态图片，不做复杂模板编辑。

---

## P2：暂不做

以下功能不进入 MVP：

- 用户账号系统。
- 云端同步。
- 社交 feed。
- 评论和点赞社区。
- 视频配乐。
- 一张图生成完整歌单。
- 多平台音乐支持。
- Spotify / YouTube Music / 网易云音乐。
- AI 原创音乐生成。
- 复杂个性化推荐模型。
- 订阅付费系统。
- 地理位置 / 天气上下文。

---

## 6. iOS 技术要求

### 6.1 技术栈

- Language：Swift。
- UI：SwiftUI。
- Architecture：MVVM 或 Clean Architecture light。
- Minimum iOS Target：iOS 17 优先，最低可根据实现调整。
- Image Selection：PhotosPicker。
- Camera：UIKit bridge 或自定义 CameraView。
- Music：MusicKit。
- Networking：URLSession async/await。
- Local Storage：SwiftData 或 UserDefaults + FileManager。
- Image Cache：简单本地缓存即可。
- Dependency Injection：轻量 protocol-based injection。

---

### 6.2 推荐项目结构

```text
PhotoMusicApp/
  App/
    PhotoMusicApp.swift
    AppState.swift

  Features/
    Home/
      HomeView.swift
      HomeViewModel.swift

    PhotoInput/
      PhotoPickerView.swift
      CameraPickerView.swift
      PhotoPreviewView.swift
      PhotoPreviewViewModel.swift

    Recommendation/
      LoadingView.swift
      ResultView.swift
      ResultViewModel.swift

    History/
      HistoryView.swift
      HistoryViewModel.swift

    Settings/
      SettingsView.swift
      SettingsViewModel.swift

  Core/
    Models/
      PhotoSceneAnalysis.swift
      MusicRequirement.swift
      SongCandidate.swift
      RecommendationResult.swift
      RecommendationHistoryItem.swift
      UserMusicPreference.swift

    Services/
      RecommendationAPIClient.swift
      ImageCompressionService.swift
      MusicAuthorizationService.swift
      MusicPlaybackService.swift
      HistoryStore.swift

    Networking/
      APIEndpoint.swift
      APIError.swift
      NetworkClient.swift

    Utilities/
      Logger.swift
      DateFormatterProvider.swift

  Resources/
    Assets.xcassets
    Localizable.xcstrings
```

---

## 7. 后端 API 需求

虽然当前只做 iOS，但 AI API key 和音乐 API token 不应该直接暴露在客户端。因此 MVP 需要一个轻量后端。

iOS 只调用自己的后端：

```text
POST /v1/recommendations
```

请求：

```json
{
  "image_base64": "string",
  "user_preference": {
    "language": "no_preference",
    "vocal": "no_preference",
    "mood": "no_preference",
    "familiarity": "balanced"
  }
}
```

响应：

```json
{
  "recommendation_id": "uuid",
  "scene_analysis": {
    "scene_type": "seaside landscape",
    "time_of_day": "sunset",
    "visual_elements": ["sea", "mountains", "pink sky"],
    "dominant_colours": ["pale blue", "soft pink", "dark navy"],
    "mood_tags": ["calm", "expansive", "nostalgic"],
    "energy_level": 0.25,
    "emotional_intensity": 0.55,
    "motion_level": 0.2,
    "human_presence": false,
    "music_unsuitable_tags": ["hard rock", "fast EDM"]
  },
  "selected_song": {
    "id": "string",
    "title": "Wait",
    "artist": "M83",
    "album": "Hurry Up, We're Dreaming",
    "artwork_url": "string",
    "apple_music_url": "string",
    "genre": "Electronic",
    "duration_ms": 343000
  },
  "reason": "The slow build and spacious atmosphere match the calm sea, distant mountains and sunset mood.",
  "fallback_used": false
}
```

错误响应：

```json
{
  "error": {
    "code": "recommendation_failed",
    "message": "We could not find a song for this photo. Please try again."
  }
}
```

---

## 8. Prompt 需求

### 8.1 Image Scene Agent Prompt

```text
You are a visual scene analysis agent for a music recommendation system.

Analyse the uploaded photo and return structured JSON only.

Do not recommend songs.
Do not write poetic descriptions.
Do not output Markdown.

Return:
- scene_type
- time_of_day
- visual_elements
- dominant_colours
- mood_tags
- energy_level: number from 0 to 1
- emotional_intensity: number from 0 to 1
- motion_level: number from 0 to 1
- human_presence: boolean
- music_unsuitable_tags

The output must be valid JSON.
```

---

### 8.2 Scene-to-Music Agent Prompt

```text
You are a scene-to-music translation agent.

Input is a JSON object describing a photo scene.
Convert the visual scene into music retrieval requirements.

Do not recommend specific songs.
Do not invent song titles.
Return JSON only.

Return:
- target_moods
- preferred_genres
- avoid_genres
- tempo_range_bpm
- vocal_preference
- instrumentation
- english_search_queries
- ranking_notes

Keep search queries short and suitable for Apple Music catalog search.
```

---

### 8.3 Ranking Agent Prompt

```text
You are a music recommendation ranking agent.

Input:
1. image_scene_json
2. music_requirement_json
3. candidate_songs

Choose exactly one song from candidate_songs.

Strict rules:
- Do not choose songs outside candidate_songs.
- Do not invent songs, artists, albums, or URLs.
- If no candidate is suitable, return fallback_needed: true.
- The recommendation reason must be concise.

Score each candidate using:
- mood_match: 0-10
- scene_match: 0-10
- energy_match: 0-10
- genre_match: 0-10
- user_preference_match: 0-10
- overused_penalty: 0-5

Return valid JSON only.
```

---

## 9. 数据模型

### 9.1 PhotoSceneAnalysis

```swift
struct PhotoSceneAnalysis: Codable {
    let sceneType: String
    let timeOfDay: String
    let visualElements: [String]
    let dominantColours: [String]
    let moodTags: [String]
    let energyLevel: Double
    let emotionalIntensity: Double
    let motionLevel: Double
    let humanPresence: Bool
    let musicUnsuitableTags: [String]
}
```

### 9.2 SongCandidate

```swift
struct SongCandidate: Codable, Identifiable {
    let id: String
    let title: String
    let artist: String
    let album: String?
    let artworkURL: URL?
    let appleMusicURL: URL
    let genre: String?
    let durationMs: Int?
}
```

### 9.3 RecommendationResult

```swift
struct RecommendationResult: Codable, Identifiable {
    let id: String
    let sceneAnalysis: PhotoSceneAnalysis
    let selectedSong: SongCandidate
    let reason: String
    let fallbackUsed: Bool
}
```

### 9.4 UserMusicPreference

```swift
struct UserMusicPreference: Codable {
    let language: MusicLanguagePreference
    let vocal: VocalPreference
    let mood: MoodPreference
    let familiarity: FamiliarityPreference
}
```

---

## 10. UI 页面需求

### 10.1 HomeView

内容：

- App title。
- Subtitle。
- `Choose from Photos`。
- `Take Photo`。
- Recent recommendation preview，可选。

状态：

- idle。
- permission denied。
- loading recent history。
- error。

---

### 10.2 PhotoPreviewView

内容：

- 选中图片。
- `Find a Song`。
- `Choose Another Photo`。
- 隐私提示。

状态：

- image loaded。
- image compressing。
- ready。
- error。

---

### 10.3 LoadingView

内容：

- Progress indicator。
- 动态文案。
- Cancel 按钮，可选。

状态文案：

- `Analysing the scene...`
- `Reading the mood...`
- `Searching Apple Music...`
- `Choosing the best match...`

---

### 10.4 ResultView

内容：

- 原图缩略图。
- 歌曲封面。
- 歌名。
- 歌手。
- 推荐理由。
- `Play / Open in Apple Music`。
- `Try Another Song`。
- `Like / Dislike`，可放 P1。
- `Save to History`，默认自动保存。

---

### 10.5 HistoryView

内容：

- 历史推荐列表。
- 每项显示缩略图、歌名、歌手、日期。
- 点击进入历史结果详情。
- `Clear History`。

---

### 10.6 SettingsView

内容：

- Apple Music authorization status。
- Music preference。
- Privacy。
- Clear History。
- App version。

---

## 11. 状态管理

推荐流程状态建议：

```swift
enum RecommendationFlowState {
    case idle
    case imageSelected
    case compressingImage
    case uploading
    case analysingScene
    case searchingMusic
    case ranking
    case success(RecommendationResult)
    case failure(AppError)
}
```

错误类型建议：

```swift
enum AppError: Error {
    case photoSelectionFailed
    case imageCompressionFailed
    case networkUnavailable
    case uploadFailed
    case recommendationFailed
    case musicAuthorizationDenied
    case appleMusicUnavailable
    case unknown
}
```

---

## 12. 隐私与数据处理

MVP 隐私原则：

- 默认不保存原始照片。
- 上传前压缩图片。
- 后端只将图片用于生成推荐。
- 如需日志，只记录匿名推荐 ID、场景标签、歌曲 ID、反馈，不记录可识别用户身份的信息。
- 历史记录默认保存在本地。
- 用户可以清空历史记录。
- 隐私政策需说明照片是否上传、保存多久、是否用于模型改进。

---

## 13. 非功能需求

### 13.1 性能

- 图片压缩不应明显卡住 UI。
- 推荐接口目标响应时间小于 10 秒。
- 图片上传失败时可以重试。
- ResultView 图片和封面加载应有 placeholder。

### 13.2 稳定性

- AI 输出 JSON 解析失败时，后端应自动重试或进入 fallback。
- Apple Music 搜索失败时，使用 fallback 曲库。
- 网络失败时显示 retry。
- App 重启后历史记录仍在。

### 13.3 安全

- 不在 iOS 客户端硬编码 AI API key。
- 不在 iOS 客户端硬编码敏感 Apple Music 服务端密钥。
- 网络请求使用 HTTPS。
- 日志中不要记录 base64 图片内容。

---

## 14. Fallback 曲库

MVP 需要一个小型 fallback 曲库，避免搜索失败。

建议先人工准备 100–200 首歌，按场景标注：

```json
{
  "track": "Wait",
  "artist": "M83",
  "genres": ["cinematic electronic", "dream pop"],
  "moods": ["expansive", "melancholic", "slow build"],
  "scenes": ["sunset", "ocean", "travel", "night"],
  "energy": 0.35,
  "vocal_density": 0.3,
  "language": "english",
  "apple_music_url": "string"
}
```

基础场景标签：

- ocean。
- sunset。
- city night。
- mountain。
- cafe。
- rain。
- snow。
- road trip。
- bedroom。
- party。
- forest。
- street。

---

## 15. MVP 验收清单

### 15.1 产品检查

- 可以选择照片。
- 可以拍照。
- 可以看到照片预览。
- 可以点击 `Find a Song`。
- 可以看到 loading 状态。
- 可以得到一首歌曲推荐。
- 可以看到推荐理由。
- 可以打开 Apple Music。
- 可以重新推荐另一首。
- 可以查看历史记录。
- 可以清空历史记录。

### 15.2 技术检查

- SwiftUI 项目结构清晰。
- 网络层独立。
- 图片压缩服务独立。
- MusicKit 授权服务独立。
- 本地历史存储独立。
- ViewModel 不直接写网络请求细节。
- 所有 API response 都有 Codable model。
- 错误状态有 UI 展示。
- 没有把 API key 放在客户端。

### 15.3 推荐逻辑检查

- AI 场景分析不直接推荐歌曲。
- 音乐需求生成不编造歌名。
- Ranking Agent 只能从候选列表选择。
- 推荐结果必须包含 Apple Music URL。
- 搜索失败时 fallback 生效。

---

## 16. 给 Codex 的执行指令

请基于以上需求，为一个 iOS-only MVP 生成产品架构和初始代码结构。

要求：

1. 使用 SwiftUI。
2. 使用 MVVM。
3. 只实现 iOS App，不做 Android 和 Web。
4. 保留后端 API client 抽象，但不要在 iOS 端实现真实 AI key。
5. 创建清晰的 Models、Services、ViewModels、Views。
6. 使用 mock recommendation response 先跑通 UI。
7. 网络层支持未来接入 `POST /v1/recommendations`。
8. MusicKit 播放和授权先做 service abstraction。
9. PhotosPicker 和图片压缩必须真实实现。
10. 历史记录可以先使用本地 JSON 文件或 SwiftData。
11. 所有 UI 必须有 loading、success、failure 三种状态。
12. 不要实现 P2 功能。
13. 不要添加账号系统。
14. 不要添加订阅系统。
15. 不要添加社交功能。

第一步请输出：

- App architecture。
- Folder structure。
- Core data models。
- Service protocols。
- Main SwiftUI views。
- ViewModel responsibilities。
- API contract。
- Implementation order。

第二步再生成代码。

---

## 17. 建议的开发顺序

1. 创建 SwiftUI 项目结构。
2. 实现 Models。
3. 实现 mock `RecommendationAPIClient`。
4. 实现 `HomeView`。
5. 实现 `PhotosPicker`。
6. 实现图片压缩。
7. 实现 `PhotoPreviewView`。
8. 实现 `LoadingView`。
9. 实现 `ResultView`。
10. 实现 `HistoryStore`。
11. 实现 `HistoryView`。
12. 实现 `MusicAuthorizationService`。
13. 实现 Apple Music 跳转。
14. 接入真实后端 API。
15. 接入真实 MusicKit 播放。
16. 加入错误处理和 fallback UI。
17. TestFlight 前做隐私和权限文案检查。

---

## 18. 核心架构约束

> iOS 端只负责输入、展示、播放和本地状态；AI 分析、Apple Music 候选检索、排序和 fallback 逻辑由后端完成。

这样可以避免把推荐逻辑和 UI 混在一起，也可以防止 API key 暴露在客户端。
