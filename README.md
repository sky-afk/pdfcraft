> [!IMPORTANT]
> **这是 [storytold/pdfcraft](https://github.com/storytold/pdfcraft) 的 fork，不是新项目。**
> 本仓库唯一目的：通过 GitHub Actions 云端构建，发布**内嵌简体中文（Hans）字体**的 Windows x64 与 macOS 版本，
> 修复官方 0.5.0 简体中文界面满屏方块 □ 的问题（上游 issue #112 / #140）。
> 软件功能、代码与上游完全一致；项目介绍、官网、Discord 等请见
> [上游 README](https://github.com/storytold/pdfcraft#readme)。

# PdfCraft 简体中文修复版（Windows x64 / macOS universal）

## 问题

官方版本切换简体中文后界面出现大量方块 □：

- 简体中文翻译词条是完整的，但 release 构建**未内嵌简中字体**（craft-fonts 只有日文字体）
- 界面中文借用日文字形渲染，简体特有字形（欢、导、添、释、密、保…）显示为方块
- 与安装方式、系统字体设置无关，环境变量 `PDFCRAFT_SYSTEM_FONTS` 不影响界面字体
- 该缺陷与操作系统无关：macOS 版同样因缺少简中字形而显示方块

## 修复方式

- fork 字体仓库 [sky-afk/craft-fonts](https://github.com/sky-afk/craft-fonts) 的 `zh-only` 分支：
  仅保留 **Noto Sans CJK SC**（思源黑体简中，SIL OFL 1.1，可自由内嵌分发）
- 改造 `.github/workflows/release.yml`：构建 Windows x64 与 macOS universal，打包时设置
  `CRAFT_FONTS_DIR` + `CRAFT_FONTS_REQUIRED=1` 触发字体嵌入，构建完成后自动发布本 Release
- 详细过程见 [BUGFIX-SUMMARY.md](BUGFIX-SUMMARY.md)

## 下载

前往 [Releases](../../releases) 页面：

| 文件 | 说明 |
|---|---|
| `pdfcraft-0.5.0-windows-x64-portable.zip` | Windows 便携版（约 74MB），解压即用，数据存于程序目录 `PdfCraftData\` |
| `pdfcraft-0.5.0-windows-x64.msi` | Windows 安装版（约 63MB） |
| `pdfcraft-0.5.0-macos-universal.dmg` | macOS 通用版（Apple Silicon + Intel） |

Windows 安装/解压后将界面语言切换为简体中文即可，无需任何额外配置。

> macOS 包为 **ad-hoc 签名、未公证**（fork 无 Apple 开发者证书）。首次打开需右键 → 打开，
> 或在终端执行 `xattr -dr com.apple.quarantine /Applications/PdfCraft.app` 后正常运行。

> [!NOTE]
> 由于本人没有 Mac 电脑环境，macOS 版本**未经实测安装验证**，仅通过与 Windows 版相同的字体嵌入
> 方案构建（egui 界面字体编译期内嵌，与操作系统无关）。如在 Mac 上遇到安装或显示问题，
> 欢迎在 [Issues](../../issues) 反馈。

## 许可与鸣谢

- 软件本体版权归上游 [storytold/pdfcraft](https://github.com/storytold/pdfcraft)，MIT OR Apache-2.0
- 内嵌字体 Noto Sans CJK SC：SIL OFL 1.1，许可证文本随便携包分发（`OFL-*.txt`）
- 感谢 ArtCraft 社区制作 PdfCraft；若上游后续版本正式修复简中字体，请以官方发布为准

