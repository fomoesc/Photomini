# Photomini 产品规格书

## 1. 项目概述

- 项目名称：Photomini（随手压）
- 当前版本：`1.2.2`
- 项目类型：桌面图片处理工具
- 核心功能：图片压缩、WebP 转换、尺寸调整、自定义水印
- 目标用户：需要快速处理图片、但不想学习专业图片软件的普通用户
- 支持平台：macOS Apple Silicon、本地 macOS 开发环境、Windows x64 GitHub Actions 构建

## 2. 窗口与布局

| 项目 | 规格 |
| --- | --- |
| 桌面框架 | Tauri 2.x |
| 默认窗口 | 520 x 880 |
| 最小窗口 | 400 x 360 |
| 拖拽文件 | 支持拖入单张或多张图片 |
| 主题 | 跟随系统浅色/深色模式 |

主界面由三部分组成：

1. TAB 栏：图片压缩 / Webp转换 / 图片调整。
2. 设置面板：仅在无文件处理列表时显示。
3. 拖拽区或文件列表：空闲时显示拖拽区，处理后显示结果列表。

水印设置采用“紧凑入口 + 弹窗”的交互：主设置面板只显示一行水印开关和入口，详细位置、边距、大小等参数在弹窗中设置，避免挤压拖拽区。

## 3. 功能规格

### 3.1 图片压缩

| 项目 | 规格 |
| --- | --- |
| 输入格式 | JPG/JPEG、PNG |
| 输出格式 | 与输入格式一致 |
| 质量选项 | 45%、75% |
| 保存方式 | 覆盖原文件 / 另存至指定文件夹 |

### 3.2 WebP 转换

| 项目 | 规格 |
| --- | --- |
| 转换模式 | JPG/PNG 转 WebP、WebP 转 JPG |
| 质量选项 | 45%、75% |
| 保存方式 | 保存至原文件夹 / 另存至指定文件夹 |

### 3.3 图片调整

| 项目 | 规格 |
| --- | --- |
| 输入格式 | JPG/JPEG、PNG、WebP |
| 目标尺寸 | 400px、800px、1024px、1200px，可多选 |
| 缩放规则 | 按最长边缩放，保持原图比例 |
| 小图处理 | 可选择图片较小时保持原样，不强制放大 |
| 输出格式 | 保持原格式 / 转为 WebP |
| 质量选项 | 45%、75% |
| 保存方式 | 原文件夹尺寸子目录 / 另存文件夹尺寸子目录 |

### 3.4 自定义水印

| 项目 | 规格 |
| --- | --- |
| 生效范围 | 图片调整 TAB |
| 水印格式 | JPG/JPEG、PNG、WebP |
| 位置 | 九宫格：左上、上中、右上、左中、居中、右中、左下、下中、右下 |
| 大小上限 | 输出图宽度的 8% 到 45%，默认 24% |
| 边距 | 输出图短边的 0% 到 12%，默认 3% |
| 自适应 | 水印大于上限时自动缩小，保持比例 |
| 透明度 | 保留 PNG/WebP 透明通道 |
| 阴影 | 自动添加轻微模糊阴影，增强复杂背景可读性 |
| 设置记忆 | 使用 localStorage 保存水印开关、文件路径、位置、大小、边距 |

## 4. 文件列表显示

| 列 | 内容 |
| --- | --- |
| 文件名 | 原始文件名 |
| 原大小 | KB/MB |
| 新大小 | KB/MB，多尺寸时显示所有输出文件大小合计 |
| 状态 | 等待 / 处理中 / 完成 / 失败 |

## 5. 输出路径规则

- 压缩保存到指定文件夹时：`目标文件夹/原文件名`。
- WebP 转换保存到原文件夹时：`原文件夹/原文件名.webp` 或 `.jpg`。
- 图片调整保存到原文件夹时：`原文件夹/尺寸px/原文件名.扩展名`。
- 图片调整另存时：`目标文件夹/尺寸px/原文件名.扩展名`。
- 转为 WebP 时扩展名固定为 `.webp`。
- 保持原格式时，PNG 输出保持 `.png`，其他常见位图输出为 `.jpg`，原 WebP 保持 `.webp`。

## 6. 技术架构

| 层级 | 技术 |
| --- | --- |
| 前端框架 | React 18 + TypeScript |
| 构建工具 | Vite |
| 桌面框架 | Tauri 2.x |
| 后端语言 | Rust |
| 图片处理 | image-rs |
| WebP 编码 | webp crate |
| 文件选择 | @tauri-apps/plugin-dialog |
| 文件权限 | @tauri-apps/plugin-fs |

## 7. Tauri 命令

```rust
fn get_file_info(path: String) -> FileInfo
fn compress_image(path: String, quality: u8, output_path: Option<String>) -> ProcessResult
fn convert_to_webp(path: String, quality: u8, output_path: Option<String>) -> ProcessResult
fn convert_to_jpg(path: String, quality: u8, output_path: Option<String>) -> ProcessResult
fn resize_image(
    path: String,
    target_size: u32,
    force_enlarge: bool,
    output_format: String,
    quality: u8,
    output_path: Option<String>,
    watermark_path: Option<String>,
    watermark_position: Option<String>,
    watermark_max_width_percent: Option<u8>,
    watermark_margin_percent: Option<u8>,
) -> ProcessResult
```

## 8. 发布规格

GitHub Actions 文件：`.github/workflows/build.yml`

触发方式：

- push `v*` 标签
- 手动 `workflow_dispatch`

构建产物：

- macOS Apple Silicon：DMG、`.app`、standalone binary
- Windows x64：MSI、NSIS `.exe` installer、standalone `.exe`

## 9. 验收标准

- 三个 TAB 均可正常切换。
- 拖入或选择图片后自动处理。
- 图片压缩和转换输出路径符合规则。
- 图片调整支持多尺寸输出。
- 自定义水印能正确合成到每个输出尺寸。
- 水印位置、大小、边距和阴影效果正确。
- 水印设置重启后仍能恢复。
- `npm run build` 通过。
- `cargo check` 通过。
- macOS arm64 可成功生成 DMG 和 `.app`。

---

文档版本：v1.2.1  
最后更新：2026-05-07
