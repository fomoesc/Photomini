# Photomini 工程说明

这份文档用于下次迭代时快速理解工程，不需要先从零读完整代码。

## 1. 当前状态

- 当前版本：`1.2.2`
- 主分支：`main`
- 远端仓库：`https://github.com/fomoesc/Photomini.git`
- 产品形态：Tauri 2 桌面应用
- 核心能力：图片压缩、WebP 转换、图片调整、自定义水印
- 当前发布流：push `v*` tag 触发 GitHub Actions 构建 macOS arm64 和 Windows x64 产物

## 2. 目录结构

```text
Photomini/
├── README.md                    # 用户向说明
├── SPEC.md                      # 产品规格和验收标准
├── ENGINEERING.md               # 工程接力说明
├── package.json                 # 前端依赖和 npm scripts
├── vite.config.ts               # Vite 配置
├── src/
│   ├── App.tsx                  # 主 UI、状态、Tauri 命令调用
│   ├── main.tsx                 # React 入口
│   └── styles.css               # 全局样式
├── src-tauri/
│   ├── tauri.conf.json          # Tauri 应用、窗口、bundle 配置
│   ├── Cargo.toml               # Rust 依赖和包信息
│   ├── capabilities/default.json # Tauri 权限
│   └── src/
│       ├── main.rs              # Tauri 启动入口
│       └── lib.rs               # 所有 Tauri commands 和图片处理逻辑
└── .github/workflows/build.yml  # macOS/Windows 构建工作流
```

## 3. 前端主流程

主文件：[src/App.tsx](src/App.tsx)

重要状态：

- `activeTab`：当前 TAB，值为 `compress`、`convert`、`resize`。
- `files`：处理队列和结果列表。
- `compressQuality`、`compressOutputMode`、`compressSavePath`：压缩参数。
- `convertMode`、`convertQuality`、`convertOutputMode`、`convertSavePath`：转换参数。
- `resizeSelectedSizes`、`resizeKeepSmall`、`resizeFormat`、`resizeQuality`、`resizeSaveMode`、`resizeSavePath`：图片调整参数。
- `resizeWatermarkEnabled`、`resizeWatermarkPath`、`resizeWatermarkPosition`、`resizeWatermarkMaxWidthPercent`、`resizeWatermarkMarginPercent`：水印参数。
- `isWatermarkSettingsOpen`：水印弹窗开关。

处理逻辑：

1. 用户拖拽图片或点击选择文件。
2. 前端调用 `get_file_info` 生成 `FileItem` 队列。
3. `useEffect` 监听 `files`，逐个取 `pending` 文件处理。
4. 根据 `activeTab` 调用对应 Tauri command。
5. 图片调整 TAB 会对每个选中的目标尺寸循环调用 `resize_image`。
6. 返回结果后更新文件列表状态和输出大小。

水印设置持久化：

- key：`photomini.resizeWatermarkSettings.v1`
- 存储位置：浏览器/Tauri WebView 的 `localStorage`
- 保存内容：开关、文件路径、位置、大小上限、边距
- 读取时会校验位置枚举，并对数字范围做 clamp

## 4. Rust 图片处理链路

主文件：[src-tauri/src/lib.rs](src-tauri/src/lib.rs)

Tauri commands：

- `get_file_info`
- `compress_image`
- `convert_to_webp`
- `convert_to_jpg`
- `resize_image`

核心结构：

- `ProcessResult`：统一返回原大小、新大小、输出路径、成功状态和错误。
- `FileInfo`：返回路径、文件名和文件大小。

图片调整流程：

1. `image::open` 读取输入图。
2. 根据最长边和 `force_enlarge` 计算目标尺寸。
3. 使用 `Lanczos3` 缩放。
4. 转为 `RgbaImage`，方便保留透明水印和合成阴影。
5. 如存在 `watermark_path`，调用 `apply_watermark`。
6. 根据 `output_format` 和原始格式决定实际输出格式。
7. 写出 WebP、PNG 或 JPG。

水印合成：

- `apply_watermark` 负责读取水印、限制最大宽度、限制可用区域、计算九宫格坐标。
- `create_watermark_shadow` 根据水印 alpha 生成黑色半透明阴影层。
- 阴影会轻微模糊并按输出图尺寸偏移 1 到 4 像素。
- 最后先 overlay 阴影，再 overlay 原水印。

## 5. 样式与交互

主样式：[src/styles.css](src/styles.css)

当前设计倾向：

- 主界面保持单窗口工具感，不做营销页。
- 设置项保持紧凑，避免挤占拖拽区。
- 水印详细参数放进 modal，不直接撑开设置面板。
- 颜色变量集中在 `:root`，支持深色模式。

后续新增设置时，优先遵循：

- 常用设置直接放面板。
- 低频或高级设置放弹窗。
- 不让拖拽区在默认窗口高度下消失。

## 6. 构建与验证

开发：

```bash
npm install
npm run tauri dev
```

常规验证：

```bash
npm run build
cd src-tauri
cargo check
```

macOS Apple Silicon 本地打包：

```bash
./node_modules/.bin/tauri build --target aarch64-apple-darwin --bundles dmg --ci --no-sign
./node_modules/.bin/tauri build --target aarch64-apple-darwin --bundles app --ci --no-sign
```

产物路径：

```text
src-tauri/target/aarch64-apple-darwin/release/photomini
src-tauri/target/aarch64-apple-darwin/release/bundle/macos/Photomini.app
src-tauri/target/aarch64-apple-darwin/release/bundle/dmg/Photomini_<version>_aarch64.dmg
```

注意：

- macOS DMG 打包需要 `hdiutil`，在受限沙箱里可能失败，需要在正常系统权限下运行。
- 当前 bundle identifier 是 `com.photomini.app`，Tauri 会提示 `.app` 后缀不推荐，但当前构建可用。

## 7. 版本发布流程

发布前同步版本号：

- `package.json`
- `package-lock.json`
- `src-tauri/tauri.conf.json`
- `src-tauri/Cargo.toml`
- `src-tauri/Cargo.lock`
- `src/App.tsx` 底部版本号
- `README.md`
- `SPEC.md`
- `ENGINEERING.md`

提交并触发 GitHub Actions：

```bash
git add README.md SPEC.md ENGINEERING.md package.json package-lock.json src-tauri/Cargo.toml src-tauri/Cargo.lock src-tauri/tauri.conf.json src/App.tsx src/styles.css src-tauri/src/lib.rs .github/workflows/build.yml
git commit -m "Release v1.2.1"
git tag v1.2.1
git push origin main
git push origin v1.2.1
```

触发后到 GitHub Actions 页面查看 `Build Release` workflow。

## 8. 已知注意事项

- `项目复盘.md` 目前在工作区有用户侧未提交改动，除非明确要求，不要覆盖或回滚。
- 前端没有拆组件，`App.tsx` 偏大；若后续功能继续增加，建议先拆出 `ResizeSettings`、`WatermarkModal`、`FileList`。
- 当前质量选项是按钮档位，不是连续滑块；如果改回滑块，需要同步 README/SPEC。
- Windows 产物依赖 GitHub Actions 或 Windows 本机环境，macOS 本机不直接产 `.exe/.msi`。

## 9. 下次迭代优先阅读顺序

1. `README.md`
2. `SPEC.md`
3. 本文件 `ENGINEERING.md`
4. `src/App.tsx`
5. `src-tauri/src/lib.rs`
6. `.github/workflows/build.yml`
