# storytold/pdfcraft 0.5.0 简体中文界面方块乱码修复总结

> 对象：**storytold/pdfcraft**（Rust + egui 原生桌面版）官方 0.5.0 Windows 便携版
> 环境：Windows 11，8GB 内存，GTX 1660 SUPER

---

## 症状

官方 `pdfcraft-0.5.0-windows-x64-portable` 界面切简体中文后，满屏方块 □。

## 根因

**上游已知缺陷**（issue #112 / #140），非安装或配置问题：

- 简体中文**翻译词条完整**（约 1800 条），故大部分文字是中文
- 但 release 构建**未内嵌简体中文字体（Hans face）**，craft-fonts 只带日文字体
- 界面中文借用日文字形渲染：
  - 中日共用字形（迎、使用、合并、文件、推荐、工具、页面）→ 正常
  - 简体特有字形（欢、导、添、释、密、保、描）→ 方块 □
- 环境变量 `PDFCRAFT_SYSTEM_FONTS` 只影响文件名/错误信息里的系统字体回退，**不影响界面字体**，实测无效

## 修复方案：fork + GitHub Actions 云端构建内嵌简中字体

利用 GitHub Actions Windows runner（4核16GB）云端编译，避开本地 8GB 内存 OOM 与慢速单线程编译。

### 步骤 1：准备精简字体仓库（仅简中）
- craft-fonts 上游 main 已含 `Noto Sans CJK SC`（OFL-1.1 许可证，可自由内嵌分发）
- 官方 release.yml pin 的旧版（`abb8331`）**不含**简中，需升级到含简中的版本
- fork `storytold/craft-fonts` → `sky-afk/craft-fonts`，建分支 `zh-only`：
  - `fonts/manifest.txt` 裁剪为仅 1 行 Noto Sans CJK SC
  - 仅保留 `fonts/noto-sans-cjk-sc/NotoSansCJKsc-Regular.otf` + `OFL.txt`
  - orphan 提交推送（避免 blobless 克隆缺历史对象）

### 步骤 2：fork 主仓库并裁剪工作流
- fork `storytold/pdfcraft` → `sky-afk/pdfcraft`
- 将 `.github/workflows/release.yml` 替换为最小版：仅保留 `version` + `windows`（x64）两个 job
  - 去掉 macOS/linux/flatpak/freebsd/web/release 等无关 job（省时省额度）
  - Windows job 移除 `environment: release`（fork 无此环境，否则 dispatch 被拒）
  - craft-fonts checkout 改为 `sky-afk/craft-fonts` / `zh-only`
  - `CRAFT_FONTS_DIR` + `CRAFT_FONTS_REQUIRED=1` 保留（触发字体嵌入）
  - 产物只上传 `dist/release/*-portable.zip` 为独立 artifact（74MB，比整包 136MB 小 46%）

### 步骤 3：授权与触发
- `gh auth login` 设备码登录，**必须追加 `workflow` scope**：
  ```
  gh auth refresh -h github.com -s workflow
  ```
  （否则推送 `.github/workflows/*.yml` 被拒：refusing to allow OAuth App to update workflow without `workflow` scope）
- `gh workflow run release.yml -R sky-afk/pdfcraft --ref main` 触发
- `gh run watch <id> -R sky-afk/pdfcraft --exit-status` 监控
- 首次构建约 22 分钟（含依赖），改产物上传后重建约 5–10 分钟（rust-cache 命中）

## 结果

云端构建出内嵌思源黑体简中的 portable 版（74MB），简体中文界面方块乱码消除。

---

## 经验记录

1. **egui 原生应用字体乱码**：必须内嵌含目标字形的字体文件；OFL 字体可自由内嵌，随包附 OFL.txt 即可。微软雅黑等专有字体禁止内嵌分发。
2. **OFL 字体内嵌零授权障碍**：Noto Sans SC / 思源黑体 / 霞鹜文楷均为 SIL OFL 1.1，免费商用、可内嵌分发，仅需随发行包附许可证文本。

---

## 扩展：macOS 版同样乱码（2026-10-12）

### 分析

- PDFCraft 是 Rust + egui 原生应用，**界面字体与操作系统无关**：字体由 `crates/fonts/build.rs`
  在编译期从 `CRAFT_FONTS_DIR` 嵌入二进制，macOS/Windows 共用同一套 crates
- 官方 Mac 包与 Windows 包根因相同：未内嵌 Hans 字形 → 简中特有字方块
- `PDFCRAFT_SYSTEM_FONTS` 在 Mac 上同样不影响界面字体
- 本地 Windows 机器无法构建 Mac 包，唯一路线仍是 GitHub Actions macOS runner

### 改动

- `release.yml` 新增 `macos` job：`runs-on: macos-latest`，targets `aarch64 + x86_64`，
  `packaging/macos/package.sh --arch universal`，同样设置 `CRAFT_FONTS_DIR` + `CRAFT_FONTS_REQUIRED=1`
- 签名：fork 无 `APPLE_CERTIFICATE` 等 secrets，`import-cert.sh` 检测到缺失会 warn 并 exit 0，
  `package.sh` 自动降级为 **ad-hoc 签名、跳过公证**；产物可直接构建成功
- 发布步骤从 windows job 抽出为独立 `publish` job：`windows` + `macos` 产物经
  `actions/download-artifact`（merge-multiple）汇总后统一发布到 `v0.5.0-zh` Release
- 未签名 DMG 在 Mac 上首次打开需右键→打开，或 `xattr -dr com.apple.quarantine PdfCraft.app`

### 经验补充

3. **egui 跨平台字体缺陷一次定位全平台**：界面字体嵌入发生在编译期，与目标 OS 无关；
   修复一个平台的字体嵌入方案可直接平移到其他平台，只需对应平台的 CI runner。
4. **macOS 无证书构建可自动降级**：上游 packaging 脚本对缺失签名 secrets 有 fallback
   （ad-hoc 签名），fork 构建无需删除签名步骤，保留即可。

---

## 待办

- [x] 本文档已上传 GitHub（fork 仓库 `sky-afk/pdfcraft` 根目录）
- [x] 云端构建版简体中文界面已解压验证，无方块
- [ ] macOS universal DMG 云端构建并验证简中界面
