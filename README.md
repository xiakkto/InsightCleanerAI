# InsightCleanerAI

本项目是一个面向 Windows 的可视化磁盘分析与清理工具。模仿 SpaceSniffer 的 treemap 展示方式，结合本地/云端 AI 对每个目录或文件生成用途说明，让你在不熟悉软件名的情况下也能快速判断文件具体作用。

## 功能介绍

- **多种 AI 模式**：可以只用内置启发式规则、本地 LLM（Ollama、koboldcpp 等）、或者调用 OpenAI/DeepSeek/百度千帆等云端接口，还支持搜索 API 作为补充。
- **缓存策略**：AI 说明会缓存在 `insights.db`，支持“仅按路径匹配缓存”模式，避免文件大小变化导致重复调用。
- **黑/白名单管理**：扫描或 AI 上传都可以单独配置名单（黑名单/白名单二选一），敏感目录会在 UI 中标记 `[禁止]` 并跳过云端。
- **本地化 UI**：默认中文界面，支持离线/禁止徽标、双击打开资源管理器、右键删除等交互。
- **日志&调试**：`%AppData%\InsightCleanerAI\logs\debug.log` 记录扫描与 AI 调用过程，便于诊断。

## 环境需求

- Windows 10/11
- .NET 5.0 SDK（开发/构建）或 .NET 5 Desktop Runtime（运行）

## 构建与运行

```powershell
# 克隆仓库后
dotnet build InsightCleanerAI/InsightCleanerAI.csproj
dotnet run --project InsightCleanerAI/InsightCleanerAI.csproj
```

### 打包发布

```powershell
dotnet publish InsightCleanerAI/InsightCleanerAI.csproj `
  -c Release `
  -r win-x64 `
  --self-contained true `
  /p:PublishSingleFile=true `
  /p:IncludeNativeLibrariesForSelfExtract=true
```

发布后的可执行文件位于 `bin/Release/net5.0-windows/win-x64/publish/`。

## 使用说明

1. **选择根路径**：在主界面顶部指定要分析的目录，然后点击“扫描”。
2. **设置面板**：通过菜单 `设置 → 打开设置...` 配置隐私模式、最大节点数、AI 模式、缓存目录等。
3. **名单管理**：`设置 → 编辑名单...` 中可以分别配置“扫描名单”和“识别名单”，支持黑/白名单模式（每行一个绝对路径）。
4. **AI 模式**：
   - *关闭*：仅显示树状结构，不做 AI 说明。
   - *离线规则*：使用内置启发式分类器（最快，不联网）。
   - *本地 LLM 服务*：填入 HTTP Endpoint、模型名，可配合 Ollama、koboldcpp 等。
   - *云端搜索*：配置符合 OpenAI 风格的 Endpoint/Key；还可以选填百度搜索 API 作为搜索来源。
5. **API Key**：默认不写入 `settings.json`。在设置中勾选“保留 API Key”后才会持久化。

### 注意事项

- 扫描过程中可以点击“取消”。若要清空缓存，使用设置界面中的“清空缓存”按钮或删除 `%AppData%\InsightCleanerAI\insights.db`。
- 以管理员身份运行能访问更多系统目录，但也会触发大量 AI 请求，建议搭配黑/白名单限制扫描范围。

## 目录结构

```
InsightCleanerAI/
├── Models/               # 基础数据结构与枚举
├── ViewModels/           # WPF MVVM 逻辑
├── Services/             # 文件扫描、云端/本地 LLM 适配器、缓存等
├── Infrastructure/       # 配置存储、日志、转换器等
├── Resources/            # 本地化字符串、XAML 资源
└── bin/, obj/            # 构建产物
```

## 贡献

欢迎提交 Issue 或 PR，提交前请运行：

```powershell
dotnet build InsightCleanerAI/InsightCleanerAI.csproj -c Release
```

确保编译通过无错误。

## 版本历史

**v1.0.1（2025.11.20）**
- 修复本地 LLM 调用 100 秒超时的问题，新增可配置的超时时长（默认 300 秒），用取消令牌控制请求。
- 新增模型管理：支持云端 `/v1/models`/千帆 `/v2/models`、本地 Ollama `/api/tags` 拉取模型列表，设置页改为可编辑下拉框并提供“获取模型”按钮，扫描前校验模型可用性。
- 本地 LLM 解析与日志增强：兼容多种返回格式，解析失败时兜底展示截断的原始响应，补充请求/响应/异常日志。
- 设置页面新增帮助按钮并调整布局，提供各配置项的中文说明。
- 详情面板增加“删除文件/文件夹”按钮（需管理员权限），隐私模式或扫描中自动禁用。
- 完善调试日志，覆盖 AI 调用、模型加载、扫描流程，便于排查问题。


**v1.0.0** - 初始版本
- 基础文件扫描和 AI 分析功能
