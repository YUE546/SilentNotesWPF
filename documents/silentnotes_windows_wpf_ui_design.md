# SilentNotes Windows WPF UI 设计文档

## 设计目标

Windows WPF 客户端应呈现为一个安静、可靠、低干扰的本地笔记工具。界面让用户快速完成三件事：找到笔记、编辑内容、确认保存状态。

## 当前 UI 状态

### 已落地

| 内容 | 状态 |
|---|---|
| 浅色顶部命令栏 | 已完成 |
| 左侧浅灰导航区 | 已完成 |
| 主编辑区白色纸面 | 已完成 |
| 底部状态栏 | 已完成（含错误红色反馈） |
| 活动笔记/回收站切换 | 已完成 |
| 搜索框 | 已完成 |
| 标签过滤 | 已完成（横向滚动列表 + Material Design tag-outline 图标） |
| 笔记列表 | 已完成两行版本（标题+标签·时间·置顶） |
| 空列表/回收站状态 | 已完成 |
| 标签编辑、置顶切换 | 已完成 |
| 基础格式工具栏 | 已完成（含 H1/H2/H3 标题按钮） |
| 回收站管理按钮 | 已完成（垃圾桶图标 + 红色删除） |
| 快捷键 | 已完成（Ctrl+N/S/F/B/I/U/Z/Y、Delete） |
| 编辑器最大阅读宽度 | 已完成（800px） |
| 编辑器撤销/重做 | 已完成（Ctrl+Z/Ctrl+Y 按钮） |
| 主题资源文件 | 已完成 |
| MainWindow 拆分 | 已完成（SidebarView、NoteEditorView、EditorToolbar、StatusStrip） |
| 应用图标 | 已完成（使用原版 SilentNotesLogoExport.ico） |
| 标题颜色 | 已完成（原版 SteelBlue + 透明度递减） |
| 引用块样式 | 已完成（左边灰色竖线 + 浅灰背景） |

### 主题资源文件

```text
src/SilentNotes.WindowsWpf/Themes/Colors.xaml
src/SilentNotes.WindowsWpf/Themes/Typography.xaml
src/SilentNotes.WindowsWpf/Themes/Controls.xaml
src/SilentNotes.WindowsWpf/Themes/ListStyles.xaml
```

## 视觉主张

界面避免：大面积高饱和渐变、多层卡片堆叠、厚边框、多主按钮同时出现。

界面优先使用：两栏工作区、浅灰导航区和白色编辑区、单一强调色、明确选中状态、状态栏反馈。

## 信息架构

| 区域 | 作用 | 当前实现 |
|---|---|---|
| 顶部命令栏 | 全局操作和仓库状态 | 应用名、本地仓库状态、重新加载、保存 |
| 左侧导航栏 | 找到笔记 | 活动笔记/回收站、搜索、标签、笔记列表、模式操作 |
| 主编辑区 | 编辑当前笔记 | 标题、标签、置顶、格式工具栏、正文编辑器 |
| 底部状态栏 | 非阻塞反馈 | 加载、保存、删除、恢复等结果 |

## 色彩规范

| Token | 值 | 用途 |
|---|---|---|
| SurfaceWindow | `#F6F7F9` | 窗口背景 |
| SurfacePanel | `#EEF1F5` | 左侧导航背景 |
| SurfacePaper | `#FFFFFF` | 编辑区背景 |
| BorderSubtle | `#D9DEE7` | 分隔线 |
| TextPrimary | `#1F2933` | 主文本 |
| TextSecondary | `#667085` | 说明文字 |
| Accent | `#4B6BFB` | 主按钮、焦点、选中 |
| AccentSoft | `#E9EDFF` | 选中列表项背景 |
| Danger | `#B42318` | 危险操作 |
| DangerSoft | `#FEE4E2` | 危险操作背景 |

## 字体与文本

- UI 字体：`Microsoft YaHei UI, Segoe UI`
- 等宽字体：`Consolas`
- 正文编辑字号：16pt，行间距 22

## 快捷键

| 快捷键 | 行为 |
|---|---|
| Ctrl+N | 新建笔记 |
| Ctrl+S | 保存当前笔记 |
| Ctrl+F | 聚焦搜索框 |
| Delete | 活动笔记移到回收站 |
| Ctrl+B/I/U | 粗体/斜体/下划线 |
| Ctrl+Z/Y | 撤销/重做 |

## 动效策略

允许：列表项选中背景变化、状态栏消息淡入。

禁止：页面大幅滑入滑出、按钮弹跳、编辑器内容动画。

## 验收标准

- 可用 VS2017 BuildTools 编译 .NET Framework 4.7.2
- 不依赖 WebView2
- 左侧找笔记，右侧写笔记
- 危险操作只在回收站模式出现
- 保存、加载、只读原因不打断用户
- 高 DPI 下不明显拥挤或截断
- 新增页面使用主题资源

## 下一步 UI 开发建议

### 待实现

- 安全箱解锁 UI
- 清单编辑 UI
- 同步向导 UI
- 设置页 UI
- 笔记背景颜色选择
