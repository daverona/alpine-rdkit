# Contributor: daverona <daverona@nowhere.on.earth>
# Maintainer: daverona <daverona@nowhere.on.earth>
pkgname="py3-rdkit"
_pkgname="rdkit"
pkgver="2020.03.3"
_pkgver="2020_03_3"
pkgrel=0
pkgdesc="A collection of cheminformatics and machine-learning software" 
url="https://www.rdkit.org/"
arch="all"
license="BSD 3-Clause License"
depends=""
makedepends="boost-dev cairo-dev cmake eigen-dev py-numpy-dev py3-numpy py3-pillow python3-dev"
checkdepends="py3-pillow"
#subpackages="$pkgname-dev $pkgname-doc"
source="https://github.com/rdkit/$_pkgname/archive/Release_$_pkgver.tar.gz"
builddir="$srcdir/$_pkgname-Release_$_pkgver"

prepare() {
  default_prepare
}

build() {
  mkdir -p "$builddir/build"
  cd "$builddir/build"
  RDBASE=/usr cmake .. \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DCMAKE_BUILD_TYPE=Release \
    -DPYTHON_EXECUTABLE=/usr/bin/python3 \
    -DPYTHON_INCLUDE_DIR="$(python3 -c 'from sysconfig import get_paths; print(get_paths()["include"])')" \
    -DPYTHON_NUMPY_INCLUDE_PATH="$(python3 -c 'import numpy; print(numpy.get_include())')" \
    -DRDK_INSTALL_INTREE=OFF \
    -DRDK_BUILD_CAIRO_SUPPORT=ON \
    -DRDK_BUILD_INCHI_SUPPORT=ON \
    -Wno-dev
  # patch isascii
  sed -i "s|__isascii|isascii|" ../External/INCHI-API/src/INCHI_BASE/src/util.c
  make -j $(nproc)
}

check() {
  cd "$builddir/build"
  sudo make install
  RDBASE="$builddir" ctest
}

package() {
  cd "$builddir/build"
  make DESTDIR="$pkgdir" install
}

sha512sums="7f5d626d52e360551a62de9c0df5055c74b2022ce874ef3077a2dfc95f7a6b99430428c787d45978563fcc174f00b30c1dbc7d9f8e19d82aa28f1c62d5408d59  Release_2020_03_3.tar.gz"
