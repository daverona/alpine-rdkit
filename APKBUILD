# Contributor: daverona
# Maintainer:
pkgname=rdkit
pkgver=2020.03.3
_pkgver=2020_03_3
pkgrel=0
pkgdesc="A collection of cheminformatics and machine-learning software" 
url="https://www.rdkit.org/"
arch="all"
license="BSD-3-Clause"
#options="!check"  # Don't call check() if the building environment is shared with others. It starts and stops postgresql server.
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
  postgresql-dev
  py-numpy-dev 
  py3-cairo 
  py3-numpy 
  python3-dev
  "
checkdepends="
  diffutils
  gfortran 
  postgresql
  postgresql-client
  py3-pillow
  "
subpackages="
  $pkgname-data:data:noarch
  py3-$pkgname:py3 
  $pkgname-pgsql
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
    -DRDK_BUILD_PGSQL=ON \
    -Wno-dev
  # error: undefined reference to `__isascii'
  # INCHI-API is downloaded by rdkit, so it cannot be patched beforehand.
  sed -i '62d' "$builddir"/External/INCHI-API/src/INCHI_BASE/src/util.c
  sed -i '62i #define __isascii(val) ((unsigned)(val) <= 0x7F)' "$builddir"/External/INCHI-API/src/INCHI_BASE/src/util.c
  make -j $(nproc)
}

check() {
  cd build
  # Install check dependencies which cannot be specified in $checkdepends
  sudo pip3 install wheel pandas
  sudo make install
  RDBASE="$builddir" ctest -j $(nproc) -E testPgSQL

  # Do PostgreSQL test
  # Install the cartridge
  sudo sh "$builddir"/build/Code/PgSQL/rdkit/pgsql_install.sh
  # Start the server
  sudo mkdir -p /run/postgresql /tmp/postgresql
  sudo chown postgres:postgres /run/postgresql /tmp/postgresql
  sudo -u postgres initdb -D /tmp/postgresql
  sudo -u postgres pg_ctl -D /tmp/postgresql start
  # Set permission of files and directories
  sudo chmod o+w -R "$builddir"/build/Testing/Temporary
  sudo chmod o+w -R "$builddir"/build/Code/PgSQL/rdkit
  # Do test
  sudo RDBASE="$builddir" -u postgres ctest -j $(nproc) -R testPgSQL
  # Set permission and ownership
  sudo chmod o-w -R "$builddir"/build/Testing/Temporary
  sudo chmod o-w -R "$builddir"/build/Code/PgSQL/rdkit
  sudo chown -R $(id -u):$(id -g) "$builddir"/build/Testing/Temporary
  sudo chown -R $(id -u):$(id -g) "$builddir"/build/Code/PgSQL/rdkit
  # Stop the server
  sudo -u postgres pg_ctl -D /tmp/postgresql stop
  sudo rm -rf /run/postgresql /tmp/postgresql
  # Delete the cartridge
  sudo rm -rf /usr/share/postgresql/extension/rdkit*
  sudo rm -rf /usr/lib/postgresql/librdkit.so

  # Uninstall check dependencies
  sudo rm -rf `cat install_manifest.txt`
  sudo rm -rf install_manifest.txt
  sudo pip3 uninstall --yes wheel pandas
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
  # TODO: Remove, if possible, messages like "so:libRDKitForceField.so.1 (missing)".
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

pgsql() {
  pkgdesc="$pkgdesc (PostgreSQL cartridge)"
  depends="$pkgname=$pkgver-r$pkgrel"

  install -D "$builddir"/build/Code/PgSQL/rdkit/rdkit--3.8.sql "$subpkgdir"/usr/share/postgresql/extension/rdkit--3.8.sql
  install -D "$builddir"/Code/PgSQL/rdkit/rdkit.control "$subpkgdir"/usr/share/postgresql/extension/rdkit.control
  install -D -m 0755 "$builddir"/build/Code/PgSQL/rdkit/librdkit.so "$subpkgdir"/usr/lib/postgresql/librdkit.so
}

sha512sums="7f5d626d52e360551a62de9c0df5055c74b2022ce874ef3077a2dfc95f7a6b99430428c787d45978563fcc174f00b30c1dbc7d9f8e19d82aa28f1c62d5408d59  rdkit-2020.03.3.tar.gz"
