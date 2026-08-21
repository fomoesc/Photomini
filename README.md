# Photomini 图片压缩工具

> 极简图片压缩、格式转换、尺寸调整与水印添加桌面工具。

Photomini 面向需要快速处理图片的非专业用户：选择功能、设置参数、拖入图片，软件会自动批量处理并展示结果。

## 当前版本

- 版本：`1.2.2`
- macOS：支持 Apple Silicon / M 系列芯片构建
- Windows：支持 Windows 10+ 64 位构建

## 功能特性

- **图片压缩**：支持 JPG/JPEG、PNG，提供 45% 和 75% 两档常用质量。
- **WebP 转换**：支持 JPG/JPEG/PNG 转 WebP，也支持 WebP 转 JPG。
- **图片调整**：按最长边批量生成 400px、800px、1024px、1200px 等尺寸，可多选。
- **自定义水印**：在图片调整流程中选择水印图片，自定义九宫格位置、大小上限和边距。
- **水印增强**：输出时为水印添加轻微阴影，提升复杂背景下的可见性。
- **设置记忆**：水印开关、文件路径、位置、大小和边距会自动保存，下次打开继续沿用。
- **输出控制**：可保存至原文件夹的尺寸子目录，或另存到指定文件夹。
- **拖拽处理**：拖入或选择图片后自动处理，无需额外确认。
- **极简设计**：不主动生成日志文件或额外临时文件。

## 使用说明

1. 选择功能 TAB：图片压缩、Webp转换、图片调整。
2. 根据当前 TAB 设置质量、格式、目标尺寸和保存位置。
3. 如需水印，在图片调整 TAB 勾选“添加自定义水印”，点击“水印设置”。
4. 在弹窗中选择水印图片，设置位置、大小上限和边距。
5. 拖入图片或点击拖拽区域选择图片。
6. 等待处理完成，查看文件列表中的原大小、新大小和状态。
7. 点击“清空列表”回到设置状态。

## 输出规则

- 图片压缩：覆盖原文件，或保存到指定文件夹下的同名文件。
- WebP 转换：保存为 `.webp` 或 `.jpg`，可保存至原文件夹或指定文件夹。
- 图片调整：按目标尺寸生成 `400px/`、`800px/` 等子目录，文件名保持原名，扩展名按输出格式决定。
- 自定义水印：仅在图片调整 TAB 生效，会参与每个目标尺寸的输出。

## 技术栈

- 前端：React 18 + TypeScript + Vite
- 桌面框架：Tauri 2.x
- 后端：Rust
- 图片处理：image-rs
- WebP 编码：webp crate
- 文件选择：@tauri-apps/plugin-dialog

## 开发

```bash
npm install
npm run tauri dev
```

## 构建

```bash
npm run build
npm run tauri build
```

macOS Apple Silicon 本地构建示例：

```bash
./node_modules/.bin/tauri build --target aarch64-apple-darwin --bundles dmg --ci --no-sign
./node_modules/.bin/tauri build --target aarch64-apple-darwin --bundles app --ci --no-sign
```

## 发布流程

GitHub Actions 在以下场景触发：

- push `v*` 标签
- 手动运行 `workflow_dispatch`

发布新版本时建议：

```bash
git add README.md SPEC.md ENGINEERING.md package.json package-lock.json src-tauri/Cargo.toml src-tauri/Cargo.lock src-tauri/tauri.conf.json src/App.tsx src/styles.css src-tauri/src/lib.rs .github/workflows/build.yml
git commit -m "Release v1.2.1"
git tag v1.2.1
git push origin main
git push origin v1.2.1
```

Actions 会构建并上传：

- macOS arm64 DMG
- macOS arm64 `.app`
- macOS arm64 standalone binary
- Windows x64 MSI
- Windows x64 NSIS installer
- Windows x64 standalone `.exe`

## 文档

- [SPEC.md](SPEC.md)：产品规格和验收标准。
- [ENGINEERING.md](ENGINEERING.md)：工程结构、关键实现、发布流程和下次迭代入口。
