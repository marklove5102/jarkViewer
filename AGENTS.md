# Repository Guidelines

## 项目结构与模块组织
`JarkViewer/` 是 Windows 桌面应用主体。实现代码放在 `JarkViewer/src/`，公共头文件放在 `JarkViewer/include/`，Win32 资源位于 `JarkViewer/file/`、`JarkViewer/JarkViewer.rc` 和 `JarkViewer/resource.h`。解决方案入口是 `JarkViewer.slnx`，主工程文件是 `JarkViewer/JarkViewer.vcxproj`。仓库根目录下的 `README.md` 与 `README_EN.md` 是主要的使用和开发说明。

## 构建、测试与开发命令
日常开发建议使用 Visual Studio 2026 打开 `devenv JarkViewer.slnx`。命令行构建使用：

```powershell
# 每次构建时需确保终端工作目录位于项目根目录，防止临时构建内容输出到其他位置。
msbuild JarkViewer\JarkViewer.vcxproj /p:Configuration=Debug /p:Platform=x64
msbuild JarkViewer\JarkViewer.vcxproj /p:Configuration=Release /p:Platform=x64
```

MSVC开发环境没有配置到系统环境变量，其中`msbuild`构建工具路径如下：
"C:\Program Files\Microsoft Visual Studio\18\Community\MSBuild\Current\Bin\amd64\MSBuild.exe"

`Debug` 用于本地调试，`Release` 对应发布构建。所有第三方静态库文件`*.lib`均已安装。

## 代码风格与命名规范
遵循 `JarkViewer/src` 现有 C++ 风格：4 空格缩进，大括号与语句同行，源码文件使用 UTF-8。使用`C++23`标准。`#include` 按逻辑分组，优先复用现有模式，不随意新增抽象层。类型名使用 `PascalCase`，函数和局部变量使用 `camelCase`，宏和既有常量使用 `ALL_CAPS`。

## 测试要求
当前仓库没有自动化测试套件。每次改动至少要保证 `Release|x64` 可干净编译。而 `Debug|x64` 保留给开发者自行调试。若涉及行为变更，请在程序内进行对应功能的手动冒烟验证，例如静态图加载、动图播放、EXIF 解析、打印预览和导出流程。

## 提交与 Pull Request 规范
近期提交信息偏向简短、祈使句风格，例如 `支持webm`、`移除废弃文件`、`修复异常EXIF解析失败`。首行应简洁并说明改动意图。Pull Request 需要概述用户可见影响，列出构建与手动验证步骤，关联相关 issue；若修改了 UI、渲染或打印效果，附上截图。

## 安全与配置提示
项目目标平台仅为 `Windows 10/11` 的 `x64` 环境。不要提交 `.vcxproj.user`、机器相关的本地库路径，或未说明用途的依赖压缩包；只有在确有必要且文档同步更新时才提交此类内容。
