# PCL2 下载解锁版（自编译源码）

基于 [Plain Craft Launcher 2](https://github.com/Meloong-Git/PCL) **2.99.0.0 正式版**（上游 commit `6b9a50df911c9d68b69b24c979efba477fb14b67`）的开源版源码，仅解锁因合规原因关闭的 Minecraft 下载功能。

> 本项目**不是** PCL 官方版本，与作者 LTCat / Meloong 无关，请勿向官方反馈问题。

## 改了什么

**只有一处改动**，位于 `Plain Craft Launcher 2/Modules/ModSecret.vb`：

```diff
-    Public Const RestrictMcDownload As Boolean = True
+    Public Const RestrictMcDownload As Boolean = False
```

这个常量在全项目被 7 处引用，全部是「是否阻止下载」的开关：

| 文件 | 位置 | 作用 |
| --- | --- | --- |
| `Modules/ModSecret.vb` | 定义 | 开关本体 |
| `Modules/Minecraft/ModModpack.vb` | 11 行 | 整合包安装入口，为 `True` 时弹提示后直接返回 |
| `Modules/Minecraft/ModModpack.vb` | 88 行 | 为 `True` 时抛异常「为遵守相关法律法规，PCL 暂时不再提供 MC 下载功能」 |
| `Pages/PageDownload/Resource/PageDownloadCompDetail.xaml.vb` | 275 行 | 整合包「安装」按钮的显示条件 |
| `Pages/PageDownload/PageDownloadInstall.xaml.vb` | 17 行 | 下载安装页的入口判断 |
| `Pages/PageDownload/PageDownloadModify.vb` | 315 行 | 版本设置页「重装 / 安装」卡片的显隐 |
| `FormMain.xaml.vb` | 1079 行 | 主窗体对该异常的特殊处理 |

改成 `False` 后，以上 7 处分支全部不再阻止，下载页会正常显示 Minecraft 原版、Forge、Fabric、NeoForge、Quilt、OptiFine、整合包等下载入口。

**其余源码保持上游原样**：没有改主题、没有改 UI、没有新增任何功能。

## 与上游的一处结构差异

上游用 git submodule 引入 `MeloongCore`。本仓库把它**内联为普通目录**（已移除 `.gitmodules`），因此 `git clone` 下来即可直接编译，不需要再执行 `git submodule update --init`。

## 编译

### 方式 A：Visual Studio 2022（推荐）

1. 安装工作负载 **.NET 桌面开发**，并在单个组件里勾选 **.NET Framework 4.8 目标包 / 开发工具**
2. 打开 `Plain Craft Launcher 2.sln`
3. 配置切到 `Release`，生成解决方案

### 方式 B：命令行（.NET SDK 8 及以上）

```powershell
dotnet build "Plain Craft Launcher 2\Plain Craft Launcher 2.vbproj" -c Release -m:1
```

仓库根目录的 `Directory.Build.targets` 是为此准备的，它做三件事：

- 把 C# 语言版本降到 12（SDK 8 自带的 Roslyn 只支持到 C# 12）
- 显式导入 WPF 标记编译目标（旧式 `vbproj` 不会自动导入，否则 `*.g.vb` 不生成，会报几百个 `BC30506`）
- 补齐 `mscorlib`、`Microsoft.VisualBasic` 等框架引用（.NET Core 版 MSBuild 不会自动补）

这些设置都带 `MSBuildRuntimeType == 'Core'` 条件，**在 Visual Studio 里完全不生效**，两种构建方式可以共存。

命令行方式还需要 .NET Framework 4.8 的引用程序集。若报 `MSB3644`，装 VS 里的 4.8 开发工具，或加 `-p:TargetFrameworkRootPath=<引用程序集目录>`。

产物位于 `Plain Craft Launcher 2\bin\Plain Craft Launcher 2.exe`，单文件已内嵌全部依赖。

## 说明与免责

- PCL 官方关闭该功能是出于合规考虑。本仓库只是把源码里的一个布尔常量改回 `False`，**使用、分发、编译产物的任何后果由使用者自行承担**。
- 下载能否成功取决于你的网络能否访问 Mojang / BMCLAPI 等上游服务，与本次改动无关。
- 许可证沿用上游 **GPL-3.0**，见 `LICENCE`。
- 上游项目：<https://github.com/Meloong-Git/PCL>
