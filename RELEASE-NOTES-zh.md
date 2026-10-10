# PdfCraft 0.5.0 简体中文界面修复版（Windows x64）

本 Release 由 fork 仓库 [sky-afk/pdfcraft](https://github.com/sky-afk/pdfcraft) 通过 GitHub Actions 云端构建，
**仅用于修复官方 0.5.0 Windows 版简体中文界面满屏方块 □ 的问题**（上游 issue #112 / #140）。

## 问题与修复

- 官方 release 构建未内嵌简体中文字体（Hans face），界面借用日文字形，简体特有字显示为方块。
- 本构建在打包时内嵌 **Noto Sans CJK SC（思源黑体简中，SIL OFL 1.1）**，简体中文界面正常显示。
- 功能与上游 0.5.0 完全一致，未做任何其他改动。

## 下载

| 文件 | 说明 |
|---|---|
| `pdfcraft-0.5.0-windows-x64-portable.zip` | 便携版，解压即用，数据存于程序目录 `PdfCraftData\` |
| `pdfcraft-0.5.0-windows-x64.msi` |  per-machine 安装版 |

## 许可与鸣谢

- 软件本体：上游 [storytold/pdfcraft](https://github.com/storytold/pdfcraft)，MIT OR Apache-2.0。
- 内嵌字体：Noto Sans CJK SC，SIL OFL 1.1，许可证文本见便携包内 `OFL-*.txt`。
- 感谢 ArtCraft 社区制作 PdfCraft。
