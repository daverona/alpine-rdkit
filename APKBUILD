# Contributor: daverona <egkimatwork@gmail.com>
# Maintainer: daverona <egkimatwork@gmail.com>
pkgname=rdkit
pkgver=2020.03.1
_pkgver=2020_03_1
pkgrel=0
pkgdesc="A collection of cheminformatics and machine-learning software"
url="https://www.rdkit.org/"
arch="all"
license="BSD-3-Clause"
#options="!check"  # Don't check if the building environment is shared with others. It will start and stop postgresql server.
depends=
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
  openjdk8
  postgresql-dev
  py3-cairo
  py3-numpy-dev
  python3-dev
  swig
  "
checkdepends="
  postgresql
  postgresql-client
  py3-pillow
  "
subpackages="
  $pkgname-doc:doc:noarch
  $pkgname-java-doc:javadoc:noarch
  $pkgname-data:data:noarch
  py3-$pkgname:py3
  $pkgname-java
  $pkgname-pgsql
  $pkgname-static
  $pkgname-dev
  "
source="
  rdkit-$pkgver.tar.gz::https://github.com/rdkit/rdkit/archive/Release_$_pkgver.tar.gz
  boost-above-1.56.0.patch
  "
builddir="$srcdir/rdkit-Release_$_pkgver"

prepare() {
  default_prepare
  mkdir -p "$builddir"/build
}

build() {
  cd "$builddir"/build
  RDBASE=/usr \
  PATH=/usr/lib/jvm/default-jvm/bin:$PATH \
  JAVA_HOME=/usr/lib/jvm/default-jvm \
  cmake .. \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DCMAKE_BUILD_TYPE=RELEASE \
    -DPYTHON_EXECUTABLE=/usr/bin/python3 \
    -DRDK_INSTALL_INTREE=OFF \
    -DRDK_BUILD_AVALON_SUPPORT=ON \
    -DRDK_BUILD_CAIRO_SUPPORT=ON \
    -DRDK_BUILD_FREESASA_SUPPORT=ON \
    -DRDK_BUILD_INCHI_SUPPORT=ON \
    -DRDK_BUILD_PGSQL=ON \
    -DRDK_BUILD_SWIG_WRAPPERS=ON \
    -DRDK_BUILD_TEST_GZIP=ON \
    -Wno-dev
  # Note that INCHI-API is downloaded by rdkit, so it cannot be patched beforehand.
  sed -i '62d' "$builddir"/External/INCHI-API/src/INCHI_BASE/src/util.c
  sed -i '62i #define __isascii(val) ((unsigned)(val) <= 0x7F)' "$builddir"/External/INCHI-API/src/INCHI_BASE/src/util.c
  make -j $(nproc)
}

check() {
  cd "$builddir"/build
  # Install check dependencies which cannot be specified in $checkdepends
  sudo make install

  # Note that pythonTestDirChem test is disabled because of a bug in the source
  # reported here: https://github.com/rdkit/rdkit/issues/2757#issue-516155570
  RDBASE="$builddir" ctest -j $(nproc) --output-on-failure -E "testPgSQL|pythonTestDirChem"

  # Test PostgreSQL cartridge
  # Install the cartridge
  sudo sh "$builddir"/build/Code/PgSQL/rdkit/pgsql_install.sh
  # Start the server
  local _pgtmp="$(mktemp -d)"
  sudo mkdir -p /run/postgresql $_pgtmp
  sudo chown postgres:postgres /run/postgresql $_pgtmp
  sudo -u postgres initdb -D $_pgtmp
  sudo -u postgres pg_ctl -D $_pgtmp start
  # Set permission of files and directories
  sudo chmod o+w -R "$builddir"/build/Testing/Temporary
  sudo chmod o+w -R "$builddir"/build/Code/PgSQL/rdkit
  # Do test
  sudo RDBASE="$builddir" -u postgres ctest -j $(nproc) --output-on-failure -R testPgSQL
  # Set permission and ownership back
  sudo chmod o-w -R "$builddir"/build/Testing/Temporary
  sudo chmod o-w -R "$builddir"/build/Code/PgSQL/rdkit
  sudo chown -R $(id -u):$(id -g) "$builddir"/build/Testing/Temporary
  sudo chown -R $(id -u):$(id -g) "$builddir"/build/Code/PgSQL/rdkit
  # Stop the server
  sudo -u postgres pg_ctl -D $_pgtmp stop
  sudo rm -rf /run/postgresql $_pgtmp
  # Delete the cartridge
  sudo rm -rf /usr/share/postgresql/extension/rdkit*
  sudo rm -rf /usr/lib/postgresql/librdkit.so

  # Uninstall check dependencies
  sudo rm -rf `cat install_manifest.txt`
  sudo rm -rf install_manifest.txt
}

package() {
  cd "$builddir"/build
  make DESTDIR="$pkgdir" install

  mv "$pkgdir"/usr/share/RDKit "$pkgdir"/usr/share/rdkit
  mkdir -p "$pkgdir"/usr/share/doc
  mv "$pkgdir"/usr/share/rdkit/Docs "$pkgdir"/usr/share/doc/rdkit
  cp "$pkgdir"/usr/share/rdkit/license.txt "$pkgdir"/usr/share/doc/rdkit/license.txt
}

data() {
  pkgdesc="$pkgdesc (data files)"
  depends=

  mkdir -p "$subpkgdir"/usr
  mv "$pkgdir"/usr/share "$subpkgdir"/usr/
}

py3() {
  pkgdesc="$pkgdesc (for Python3)"
  depends="
    py3-cairo
    py3-numpy
    "

  local pyver="${subpkgname:2:1}"
  mkdir -p "$subpkgdir"/usr/lib
  mv "$pkgdir"/usr/lib/python$pyver* "$subpkgdir"/usr/lib/
}

pgsql() {
  pkgdesc="$pkgdesc (PostgreSQL cartridge)"
  depends=

  install -Dm755 "$builddir"/build/Code/PgSQL/rdkit/librdkit.so "$subpkgdir"/usr/lib/postgresql/librdkit.so
  install -Dm644 "$builddir"/build/Code/PgSQL/rdkit/rdkit--3.8.sql "$subpkgdir"/usr/share/postgresql/extension/rdkit--3.8.sql
  install -Dm644 "$builddir"/Code/PgSQL/rdkit/rdkit.control "$subpkgdir"/usr/share/postgresql/extension/rdkit.control
}

java() {
  pkgdesc="$pkgdesc (Java wrapper)"
  depends=

  install -Dm755 "$pkgdir/$builddir"/Code/JavaWrappers/gmwrapper/libGraphMolWrap.so "$subpkgdir"/usr/share/rdkit/JavaWrappers/gmwrapper/libGraphMolWrap.so
  install -Dm644 "$builddir"/Code/JavaWrappers/gmwrapper/org.RDKit.jar "$subpkgdir"/usr/share/rdkit/JavaWrappers/gmwrapper/org.RDKit.jar
  install -Dm644 "$builddir"/Code/JavaWrappers/gmwrapper/org.RDKitDoc.jar "$subpkgdir"/usr/share/rdkit/JavaWrappers/gmwrapper/org.RDKitDoc.jar
  local prefix=${builddir#/}
  rm -rf "$pkgdir/${prefix%%/*}"
}

javadoc() {
  pkgdesc="$pkgdesc (Java wrapper documentation)"
  depends=

  mkdir -p "$subpkgdir"/usr/share/doc/rdkit/JavaWrappers
  cp -R "$builddir"/Code/JavaWrappers/gmwrapper/doc "$subpkgdir"/usr/share/doc/rdkit/JavaWrappers/gmwrapper
}

sha512sums="e62c8b45753eb59716e6d008c5bf77b0fff2a0533919478dcce23b3a5fd760f9958b4de750982c97cc5ff737c0170789ad295a433276cb927634e813dfaafd54  rdkit-2020.03.1.tar.gz
ee863c1c94ff959c0021c0ce7cecf099b8683129070b97184e026dc71d4ad1522d4746b09453848911287c73f272cc5c07eb51aa68bc81f88a53a5adbabc612f  boost-above-1.56.0.patch"
