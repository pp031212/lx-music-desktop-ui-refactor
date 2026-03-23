# Any Listen 播放详情页 UI 移植计划书

## 1. 目标

将 `any-listen` 的播放详情页 / 歌词播放页视觉设计移植到 `lx-music-desktop` 主窗口的播放详情页中，在不破坏 `lx-music-desktop` 现有歌词能力、设置项、平台兼容性的前提下，提升页面的现代感、层次感与可读性。

本次移植的目标是 **UI 重构**，不是歌词系统迁移。

## 2. 范围界定

### 2.1 本次纳入范围

- `src/renderer/components/layout/PlayDetail/index.vue`
- `src/renderer/components/layout/PlayDetail/LyricPlayer.vue`
- `src/renderer/components/layout/PlayDetail/PlayBar.vue`
- `src/renderer/components/layout/PlayDetail/components/*` 中与播放详情页视觉呈现直接相关的组件
- 播放详情页所需的样式变量、布局拆分、视觉状态调整

### 2.2 本次不纳入范围

- `src/renderer-lyric/*` 独立桌面歌词工程
- 主进程歌词窗口逻辑
- 歌词解析、缓存、同步、偏移算法
- 音频可视化底层实现
- 快捷键系统
- 设置存储结构与已有配置项语义

结论：

- 第一阶段只重做主窗口播放详情页 UI
- 桌面歌词窗口只做兼容性确认，不做视觉移植

## 3. 参考源

### 3.1 参考实现

`any-listen` 参考文件：

- `packages/view-main/src/components/layout/PlayDetail/index.svelte`
- `packages/view-main/src/components/layout/PlayDetail/Main.svelte`
- `packages/view-main/src/components/layout/PlayDetail/LeftInfo/*`
- `packages/view-main/src/components/layout/PlayDetail/RightLyric/*`
- `packages/view-main/src/components/layout/PlayDetail/Footer/*`

### 3.2 被改造目标

`lx-music-desktop` 目标文件：

- `src/renderer/components/layout/PlayDetail/index.vue`
- `src/renderer/components/layout/PlayDetail/LyricPlayer.vue`
- `src/renderer/components/layout/PlayDetail/PlayBar.vue`

## 4. 移植原则

### 4.1 保留 `lx` 既有能力

必须继续兼容以下功能：

- 逐行歌词
- 逐字歌词
- 翻译歌词 / 罗马音歌词
- 歌词偏移调节
- 歌词拖拽定位与滚动跳播
- 歌词右键菜单
- 评论区切换
- 全屏模式下的行为
- 已有设置项对播放详情页的控制

### 4.2 只迁移展示层

移植时优先复制以下设计语言，而不是复制实现方式：

- 左右布局比例
- 封面区样式
- 动态背景层次
- 歌词区留白与层次
- 激活歌词的视觉强调
- 页头、页脚排布
- 渐入过渡和整体动效节奏

### 4.3 不改变现有数据模型

新 UI 必须适配 `lx-music-desktop` 已有状态结构，不能为了接近 `any-listen` 而重构播放器状态流。

## 5. 兼容性要求

### 5.1 功能兼容

以下行为移植后必须与当前版本保持一致：

- 歌词滚动与当前行定位正确
- 歌词跳播功能正常
- 歌词菜单可打开且保存逻辑不变
- 评论展开后页面仍可正常布局
- 全屏进入 / 退出无明显布局错乱

### 5.2 设置兼容

以下设置项必须继续生效：

- `playDetail.style.align`
- `playDetail.style.fontSize`
- `playDetail.isZoomActiveLrc`
- `playDetail.isShowLyricProgressSetting`
- `player.audioVisualization`
- 其他已影响播放详情页显示的现有设置项

### 5.3 性能兼容

应避免直接照搬高成本效果，特别是：

- 大面积强模糊
- 高频重绘动画
- 对逐字歌词播放造成额外布局抖动的复杂 transform
- 评论区展开时过重的阴影与滤镜叠加

要求：

- 动态背景可降级
- 激活歌词动效尽量使用轻量 transform / opacity
- 保持低配置设备下基本可用

### 5.4 平台兼容

本次页面位于主窗口渲染层，原则上平台风险低于桌面歌词窗口，但仍需确认：

- Windows
- macOS
- Linux

至少保证主窗口播放详情页不出现以下问题：

- 文本截断异常
- 高 DPI 下字号或间距严重失衡
- 全屏下头部 / 底部控件错位
- 背景图遮挡歌词可读性

## 6. UI 移植拆解

### 6.1 第一批移植项

- 页面整体结构改为更接近 `any-listen` 的三段式：
  - 顶部 Header
  - 中部 Main
  - 底部 Footer
- 左侧信息区重排：
  - 封面区域视觉强化
  - 歌曲信息字号层级优化
- 右侧歌词区重排：
  - 行间距
  - 左中右对齐下的边距调整
  - 激活歌词放大和强调
  - 更接近 `any-listen` 的 padding 和文字阴影
- 底部控制栏重排：
  - 进度条和时间信息层次优化
  - 左右控制区分离

### 6.2 第二批移植项

- 动态背景加强
- 封面 CD 风格可选样式
- 页面进入 / 退出动画细化
- 评论展开时的自适应布局优化

### 6.3 暂缓项

- 将 `any-listen` 的视觉语言移植到 `renderer-lyric` 独立桌面歌词窗口
- 新增设置项来完全复刻 `any-listen` 行为
- 大范围拆解 `lx` 现有歌词逻辑

## 7. 实施步骤

### 阶段 0：基线确认

- 记录当前播放详情页截图与行为
- 梳理受影响设置项
- 确认评论区、全屏、歌词选择、跳播等关键交互

产出：

- 基线功能清单
- 关键页面截图

### 阶段 1：结构重排

- 调整 `PlayDetail` 主模板结构
- 将页面划分为 Header / Main / Footer
- 规范 left / right / comment 区块职责

验收：

- 页面结构完成
- 不破坏原有显示逻辑

### 阶段 2：视觉迁移

- 重写封面区视觉
- 重写歌词区 spacing / 对齐 / 强调态
- 重写页脚排布
- 调整背景层与文本阴影

验收：

- 视觉上明显接近 `any-listen`
- 歌词区可读性优于现状

### 阶段 3：兼容性修正

- 检查逐字歌词模式
- 检查评论区展开
- 检查全屏
- 检查不同字体大小和歌词对齐方式
- 检查无封面 / 无歌词 / 长歌词 / 多扩展歌词情况

验收：

- 核心功能无回退
- 设置项有效

### 阶段 4：可选增强

- 评估是否引入 CD 封面风格切换
- 评估是否增加动态背景开关或自动降级策略
- 评估是否复用到桌面歌词工程

## 8. 风险清单

### 高风险

- `LyricPlayer.vue` 既有逻辑和 DOM 结构耦合较深，改模板时可能影响滚动、跳播、歌词激活状态
- 评论区展示逻辑依赖当前布局比例，重排后可能出现挤压或遮挡
- 全屏模式下 auto hide 与动画时序可能受影响

### 中风险

- 现有 CSS Module + 全局样式混用，重构时可能引入样式覆盖问题
- 逐字歌词在新 padding / transform 下可能出现对齐变化
- 背景图与文字阴影组合可能导致某些主题下可读性下降

### 低风险

- 页头页脚纯视觉微调
- 信息区字号层级调整

## 9. 验收标准

### 9.1 视觉验收

- 整体观感接近 `any-listen`
- 左右分区层次清晰
- 歌词区更聚焦、更有呼吸感
- 激活歌词突出但不过度跳动

### 9.2 功能验收

- 歌词同步正常
- 歌词跳播正常
- 歌词菜单正常
- 评论区正常
- 全屏正常
- 设置项正常

### 9.3 回归验收

至少验证以下场景：

- 有封面 + 有歌词
- 无封面 + 有歌词
- 有封面 + 无歌词
- 有翻译歌词 / 罗马音歌词
- 有逐字歌词
- 评论区展开
- 全屏模式
- 字号最小 / 最大
- 对齐方式为左 / 中 / 右

## 10. 建议实施策略

建议采用以下落地策略：

1. 先只改主窗口播放详情页，不动 `renderer-lyric`
2. 先保留 `LyricPlayer.vue` 的逻辑层，只重构模板和样式层
3. 分阶段提交，确保每一步都可回退
4. 每阶段完成后做一次人工回归

## 11. 第一版实施结论

建议立即执行的第一版内容：

- 重构 `PlayDetail` 页面结构
- 重做左侧封面信息区视觉
- 重做右侧歌词区视觉
- 重排底部控制栏
- 保持所有既有歌词功能和设置不变

不建议在第一版中执行的内容：

- 迁移独立桌面歌词窗口 UI
- 改歌词内核
- 扩充设置项

---

文档状态：初版  
创建日期：2026-03-23  
适用仓库：`lx-music-desktop`
