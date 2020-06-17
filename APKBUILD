# Contributor: daVerona
# Maintainer:
pkgname="rdkit"
pkgver="2020_03_3"
pkgrel=0
pkgdesc="a collection of cheminformatics and machine-learning software" 
url="https://www.rdkit.org/"
arch="all"
license="BSD 3-Clause License"
depends=""
makedepends=""
install=""
subpackages="$pkgname-dev $pkgname-doc"
source="https://github.com/rdkit/$pkgname/archive/$pkgver.tar.gz"
builddir="$srcdir/$pkgname-$pkgver"

prepare() {
  default_prepare
}

build() {
  cd "$builddir/build"
  cmake .. \
    -Wno-dev \
    -DRDK_INSTALL_INTREE=OFF \
    -DRDK_BUILD_CAIRO_SUPPORT=ON \
    -DRDK_BUILD_INCHI_SUPPORT=ON \
    -DPYTHON_EXECUTABLE=/usr/bin/python \
    -DPYTHON_INCLUDE_DIR=/usr/include/python \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DCMAKE_BUILD_TYPE=Release
  # Patch
  make -j $(nproc)
}

check() {
  cd "$builddir"
  make check
}

package() {
  cd "$builddir"
  make DESTDIR="$pkgdir" install
}

sha512sums="a8f87e21f654d482a67709da273cf58808af377cfcb20f189986bad58530e382f8f1d3a5eaf33aa02284015a6fa7ccfc7fc383e487de6e0e4f57405a16537217  ssdeep-2.14.1.tar.gz"
