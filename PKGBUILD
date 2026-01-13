# Maintainer: Ezozbek Rasulov <arch@ezozbek.dev>
pkgname=bleur
pkgver=0.0.4
pkgrel=1
pkgdesc="That buddy that will get everything ready for you"
arch=('x86_64')
url="https://github.com/bleur-org/bleur"
license=('MIT' 'Apache-2.0')
depends=('libgit2' 'libssh2' 'openssl' 'zstd' 'xz')
makedepends=('cargo' 'cmake')
source=(
   "$pkgname-$pkgver.tar.gz::https://github.com/bleur-org/bleur/archive/refs/tags/v$pkgver.tar.gz"
   "native-tls.patch"
)
sha256sums=('aa91cc7352c0f185ce9d334ca32d6b3c0647fd3480552af8ca148576be579b09'
            'ec05107fdcc4585d516d66172f686e2b9f9be2198f6b3ee357632205cab39823')

prepare() {
    cd "$pkgname-$pkgver"

    # Apply the patch to switch from rustls to native-tls (OpenSSL)
    patch -p1 -i "$srcdir/native-tls.patch"

    export RUSTUP_TOOLCHAIN=stable

    # Fetch dependencies ahead of time
    cargo fetch --target "$CARCH-unknown-linux-gnu"
}

build() {
    cd "$pkgname-$pkgver"
    export RUSTUP_TOOLCHAIN=stable
    export CARGO_TARGET_DIR=target

    # Ensure pkg-config can find system libraries
    export PKG_CONFIG_PATH="/usr/lib/pkgconfig:/usr/share/pkgconfig"
    export PKG_CONFIG_ALLOW_SYSTEM_LIBS=1
    export PKG_CONFIG_ALLOW_SYSTEM_CFLAGS=1

    # FORCE usage of system libraries
    export LIBSSH2_SYS_USE_PKG_CONFIG=1
    export ZSTD_SYS_USE_PKG_CONFIG=1
    export LIBLZMA_SYS_USE_PKG_CONFIG=1
    export LIBGIT2_NO_VENDOR=1

    # Explicitly add lzma to linker flags
    export RUSTFLAGS="-C link-arg=-L/usr/lib -C link-arg=-llzma"

    # Build release binary
    cargo build --frozen --release --all-features
}

check() {
    cd "$pkgname-$pkgver"

    export RUSTUP_TOOLCHAIN=stable
    export CARGO_TARGET_DIR=target

    # Run tests to ensure package integrity
    cargo test --frozen --all-features
}

package() {
    cd "$pkgname-$pkgver"

    # Install binary
    install -Dm755 "target/release/$pkgname" "$pkgdir/usr/bin/$pkgname"

    # Install licenses
    install -Dm644 LICENSE-MIT "$pkgdir/usr/share/licenses/$pkgname/LICENSE-MIT"
    install -Dm644 LICENSE-APACHE "$pkgdir/usr/share/licenses/$pkgname/LICENSE-APACHE"
}
