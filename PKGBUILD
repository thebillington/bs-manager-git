# Maintainer: Pierce Thompson <pierce at insprill dot net>
# Fork: aarch64 (ARM64 Steam Frame) build pipeline of bs-manager-git.
# Point _branch=master for pristine upstream builds.

pkgname=bs-manager-git
pkgver=v1.6.0.r14.g3fc13bc
pkgrel=1
pkgdesc="An all-in-one tool for managing Beat Saber versions, maps, mods, and more"
arch=("x86_64" "aarch64")
url="https://github.com/Zagrios/bs-manager"
license=('GPL')
depends=()
makedepends=('git' 'pnpm' 'corepack' 'libxcrypt-compat' 'rust' 'musl')
provides=("${pkgname%-git}")
conflicts=("${pkgname%-git}")
options=('!strip') # DepotDownloader/bs-downloader breaks without this

_repo="https://github.com/thebillington/bs-manager.git"
_branch="fix/arm64-wine-path-main"

source=(
  "git+${_repo}#branch=${_branch}"
  "${pkgname%-git}.desktop"
)
sha256sums=(
  'SKIP'
  'cb35ac15f308e0dca35aa2a948f3102eb96ec0c9faa1771b91d5a49309398874'
)

pkgver() {
    cd "${pkgname%-git}"
    git describe --long --tags --abbrev=7 | sed 's/\([^-]*-g\)/r\1/;s/-/./g'
}

_prepare_corepack() {
    local corepack_home="${srcdir}/.corepack"
    export COREPACK_HOME="${srcdir}/.corepack-cache"

    mkdir -p "$corepack_home"
    corepack enable --install-directory "$corepack_home"

    export PATH="$corepack_home:$PATH"
}

prepare() {
    _prepare_corepack
    cd "${srcdir}/${pkgname%-git}"

    corepack install
}

build() {
    _prepare_corepack
    cd "${pkgname%-git}"

    # makepkg CFLAGS carry -flto=auto, which turns aws-lc-sys C objects into
    # LTO bytecode the plain musl-gcc link cannot materialize. Strip it.
    export CFLAGS="${CFLAGS/ -flto=auto/}"
    export LDFLAGS="${LDFLAGS/ -flto=auto/}"

    corepack pnpm install --frozen-lockfile
    corepack pnpm run build-rust-scripts
    corepack pnpm run build
    if [ "${CARCH}" = "x86_64" ]; then
        corepack pnpm exec electron-builder --config electron-builder.config.js --publish never --linux pacman --x64
    else
        corepack pnpm exec electron-builder --config electron-builder.config.js --publish never --linux pacman --arm64
    fi
}

package() {
    cd "${pkgname%-git}"

    install -d "$pkgdir/opt/${pkgname%-git}"
    if [ "${CARCH}" = "x86_64" ]; then
        cp -r "release/build/linux-unpacked/". "$pkgdir/opt/${pkgname%-git}/"
    else
        cp -r "release/build/linux-arm64-unpacked/". "$pkgdir/opt/${pkgname%-git}/"
    fi

    install -Dm644 "$srcdir/${pkgname%-git}.desktop" "$pkgdir/usr/share/applications/${pkgname%-git}.desktop"
    install -Dm644 "resources/readme/SVG/icon.svg" "$pkgdir/usr/share/pixmaps/${pkgname%-git}.svg"
    install -Dm644 "LICENSE" -t "$pkgdir/usr/share/licenses/${pkgname%-git}"
    install -Dm644 "README.md" -t "$pkgdir/usr/share/doc/${pkgname%-git}"
}