# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

JarkViewer 是一个轻量级 Windows (x64) 图片查看器，支持 70+ 种图片格式（含 AVIF、HEIC、JPEG XL、WebP2、RAW、PSD、SVG、BLP 等），支持动图播放和 iOS/Android Live Photo。使用 GPL-3.0 许可证。

## 构建

工具链：Visual Studio 2026, MSVC v145, C++23, 静态链接所有依赖。

```powershell
# MSBuild 路径（未加入系统 PATH）
"C:\Program Files\Microsoft Visual Studio\18\Community\MSBuild\Current\Bin\amd64\MSBuild.exe"

# 在项目根目录执行
msbuild JarkViewer\JarkViewer.vcxproj /p:Configuration=Release /p:Platform=x64
msbuild JarkViewer\JarkViewer.vcxproj /p:Configuration=Debug /p:Platform=x64
```

- 所有第三方 `*.lib` 已预编译并存放在 `JarkViewer/lib/`、`JarkViewer/ffmpeg/`、`JarkViewer/libavif/` 等子目录，vcpkg 已禁用
- Release 使用 `/O2` + LTCG + OpenMP；Debug/Release 均使用静态运行时 `/MT`
- 输出为单个独立 `.exe`，无 DLL 依赖

## 测试

无自动化测试套件。每次改动必须保证 `Release|x64` 干净编译。行为变更需手动冒烟验证：静态图加载、动图播放、EXIF 解析、打印预览、导出流程。

## 架构

### 渲染双轨制
- **主窗口**：Direct2D 1.1 + D3D11 + DXGI 交换链，硬件加速渲染（`D2D1App` 基类）
- **辅助对话框**（设置、打印）：OpenCV HighGUI (`cv::imshow` + 鼠标回调)

### 核心源码 (`JarkViewer/src/`)

| 文件 | 职责 |
|------|------|
| `main.cpp` | WinMain 入口，主应用类继承 `D2D1App`，包含所有交互事件、渲染循环、幻灯片逻辑 |
| `ImageDatabase.cpp` | 图片加载引擎，扩展 `LRU` 缓存模板，按格式分发到各解码器（OpenCV/libheif/libavif/libjxl/libraw/lunasvg/PSD SDK/FFmpeg 等） |
| `D2D1App.cpp` | Direct2D 应用框架：窗口创建、D3D11/D2D1 设备初始化、交换链管理、消息循环、设置二进制读写 |
| `jarkUtils.cpp` | 工具函数：字符串转换(UTF-8/wstring)、文件I/O、剪贴板、窗口操作 |
| `exifParse.cpp` | EXIF/XMP/IPTC 元数据提取（exiv2），含 AI 生图 prompt 解析 |
| `videoDecoder.cpp` | FFmpeg 视频帧解码 |
| `TextDrawer.cpp` | stb_truetype 字体渲染 |
| `blpDecoder.cpp` | Blizzard BLP 纹理格式解码 |
| `stringRes.cpp` | 双语(中/英)字符串资源表 |
| `tinyxml2.cpp` | vendored XML 解析器 |

### 关键头文件 (`JarkViewer/include/`)

| 头文件 | 职责 |
|--------|------|
| `jarkUtils.h` | 中枢头文件：包含 STL、Windows API、DirectX、OpenCV，定义 `SettingParameter`(4096字节二进制设置)、`GlobalVar`(全局状态)、`ThemeColor` 等核心类型 |
| `ImageDatabase.h` | `ImageDatabase` 类，定义支持的格式集合，通过 `#pragma comment(lib)` 链接所有静态库 |
| `LRU.h` | 泛型 LRU 缓存模板，带后台预加载线程，使用 `shared_mutex` 并发读 |
| `Printer.h` | 打印/编辑对话框 |
| `Setting.h` | 设置对话框（4 个标签页） |

### 图片解码流程
`ImageDatabase::loadImage()` 根据文件扩展名分发 -> 格式特定解码器 -> 返回 `ImageAsset`（含帧序列和元数据） -> 存入 LRU 缓存 -> `D2D1App` 渲染到屏幕

## 代码风格

- 4 空格缩进，大括号与语句同行
- 源码 UTF-8，C++23 标准
- 类型名 `PascalCase`，函数和局部变量 `camelCase`，宏和常量 `ALL_CAPS`
- `#include` 按逻辑分组，优先复用现有模式
- 提交信息：简短中文祈使句（如 `支持webm`、`修复异常EXIF解析失败`）

## 注意事项

- 目标平台仅 Windows 10/11 x64
- 不要提交 `.vcxproj.user` 或机器相关的本地库路径
- 设置持久化是固定 4096 字节的 `SettingParameter` 结构体直接二进制读写，修改字段时注意保持向后兼容
- 国际化通过 `stringRes.cpp` 中的字符串 ID 表实现，新增 UI 文本需同时添加中英文
