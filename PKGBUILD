# Maintainer: Marios Andreopoulos <opensource at andmarios dot com>
# Based on the official libfprint PKGBUILD by Jan Alexander Steffens (heftig)
#
# libfprint with the "goodixtls" driver for the Goodix GXFP5187 SPI fingerprint
# sensor of the Huawei MateBook X Pro 2018 (MACH-WX9). The driver started as
# https://github.com/Sigfrodr/libfprint-goodixtls by Benjamin Allègre; the
# patches carry its history and authorship.

pkgname=libfprint-gxfp5187
_pkgname=libfprint
pkgver=1.94.100
pkgrel=1
pkgdesc="Library for fingerprint readers, with a driver for the Goodix GXFP5187 SPI sensor (Huawei MateBook X Pro 2018)"
url="https://fprint.freedesktop.org/"
arch=(x86_64)
license=(LGPL-2.1-or-later)
depends=(
  libgcc
  glib2
  glibc
  libgudev
  libgusb
  openssl
  pixman
)
makedepends=(
  git
  glib2-devel
  gobject-introspection
  meson
  python-cairo
  python-gobject
  systemd
)
checkdepends=(
  cairo
  umockdev
)
provides=("libfprint=$pkgver" libfprint-2.so)
conflicts=(libfprint)
groups=(fprint)
install=$pkgname.install
source=(
  "git+https://gitlab.freedesktop.org/libfprint/libfprint.git#tag=v$pkgver"
  0001-goodixtls-import-the-Goodix-GXFP5187-driver-by-Benja.patch
  0002-goodixtls-build-the-driver-in-tree.patch
  0003-goodixtls-reliable-transport-session-reuse-and-a-rot.patch
  60-goodixtls-spidev.rules
  goodixtls-spidev.modprobe.conf
  goodixtls-spidev.modules-load.conf
  fprintd-goodixtls.conf
)
sha256sums=('18fcee9ffb2adb9ba2ea229850e9266d0bcad2df5fad556beffce6c8ba84f192'
            '41eaf706f13caf1598f3d2f9687fcd78e9d40b6025b66263ffc4beabf96e8b4c'
            '8c3bfd631382573e3bc02fd0d77372622ca11adc6d7a83909005c20b53cb5135'
            'b15f1b873ffa595d1ec0c6c9ef58bc688bd5b1795cbac7b1412c594d0da785a9'
            '5da59e7189c12e5c022f9726827f903944b7059fbd473551dbf52a014c0a71dc'
            '3582641437341e2f27814a89c7a1bb6b91a5a3f6599e63f5696253c0d737d3bd'
            '9731f3b814e0282987ab3cd939c7484485cebae05b0a246d3ffb922d1a5c32ad'
            '7e0e49a3743a5e8ef6d560c469bf7a1c29fd45e217aa4a1581070d30a65a8d00')

prepare() {
  cd $_pkgname
  local p
  for p in "$srcdir"/0*.patch; do
    git apply "$p"
  done
}

build() {
  local meson_options=(
    -D drivers=all
    -D installed-tests=false
    -D doc=false
  )

  arch-meson $_pkgname build "${meson_options[@]}"
  meson compile -C build
}

check() {
  # Serially: under parallel load the umockdev replay of the (unrelated)
  # egis_etu905 test can time out.
  meson test -C build --print-errorlogs --num-processes 1
}

package() {
  meson install -C build --destdir "$pkgdir"

  install -Dm644 60-goodixtls-spidev.rules \
    "$pkgdir/usr/lib/udev/rules.d/60-goodixtls-spidev.rules"
  install -Dm644 goodixtls-spidev.modprobe.conf \
    "$pkgdir/usr/lib/modprobe.d/goodixtls-spidev.conf"
  install -Dm644 goodixtls-spidev.modules-load.conf \
    "$pkgdir/usr/lib/modules-load.d/goodixtls-spidev.conf"
  install -Dm644 fprintd-goodixtls.conf \
    "$pkgdir/usr/lib/systemd/system/fprintd.service.d/goodixtls.conf"
}

# vim:set sw=2 sts=-1 et:
