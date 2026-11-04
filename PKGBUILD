# Maintainer: AndreaFrederica <andreafrederica@outlook.com>
pkgname=lunalauncher-git
pkgver=11.0.2.r11406.g9176c13fb
pkgrel=1
pkgdesc='A custom Minecraft launcher based on Prism Launcher (git build with bundled YukariConnect)'
arch=('x86_64')
url='https://github.com/AndreaFrederica/LunaLauncher'
license=('GPL-3.0-or-later')
options=('!debug')
depends=(
  'gamemode'
  'hicolor-icon-theme'
  'java-runtime>=8'
  'libarchive'
  'qrencode'
  'qt6-base'
  'qt6-imageformats'
  'qt6-multimedia'
  'qt6-networkauth'
  'qt6-svg'
  'qt6-websockets'
  'tomlplusplus'
  'zlib'
)
makedepends=(
  'clang'           # YukariConnect NativeAOT 发布需要
  'cmake'           # meson 以 cmake 方式探测 Qt6
  'dotnet-sdk'      # YukariConnect 子项目构建
  'git'
  'java-environment'
  'meson'
  'ninja'
  'pkgconf'
)
provides=('lunalauncher')
conflicts=('lunalauncher')
source=('git+https://github.com/AndreaFrederica/LunaLauncher.git')
sha256sums=('SKIP')

pkgver() {
  cd LunaLauncher
  local version rev_count short_hash
  version=$(sed -n "s/^[[:space:]]*version: '\([^']*\)'.*/\1/p" meson.build | head -n1)
  rev_count=$(git rev-list --count HEAD)
  short_hash=$(git rev-parse --short HEAD)
  printf '%s.r%s.g%s\n' "$version" "$rev_count" "$short_hash"
}

prepare() {
  cd LunaLauncher
  git submodule update --init --recursive
  # 预取 YukariConnect wrap 子项目，保证 build() 阶段无需访问 git 远端。
  # 注意：dotnet publish 仍需联网还原 NuGet 包，本包不支持离线 chroot 构建。
  meson subprojects download yukariconnect
}

build() {
  meson setup build LunaLauncher --prefix=/usr --buildtype=release \
    -Dbuild_testing=false \
    -Dbuild_yukari_connect=true \
    -Dbuild_pcl_download=false \
    -Dneo_ui=disabled \
    -Dyggdrasil=disabled \
    -Dcurseforge_webview=disabled \
    -Dgamemode=enabled
  meson compile -C build
}

package() {
  DESTDIR="$pkgdir" meson install -C build --no-rebuild

  # quickjs 在构建期静态链接进启动器；其开发产物与 CLI 工具不属于运行时。
  rm -f \
    "$pkgdir/usr/bin/qjs" \
    "$pkgdir/usr/bin/qjsc" \
    "$pkgdir/usr/include/quickjs.h" \
    "$pkgdir/usr/include/quickjs-libc.h" \
    "$pkgdir/usr/lib/libqjs.a"

  rm -rf \
    "$pkgdir/usr/lib/cmake/quickjs" \
    "$pkgdir/usr/share/doc/quickjs"

  rmdir --ignore-fail-on-non-empty \
    "$pkgdir/usr/include" \
    "$pkgdir/usr/lib/cmake" \
    "$pkgdir/usr/lib" \
    "$pkgdir/usr/share/doc"
}
