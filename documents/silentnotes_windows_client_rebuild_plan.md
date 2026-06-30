# SilentNotes Windows 客户端重做需求文档

## 当前目标

Windows 客户端改为独立的 WPF 原生桌面程序，不依赖 WebView2，不要求用户安装 .NET 8+ Desktop Runtime。

技术约束：

- 目标框架：`.NET Framework 4.7.2`
- 编译工具：`C:\Program Files (x86)\Microsoft Visual Studio\2017\BuildTools`
- UI 技术：WPF/XAML
- 输出目录：`src/SilentNotes.WindowsWpf/build-output/Debug`
- 应用图标：`design/SilentNotesLogoExport.ico`

## 当前进度

### 已完成

| 模块 | 状态 |
|---|---|
| WPF 项目框架 | 已创建，可启动窗口 |
| 编译链 | VS2017 BuildTools/MSBuild 15，Debug 配置可编译 |
| 核心类库 | SilentNotes.Core + VanillaCloudStorageClient |
| 平台服务 | DPAPI、随机数、仓库、设置、语言资源、文件选择、浏览器、反馈 |
| 本地仓库 | 加载、创建、保存 |
| 笔记列表 | 活动笔记、搜索、标签过滤、两行 ItemTemplate |
| 普通笔记编辑 | 基础富文本编辑和保存 |
| 回收站 | 查看、恢复、永久删除、清空 |
| 标签与置顶 | 编辑标签、切换置顶 |
| 标题格式 | H1/H2/H3（SteelBlue 配色 + 透明度递减） |
| 引用块 | 左边灰色竖线 + 浅灰背景 |
| 清单笔记 | HTML ↔ FlowDocument 转换 |
| 富文本增强 | 删除线、代码块、水平分割线、超链接 |
| 格式切换 | 所有格式按钮支持开关切换 |
| 安全箱 | 创建、密码打开/关闭、笔记自动加解密 |
| 侧边栏 UI | 标签过滤横向滚动 + Material Design tag-outline 图标 |
| 编辑器撤销/重做 | Ctrl+Z/Y 按钮 |
| 快捷键 | Ctrl+N/S/F/B/I/U/Z/Y、Delete |
| 状态栏 | 错误红色文字反馈 |
| 编辑器宽度 | 最大阅读宽度 800px |
| 应用图标 | 使用原版 SilentNotesLogoExport.ico |
| 日志系统 | ILogService + WindowsLogService，从注册表读取数据目录 |
| 禁止多开 | Mutex + SetForegroundWindow |
| WebDAV 同步 | ExistsFile、上传、下载、智能合并、Fingerprint 比较 |
| 数据目录注册表存储 | HKCU\Software\SilentNotes\DataDirectory |
| 同步备份限制 | 最多保留5份，超出自动删除 |
| 数据目录迁移 | 更改目录时自动复制文件并清理旧目录 |
| Git 跟踪范围 | 仅跟踪 SilentNotes/src/SilentNotes.WindowsWpf/ |

### 当前可运行产物

```text
src/SilentNotes.WindowsWpf/build-output/Debug/SilentNotes.WindowsWpf.exe
```

### 编译命令

```powershell
& 'C:\Program Files (x86)\Microsoft Visual Studio\2017\BuildTools\MSBuild\15.0\Bin\MSBuild.exe' 'H:\silentnotes-trae\SilentNotes\src\SilentNotes.WindowsWpf\SilentNotes.WindowsWpf.csproj' /t:Build /p:Configuration=Debug /verbosity:minimal
```

## 架构决策

### 数据目录注册表存储

```
HKCU\Software\SilentNotes
└── DataDirectory (REG_SZ) — 数据目录完整路径
```

- `WindowsSettingsService.GetDirectoryPath()` 优先从注册表读取
- `WindowsSettingsService.TrySaveSettingsToLocalDevice()` 在数据目录变化时更新注册表
- `WindowsLogService` 同样从注册表读取日志目录
- 默认值为 `AppDomain.CurrentDomain.BaseDirectory`

### 同步备份机制

- 同步前自动创建备份到 `sync_backups/` 目录
- 文件名格式：`presync_{yyyyMMdd_HHmmss}.silentnotes`
- 最多保留5份，超出自动删除最旧备份

### 数据目录迁移

更改数据目录时：
1. 复制仓库文件（.silentnotes + .backup + .new/.old）
2. 复制日志文件
3. 复制 sync_backups 目录
4. 自动删除旧目录中所有内容

## 功能现状

### 主窗口

- 启动后加载本地仓库
- 仓库不存在时创建新仓库
- 左侧显示活动笔记或回收站笔记
- 搜索按正文和标签过滤
- 标签列表按当前视图生成
- 选择笔记前自动保存当前普通笔记
- 关闭窗口前保存当前普通笔记
- 禁止多开（Mutex + SetForegroundWindow）

### 编辑器支持的格式

- 段落、标题（H1/H2/H3）
- 粗体、斜体、下划线、删除线
- 无序列表、有序列表
- 引用块、代码块
- 水平分割线
- 超链接

### 安全箱

- 安全箱密钥用密码经 `Cryptor("SilentSafe")` 加密存储
- 笔记内容用安全箱密钥经 `Cryptor("SilentNotes")` 加密
- 打开/关闭安全箱时自动解密/加密当前笔记
- 标题自动解密

### WebDAV 同步

同步流程：
1. 检查云端是否存在文件
2. 不存在 → 自动生成传输码（首次）→ 上传
3. 存在 → 下载 → 解密 → 检查仓库 ID
4. ID 相同 → 自动合并
5. ID 不同 → 显示选择对话框
6. 合并后仅在有变化时保存/上传

## 需要继续完成的功能

### 第二优先级

- 同步扩展：30 分钟定时自动同步、OAuth 流程
- 修改安全箱密码
- JEX 导入导出
- 更完整的 HTML round-trip
- 笔记背景颜色（BackgroundColorHex）

### 第三优先级

- 打包（ZIP 和安装包）
- 测试项目与 CI

## 与原版已知差异

| 功能 | 原版 SilentNotes | 当前 WPF | 状态 |
|------|-----------------|----------|------|
| 笔记背景颜色 | 支持 BackgroundColorHex | 不支持 | 待实现 |
| 安全箱修改密码 | 支持 | 不支持 | 待实现 |
| JEX 导入导出 | 支持 | 不支持 | 待实现 |
| OAuth2 云存储 | 支持 Google Drive 等 | 仅 WebDAV | 待实现 |
| 定时自动同步 | 30 分钟定时器 | 仅启动时同步 | 待实现 |

## 当前风险

| 风险 | 当前处理 |
|---|---|
| HTML 转换覆盖面不足 | 当前只支持基础子集；复杂 HTML 保守只读 |
| 同步仅支持 WebDAV | 基础同步已完成，OAuth 云服务和定时自动同步待后续扩展 |
| 自动同步定时器未启用 | 仅启动时同步一次 |

## 文档关系

本文件定义产品目标、技术边界、功能范围、进度和验证方式。UI 的视觉体系、布局规范、控件风格以 `silentnotes_windows_wpf_ui_design.md` 为准。
