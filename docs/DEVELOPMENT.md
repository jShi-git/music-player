# 开发文档

## 环境

- Swift 6 / Package tools 6.0，最低支持 macOS 14.0
- 打包脚本使用 `swift build`、`ditto` 与 `codesign`（ad-hoc 签名）
- 元数据写入依赖本机 ffmpeg（`brew install ffmpeg`）

## 项目结构

```
MusicDownloaderMac/
├── Package.swift
├── Sources/
│   ├── MusicCore/              # 与界面无关的核心层
│   │   ├── Models.swift        # Song / MusicSource / 下载范围等模型
│   │   ├── HTTPClient.swift    # URLSession 封装、超时与重试策略
│   │   ├── KuwoClient.swift    # 酷我搜索、歌单、歌词、下载地址解析
│   │   ├── KugouClient.swift   # 酷狗搜索、歌单、歌词、下载地址解析
│   │   ├── NeteaseClient.swift # 网易云搜索、歌单、歌词、外链解析
│   │   ├── QishuiClient.swift  # 汽水音乐搜索、歌单 / 单曲解析、歌词、音频地址
│   │   ├── SearchService.swift # 并发多源搜索、歌单解析
│   │   ├── QQPlaylistService.swift # QQ 歌单解析、多源逐首匹配
│   │   ├── ArtistDirectoryService.swift # QQ 音乐歌手搜索与头像补全
│   │   ├── OnlinePlaylistService.swift # 各平台在线歌单目录、公开歌单搜索与曲目读取
│   │   ├── PlaylistImportService.swift # 歌单链接解析 / QQ 逐首匹配的统一入口
│   │   ├── QQChartService.swift # QQ 榜单拉取与多源逐首匹配
│   │   ├── AppleMusicChartService.swift # Apple Music 地区榜单拉取与多源逐首匹配
│   │   ├── SpotifyChartService.swift # Spotify 公开歌单元数据拉取与多源逐首匹配
│   │   ├── RadioService.swift  # 榜单拉取、情景关键词检索与随机抽取
│   │   ├── InternetRadio.swift # Radio Browser 网络电台目录、分类与直播流筛选
│   │   ├── LyricsMatcher.swift # Audius 歌词匹配（LRCLib + 原平台兜底）
│   │   ├── DownloadService.swift # 下载、试听检测、歌词写入、命名
│   │   ├── AudioTagReader.swift # 标签与内嵌封面读取（含 FLAC 图片解析）
│   │   ├── MetadataEnricher.swift # 网易云 / QQ 音乐匹配缺失元数据，酷我 / 酷狗兜底补全
│   │   ├── MetadataWriter.swift # 调用 ffmpeg 写回标签与封面
│   │   ├── LooseJSON.swift     # 兼容单引号伪 JSON 的宽松解析器
│   │   └── TextUtils.swift     # HTML 实体、GB18030、文件名清洗等
│   ├── MusicDownloaderApp/     # SwiftUI 界面层
│   │   ├── App.swift
│   │   ├── AppViewModel.swift
│   │   ├── ContentView.swift
│   │   ├── Theme.swift         # 深色主题与通用组件
│   │   ├── AddSongMenu.swift   # 统一的「＋」复合菜单（队列 / 歌单）
│   │   ├── PlaybackModels.swift # 播放条目、歌词解析、音乐库曲目模型
│   │   ├── PlayerService.swift  # AVPlayer 播放引擎、队列与控制
│   │   ├── MusicStore.swift     # 收藏、历史、自定义歌单与本地持久化
│   │   ├── LibraryStore.swift   # 本地音乐库扫描、元数据与歌手头像缓存
│   │   ├── SearchPage.swift     # 搜索与下载页
│   │   ├── DownloadManager.swift # 后台下载队列、并发与暂停 / 继续
│   │   ├── DownloadPage.swift   # 下载任务页
│   │   ├── LibraryPage.swift    # 本地音乐库页（左侧两列歌手卡片 + 右侧歌曲 / 专辑视图）
│   │   ├── MetadataRepairPage.swift # 整理元数据独立页（后台扫描 / 写入与进度）
│   │   ├── MinePage.swift       # 喜欢 / 最近页与共用的歌曲列表组件
│   │   ├── PlaylistPage.swift   # 歌单页（我的歌单 / 在线歌单切换与歌单详情）
│   │   ├── DiscoverPage.swift   # 在线歌单浏览器（左侧歌单列表、右侧详情与就地解析）
│   │   ├── OnlinePlaylistStore.swift # 在线歌单目录缓存、收藏与加载状态
│   │   ├── PlaylistTaskCenter.swift # 可收起的歌单后台任务面板与进度
│   │   ├── RadioModels.swift    # 情景电台场景定义与配色
│   │   ├── RadioStore.swift     # 电台状态、榜单 / 情景 / 网络电台加载与刷新
│   │   ├── ScenarioPage.swift   # 情景页（默认极简情景网格 + 热门高亮）
│   │   ├── ChartPage.swift      # 排行榜独立页：榜单前列歌曲与单独刷新
│   │   ├── RadioPage.swift      # 网络电台页（分类与直播台列表）
│   │   ├── MiniPlayerView.swift # 底部迷你播放器
│   │   ├── PlaybackQueuePanel.swift # 右侧播放队列侧栏
│   │   ├── NowPlayingView.swift # 沉浸式播放页与歌词面板
│   │   └── SettingsView.swift   # 设置面板
│   └── MDLSmoke/               # 命令行冒烟测试工具
├── scripts/
│   ├── Info.plist
│   ├── build_app.sh
│   ├── AppIcon.icns
│   ├── icon_master.png
│   └── make_icon.swift        # 图标打包脚本（由 icon_master.png 生成 AppIcon.icns）
└── docs/
```

## 实现说明

- 各平台的搜索、歌单、歌词接口均按其 Web / 移动端接口实现；酷我返回的是单引号伪 JSON，核心层内置了宽松解析器。
- 汽水音乐通过公开目录接口搜索，匿名访问时会员曲目只能获取约 30 秒试听，免费曲目可下载完整 M4A；歌词为 KRC 格式，已转换为标准 LRC。
- 汽水音乐歌单通过 `/luna/pc/playlist/detail` 一次读取公开可播放曲目；`qishui.douyin.com/s/…` 短链会自动跳转后识别歌单或单曲。音频地址优先取 `seo_track` 返回的 `video_model`，缺失时回退同一详情的 VOD `url_player_info`。
- 汽水会员权益通过账号 Cookie 接入：解析时优先使用账号播放接口返回的完整音频，并与歌曲标注时长比对以识别试听片段；账号信息通过 `luna/pc/me` 校验。Cookie 仅保存在本机。
- 酷我无损下载优先通过第三方解析接口获取 FLAC，再回退到官方 MP3 试听地址；下载完成后用 AVFoundation 校验实际时长，明显短于标注时长时判定失败。
- 文件名中的非法字符（`\ / * ? : " < > |`）统一替换为 `_`；酷我歌词为 XOR + zlib + GB18030 编码，已按原算法解密。
- 网络层使用共享 Cookie 与缓存，酷狗、网易云接口所需的 Referer / Cookie 头均已处理。
- 播放基于 AVFoundation：在线曲目播放前先调用各平台解析接口获取音频地址，本地曲目直接播放文件；歌词按时间轴二分查找当前行并自动滚动。
- Audius 官方接口不提供歌词：播放时按歌名 / 歌手 / 时长先查 LRCLib，仍未命中时依次在网易云、酷狗、酷我中匹配同曲再取歌词；匹配结果在本次运行内缓存。
- 本地音乐库直接读写保存目录，封面优先取缓存中的在线封面，其次读取音频文件内嵌封面；无内嵌封面的旧文件显示占位图。
- 整理元数据时，每首缺失标签的歌曲会并行查询网易云与 QQ 音乐，按标题、歌手、时长打分选优；标题完全无重叠的候选会被排除。若仍有缺失字段，再查询酷我、酷狗并按一致性合并字段。
- 写入通过 ffmpeg 完成（`-map_metadata 0 -c:a copy`），写入前先生成临时文件并用 AVFoundation 校验时长，通过后才替换原文件，同时保留原修改时间。
- 情景电台按场景关键词在多个平台检索并随机组合，按「歌名 + 首位歌手」跨源去重，并过滤伴奏 / KTV 版本；榜单电台每次随机抽取 20 首，队列末尾自动续播。
- QQ 榜单通过公开榜单接口读取曲目，再按歌名、歌手、时长在已勾选音源中逐首匹配；Spotify 榜单从 `open.spotify.com/embed/playlist/{id}` 的公开页面读取 Top 50 元数据，同样逐首匹配；Apple Music 美区榜单优先读取 iTunes Top Songs，异常时回退 Apple Marketing Tools。
- 收藏、历史与歌单以 JSON 持久化在应用支持目录；歌曲按「来源 + 曲目 ID」去重，历史超过 300 首自动截断。
- SwiftUI `Menu` 的内容会被重新托管到独立环境，无法读取 `@EnvironmentObject`；因此 `AddSongMenu` 通过初始化参数显式接收 `player`、`music`、`vm`，避免在 `Table` 行内展开菜单时崩溃。

## 命令行调试工具

项目附带 `mdl-smoke`，用于验证各平台接口是否可用：

```bash
swift build -c debug --product mdl-smoke

./.build/debug/mdl-smoke search kuwo 晴天 5
./.build/debug/mdl-smoke download kugou 晴天 /tmp/music
./.build/debug/mdl-smoke search qishui 晴天 5
./.build/debug/mdl-smoke playlist "https://music.163.com/#/playlist?id=3778678"
./.build/debug/mdl-smoke qqplaylist "https://y.qq.com/n/ryqq/playlist/9262930475" 10
./.build/debug/mdl-smoke qqmatch "https://y.qq.com/n/ryqq/playlist/9262930475" 8
./.build/debug/mdl-smoke radio chart kuwo-hot
./.build/debug/mdl-smoke radio scene 深夜 治愈
./.build/debug/mdl-smoke netradio pop 8
./.build/debug/mdl-smoke netradio chinese 8
./.build/debug/mdl-smoke fallback kugou 晴天
./.build/debug/mdl-smoke lyrics netease 晴天
./.build/debug/mdl-smoke lyrics match "Blinding Lights" "The Weeknd" 200
./.build/debug/mdl-smoke enrich /tmp/music-test            # 只预览建议，不写入
./.build/debug/mdl-smoke enrich /tmp/music-test --apply    # 写入标签与封面
./.build/debug/mdl-smoke enrich /tmp/music-test --limit 5 --no-cover
```

`search` 支持 `kuwo`、`kugou`、`netease`、`qishui`、`audius` 五个平台；`lyrics` 用 `match <标题> <歌手> [时长秒]` 可直接验证 Audius 的歌词匹配链路。

榜单 ID 可用：`kuwo-hot`、`kuwo-new`、`kuwo-rising`、`netease-hot`、`netease-rising`、`kugou-top500`、`qq-hot`、`qq-new`、`qq-rising`、`apple-us`、`spotify-global`、`spotify-us`、`spotify-uk`、`spotify-jp`、`qishui-hot`、`qishui-new`、`qishui-global`、`audius-trending`。

`netradio` 的分类 ID 可用：`pop`、`chinese`、`rock`、`electronic`、`hiphop`、`jazz`、`classical`、`chillout`，不带参数时会列出全部分类。