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

云端构建出内嵌思源黑体简中的 portable 版（74MB），简体中文界面方块乱码消除（待最终界面验证确认）。

---

## 经验记录

1. **egui 原生应用字体乱码**：必须内嵌含目标字形的字体文件；OFL 字体可自由内嵌，随包附 OFL.txt 即可。微软雅黑等专有字体禁止内嵌分发。
2. **OFL 字体内嵌零授权障碍**：Noto Sans SC / 思源黑体 / 霞鹜文楷均为 SIL OFL 1.1，免费商用、可内嵌分发，仅需随发行包附许可证文本。
3. **Windows Rust 编译 OOM**：8GB 内存机器上 cargo release 编译多进程并发会触发 `rustc-LLVM ERROR: out of memory`，用 `CARGO_BUILD_JOBS=1` 单线程可解（代价是慢，故优先云端 Actions）。
4. **C 盘空间保护**：`.rustup`/`.cargo` 用 junction 迁到 E 盘；npm 缓存、TEMP、打包工具缓存均可指到 E 盘。

---

## 待办

- [ ] 网络好转后上传本文档到 GitHub（fork + PR 或直接 push）
- [ ] 最终验证云端构建版简体中文界面（解压后启动截图确认无方块）
