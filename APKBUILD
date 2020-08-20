# Contributor: daverona
# Maintainer:
pkgname=rdkit
pkgver=2020.03.3
_pkgver=2020_03_3
pkgrel=0
pkgdesc="A collection of cheminformatics and machine-learning software" 
url="https://www.rdkit.org/"
arch=all
license=BSD-3-Clause
depends="
  boost-iostreams 
  boost-python3 
  boost-serialization 
  cairo 
  "
depends_dev="
  boost-dev
  cairo-dev
  eigen-dev
  "
makedepends="
  boost-dev 
  cairo-dev 
  cmake 
  eigen-dev 
  py-numpy-dev 
  py3-cairo 
  py3-numpy 
  python3-dev
  "
checkdepends="
  gfortran 
  py3-pillow
  "
subpackages="
  $pkgname-data:data:noarch
  py3-$pkgname:py3 
  $pkgname-static 
  $pkgname-dev
  "
source="rdkit-$pkgver.tar.gz::https://github.com/rdkit/rdkit/archive/Release_$_pkgver.tar.gz"
builddir="$srcdir/rdkit-Release_$_pkgver"

prepare() {
  default_prepare
  mkdir -p "$builddir"/build
}

build() {
  cd build
  RDBASE=/usr \
  cmake .. \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DCMAKE_BUILD_TYPE=RELEASE \
    -DPYTHON_EXECUTABLE=/usr/bin/python3 \
    -DRDK_INSTALL_INTREE=OFF \
    -DRDK_BUILD_AVALON_SUPPORT=ON \
    -DRDK_BUILD_CAIRO_SUPPORT=ON \
    -DRDK_BUILD_INCHI_SUPPORT=ON \
    -Wno-dev
  # error: undefined reference to `__isascii'
  # INCHI-API is downloaded by rdkit, so it cannot be patched beforehand.
  sed -i '62d' "$builddir"/External/INCHI-API/src/INCHI_BASE/src/util.c
  sed -i '62i #define __isascii(val) ((unsigned)(val) <= 0x7F)' "$builddir"/External/INCHI-API/src/INCHI_BASE/src/util.c
  make -j $(nproc)
}

check() {
  cd build
  sudo pip3 install wheel pandas
  sudo make install
  RDBASE="$builddir" ctest -j $(nproc)
  sudo rm -rf install_manifest.txt
}

package() {
  cd build
  make DESTDIR="$pkgdir" install
}

data() {
  pkgdesc="$pkgdesc (data files)"
  depends=

  mkdir -p "$subpkgdir"/usr/share
  mv "$pkgdir"/usr/share/RDKit "$subpkgdir"/usr/share/
}

py3() {
  # This subpackage contains shared libraries, which makes it dependent on architecture.
  pkgdesc="$pkgdesc (for python3)"
  depends="
    $pkgname=$pkgver-r$pkgrel 
    $pkgname-data=$pkgver-r$pkgrel
    py3-cairo 
    py3-numpy
    " 

  local sitedir="$(python3 -c 'import site; print(site.getsitepackages()[0])')"
  mkdir -p "$subpkgdir/$sitedir"
  mv "$pkgdir/$sitedir/$pkgname" "$subpkgdir/$sitedir/"
}

sha512sums="7f5d626d52e360551a62de9c0df5055c74b2022ce874ef3077a2dfc95f7a6b99430428c787d45978563fcc174f00b30c1dbc7d9f8e19d82aa28f1c62d5408d59  rdkit-2020.03.3.tar.gz"
