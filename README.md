# lunalauncher-aur

`lunalauncher-git` 的 AUR 打包仓库，独立于主仓库
[AndreaFrederica/LunaLauncher](https://github.com/AndreaFrederica/LunaLauncher) 维护。

## 模式说明

这是一个 VCS（`-git`）包：`source` 指向上游 git 仓库，最终用户的
`makepkg`/AUR helper 会在本机拉取最新提交并本地编译，`pkgver()` 按
`meson.build` 中的版本号 + 提交计数 + 短 hash 自动生成。

因此本仓库**不跟随上游每次提交更新**，只在以下情况需要修改 PKGBUILD：

- 构建依赖或选项变化（`meson_options.txt`、`subprojects/`、`meson.build`）；
- 构建步骤或打包修复；
- `pkgver()` 格式调整。

## 构建特性

- 捆绑 YukariConnect（.NET NativeAOT 单文件，安装至
  `/usr/tools/yukariconnect/`），构建期需要 `dotnet-sdk` 与 `clang`，运行时无
  dotnet 依赖。
- 关闭 `build_pcl_download` / `neo_ui` / `yggdrasil` / `curseforge_webview`，
  保持依赖面最小。
- **构建需要联网**：`prepare()` 拉取 git 子模块与 YukariConnect wrap，
  `build()` 中 `dotnet publish` 需还原 NuGet 包。不支持 devtools 离线
  clean chroot 构建。

## 已知的 namcap 提示（均属预期，无需处理）

- `ELF file outside of a valid path`（`usr/tools/yukariconnect/YukariConnect`、
  `usr/imageformats/libqwebp.so`）：上游安装布局，启动器按该相对路径定位
  这些文件，不可移动。
- `Dependency included, but may not be needed ('gamemode')`：误报，
  `Application.cpp` 直接调用 `gamemode_query_status()`（链接依赖）。
- `!lto`：`/etc/makepkg.conf` 默认注入的 `-flto` 与 Qt6 protected symbol
  链接冲突；上游官方构建本就不启用 LTO，此项为对齐而非裁剪。

## 自动化

- `publish.yml`：`PKGBUILD` 变更推送到 `main` 时，先在 `archlinux:base-devel`
  容器中完整构建 + `namcap` 检查，通过后生成 `.SRCINFO` 回写本仓库，再用
  专用 SSH key 推送到 AUR。
- `healthcheck.yml`：每周一巡检——对上游最新 HEAD 试构建，并粗粒度比对上游
  `meson_options.txt` 与 PKGBUILD 中使用的选项，失败或漂移时自动开 issue。

## 凭据

| Secret | 用途 |
|---|---|
| `AUR_SSH_PRIVATE_KEY` | 推送 AUR 的专用 ed25519 私钥（对应公钥绑定在 AUR bot 账号） |

AUR 包地址：`ssh://aur@aur.archlinux.org/lunalauncher-git.git`

## 手工操作备忘

- **首次发布**：AUR 账号绑定公钥后，首次 `git push` 到上述地址即自动创建
  `lunalauncher-git` 包。
- **AUR 端被他人改动导致推送被拒**：手动
  `git pull ssh://aur@aur.archlinux.org/lunalauncher-git.git master`
  合并差异后推送回 `main`，再触发 `publish.yml`。
