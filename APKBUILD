# Contributor: daVerona
# Maintainer:
pkgname="rdkit"
pkgver="2020.03.3"
_pkgver="2020_03_3"
pkgrel=0
pkgdesc="a collection of cheminformatics and machine-learning software" 
url="https://www.rdkit.org/"
arch="all"
license="BSD 3-Clause License"
depends=""
makedepends="boost-dev cairo-dev cmake eigen-dev py-numpy-dev python3-dev"
install=""
subpackages="$pkgname-dev $pkgname-doc"
source="https://github.com/rdkit/$pkgname/archive/$_pkgver.tar.gz"
builddir="$srcdir/$pkgname-$_pkgver"

prepare() {
  default_prepare
}

build() {
  cd "$builddir"
  mkdir build
  cd build
  cmake .. \
    -Wno-dev \
    -DRDK_INSTALL_INTREE=OFF \
    -DRDK_BUILD_CAIRO_SUPPORT=ON \
    -DRDK_BUILD_INCHI_SUPPORT=ON \
    -DPYTHON_EXECUTABLE=/usr/bin/python3 \
    -DPYTHON_INCLUDE_DIR="$(python3 -c 'from sysconfig import get_paths; print(get_paths()["include"])')" \
    -DPYTHON_NUMPY_INCLUDE_PATH="$(python3 -c 'import numpy; print(numpy.get_include())')" \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DCMAKE_BUILD_TYPE=Release
  # Patch isascii
  sed -i "s|__isascii|isascii|" ../External/INCHI-API/src/INCHI_BASE/src/util.c
  make -j $(nproc)
  make install
}

check() {
  cd "$builddir/build"
  RDBASE="$builddir" ctest
}

package() {
  cd "$builddir/build"
  make DESTDIR="$pkgdir" install
}

sha512sums="7b8843f53dbc81969977ed6a941ce089a8871ef3ab58d8aab559415776208905a8b82b2480818592c5de53965c3b62e7d4bd083119cbcfa288b3828a0a9a6354  2020_03_3.tar.gz"
