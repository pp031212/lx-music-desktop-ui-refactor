# Any Listen 播放详情页 UI 移植进展记录

日期：2026-03-23  
仓库：`lx-music-desktop`

## 1. 本次目标

在不改动 `lx-music-desktop` 歌词内核、桌面歌词工程和设置语义的前提下，将主窗口播放详情页的视觉与布局朝 `any-listen` 的播放详情页风格靠拢。

本次工作定位：

- 属于主窗口播放详情页 UI 重构
- 不属于歌词系统迁移
- 不包含 `renderer-lyric` 独立桌面歌词窗口改造

## 2. 已完成内容

### 2.1 页面主结构重排

已完成文件：

- `src/renderer/components/layout/PlayDetail/index.vue`

已完成项：

- 将播放详情页重构为更清晰的三段式布局：
  - 顶部 Header
  - 中部主内容区
  - 底部控制区
- 将中部主内容区强化为左侧信息卡片 + 右侧歌词区的结构
- 评论面板继续保持绝对定位展开，但重新调整了展开时的比例关系

### 2.2 左侧封面与歌曲信息区改造

已完成文件：

- `src/renderer/components/layout/PlayDetail/index.vue`

已完成项：

- 新增更强的卡片化容器
- 封面区域改为更接近唱片感的表现
- 增加封面中心轴点
- 增加封面播放时缓慢旋转效果
- 无封面状态增加占位视觉
- 歌曲名、歌手、专辑信息层级重排

### 2.3 歌词区视觉迁移

已完成文件：

- `src/renderer/components/layout/PlayDetail/LyricPlayer.vue`

已完成项：

- 保留原有歌词逻辑、菜单、跳播、选择功能
- 重做歌词区留白和 padding 结构
- 强化歌词文本阴影和层次感
- 调整激活歌词的放大与过渡
- 调整跳播提示线与按钮的布局
- 优化歌词选择态展示样式

未改动内容：

- 歌词滚动逻辑
- 歌词同步逻辑
- 右键菜单逻辑
- 偏移保存逻辑

### 2.4 底部控制栏重排

已完成文件：

- `src/renderer/components/layout/PlayDetail/PlayBar.vue`
- `src/renderer/components/layout/PlayDetail/components/ControlBtns.vue`

已完成项：

- 将底部控制栏重排为中置播放按钮 + 下方进度条/时间信息
- 主播放按钮做了更强的视觉强调
- 左侧功能按钮统一为圆形玻璃感按钮
- 进度条与时间信息层级重新整理

### 2.5 Header 视觉统一

已完成文件：

- `src/renderer/components/layout/PlayDetail/ControlBtnsLeftHeader.vue`
- `src/renderer/components/layout/PlayDetail/ControlBtnsRightHeader.vue`

已完成项：

- 左侧圆点式控制按钮改得更贴近新页面风格
- 右侧标题栏按钮 hover 表现与页面新视觉协调

### 2.6 评论面板视觉统一

已完成文件：

- `src/renderer/components/layout/PlayDetail/components/MusicComment/index.vue`

已完成项：

- 评论面板改为更强的卡片感与玻璃感容器
- 调整评论面板头部按钮样式
- 调整 tab 头部与激活态样式
- 调整滚动区域边距和分页区间距
- 调整评论展开时主页面比例

### 2.7 细节回修：封面比例与歌曲信息可见性

已完成文件：

- `src/renderer/components/layout/PlayDetail/index.vue`

已完成项：

- 修正左侧专辑封面在当前移植样式下被拉伸成椭圆的问题
- 将封面容器改为明确的等宽高尺寸，避免依赖宿主环境对 `aspect-ratio` 的兼容表现
- 保持封面仍为圆形唱片视觉，不再出现“圆形图片塞进圆角矩形后被压扁”的情况
- 调整歌曲名、歌手、专辑信息区展示方式，移除纵向滚动依赖
- 长文本改为单行展示 + 横向滚动，避免信息被挤到下方后还需要继续往下滚动
- 保持左侧信息区整体更紧凑，进入页面时即可完整看到核心歌曲信息结构

### 2.8 细节回修：信息区滚动表现与封面卡片伪影处理

已完成文件：

- `src/renderer/components/layout/PlayDetail/index.vue`
- `src/renderer/components/layout/PlayDetail/PlayBar.vue`

已完成项：

- 将歌曲名、歌手、专辑的长文本展示从显式横向滚动条改为自动滚动
- 自动滚动策略从左右往返调整为单向循环跑马灯
- 保持仅在文本实际溢出时才启用自动滚动，不溢出时保持静止
- 修正短文本场景下因边缘渐变遮罩导致文字被吃掉一部分的问题
- 恢复歌手 / 专辑说明标签，避免信息层级变得过于弱化
- 处理左侧 `mediaCard` 容器出现粗条伪影的问题，将该层收敛为纯布局容器
- 去除播放 / 上一曲 / 下一曲三个主控按钮外围的圆形底与描边，只保留图标主体与交互反馈

### 2.9 细节回修：封面样式向 any-listen 靠拢

已完成文件：

- `src/renderer/components/layout/PlayDetail/index.vue`

已完成项：

- 对照 `any-listen` 的 `CoverCD.svelte` 重做左侧唱片结构
- 将原来的“圆图 + 光晕 + 厚矩形外壳”方案调整为更接近 `any-listen` 的“薄方形底板 + SVG CD”方案
- 封面图改为通过 SVG `mask + foreignObject` 的方式嵌入唱片结构
- 调整整张 CD 的旋转逻辑，使其更接近参考实现
- 收敛外层装饰，保留四角小圆点与轻量底板

### 2.10 细节回修：底部控制与歌词菜单增强

已完成文件：

- `src/renderer/components/layout/PlayDetail/PlayBar.vue`
- `src/renderer/components/layout/PlayDetail/components/LyricMenu.vue`
- `src/renderer/components/layout/PlayDetail/LyricPlayer.vue`
- `src/common/defaultSetting.ts`
- `src/common/types/app_setting.d.ts`
- `src/renderer/store/setting.ts`

已完成项：

- 将原来位于播放键左侧的一组功能按钮移到右侧
- 统一播放 / 上一曲 / 下一曲三键的默认颜色表现
- 为播放详情页歌词新增“加粗歌词”选项
- 加粗歌词入口放在歌词右键菜单中
- 新增 `playDetail.style.fontWeight` 设置项，默认启用
- 将该设置接入详情页歌词正常态与歌词选择态样式

### 2.11 评论区联动动画尝试与当前保留方案

已完成文件：

- `src/renderer/components/layout/PlayDetail/index.vue`

已完成项：

- 评论区打开时，歌词区会为评论面板预留右侧空间，不再完全被评论面板遮挡
- 为主内容区 `padding-right` 增加过渡，缓和评论区展开时的突兀感

尝试但已回退：

- 曾尝试将评论区改为真正参与主布局的右侧栏位，实现“双栏联动”宽度动画
- 该方案导致评论组件在未展开时仍可能以最小宽度参与占位，进而压缩默认歌词区
- 该方案目前已回退，当前保留的是“绝对定位评论面板 + 主内容区 padding-right 让位”的实现

### 2.12 歌词细节回修：加粗开关与缩放裁切修正

已完成文件：

- `src/renderer/components/layout/PlayDetail/components/LyricMenu.vue`
- `src/renderer/components/layout/PlayDetail/LyricPlayer.vue`
- `src/common/defaultSetting.ts`
- `src/common/types/app_setting.d.ts`
- `src/renderer/store/setting.ts`

已完成项：

- 在歌词右键菜单中新增“加粗歌词”开关
- 为播放详情页新增 `playDetail.style.fontWeight` 设置项，默认启用
- 将加粗设置接入详情页歌词正常态与歌词选择态
- 修正开启“缩放当前正在播放的歌词”后，当前歌词在左对齐场景下向左挤出可视区的问题
- 根据歌词对齐方式动态调整激活歌词缩放时的 `transform-origin`
- 为激活歌词增加额外横向安全边距，缓解居中模式下的轻微裁切

## 3. 明确未动的部分

以下内容本次未改：

- `src/renderer-lyric/*` 独立桌面歌词工程
- 主进程桌面歌词窗口逻辑
- 歌词解析与缓存
- 歌词同步算法
- 歌词偏移数据结构
- 快捷键系统
- 设置项的数据结构与语义

## 4. 当前结论

### 4.1 已达到的状态

当前主窗口播放详情页已经完成一版较完整的 `any-listen` 风格移植，主要体现在：

- 结构更现代
- 封面区更有表现力
- 左侧信息卡片的比例更稳定
- 歌曲基础信息首屏可见性更强
- 长文本信息展示更接近播放器场景的自动流动效果
- 主播放控制区视觉更轻，更接近图标化表达
- 左侧唱片样式更接近 any-listen 的 CD 结构
- 歌词菜单具备更完整的展示控制能力
- 歌词放大效果与不同对齐方式的兼容性更好
- 歌词区更聚焦
- 底部控制区更统一
- 评论区与整体页面不再割裂

### 4.2 仍需确认的部分

由于当前目录内没有 `node_modules`，本次未完成实际运行验证，因此以下内容仍需在运行后确认：

- 页面实际渲染结果
- 评论展开时的真实布局表现
- 不同歌词模式下的视觉表现
- 无歌词 / 无封面 / 长歌词等边界场景
- 全屏模式下的细节

## 5. 后续建议

### 建议优先做的下一步

1. 安装依赖并启动项目，做一轮实际视觉回归
2. 对以下场景逐项检查：
   - 有封面 + 有歌词
   - 无封面 + 有歌词
   - 有封面 + 无歌词
   - 翻译歌词 / 罗马音歌词
   - 逐字歌词
   - 评论区展开
   - 全屏模式
   - 不同字体大小与对齐方式

### 可选的后续增强

- 进一步提高封面区细节，向 `any-listen` 的 CD 风格继续靠拢
- 评估是否为播放详情页引入更明显的动态背景可调策略
- 评估是否为 `renderer-lyric` 独立桌面歌词工程设计同风格版本

## 6. 本次涉及文件

- `src/renderer/components/layout/PlayDetail/index.vue`
- `src/renderer/components/layout/PlayDetail/LyricPlayer.vue`
- `src/renderer/components/layout/PlayDetail/PlayBar.vue`
- `src/renderer/components/layout/PlayDetail/components/ControlBtns.vue`
- `src/renderer/components/layout/PlayDetail/ControlBtnsLeftHeader.vue`
- `src/renderer/components/layout/PlayDetail/ControlBtnsRightHeader.vue`
- `src/renderer/components/layout/PlayDetail/components/MusicComment/index.vue`

## 7. 风险备注

- 当前改动主要集中在样式和布局层，逻辑层改动较少，整体风险可控
- 但 `LyricPlayer.vue` 与现有 DOM 结构绑定较深，仍需实际运行确认没有影响滚动和跳播
- 评论面板宽度逻辑仍保留旧实现方式，运行后可能需要进一步微调
- 标题、歌手、专辑当前采用自动跑马灯策略，仍需实际运行确认滚动速度、启停时机与可读性是否合适
- 去除主播放按钮外圈后，仍需确认在不同背景亮度下图标辨识度是否足够
- 评论区展开动画目前为折中方案，已缓和但仍不是理想的联动宽度动画，后续如继续优化需重新设计评论组件的布局参与方式
- 激活歌词缩放的安全边距目前采用经验值，后续仍可根据实际视觉效果继续微调

---

文档状态：已保存  
用途：移植进度记录 / 后续回归参考
