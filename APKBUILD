# Contributor: daverona <egkimatwork@gmail.com>
# Maintainer: daverona <egkimatwork@gmail.com>
pkgname=rdkit
pkgver=2020.03.2
_pkgver=2020_03_2
pkgrel=0
pkgdesc="A collection of cheminformatics and machine-learning software" 
url="https://www.rdkit.org/"
arch="all"
license="BSD-3-Clause"
#options="!check"  # Don't check if the building environment is shared with others. It will start and stop postgresql server.
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
  openjdk8
  postgresql-dev
  py3-cairo 
  py3-numpy-dev 
  python3-dev
  "
checkdepends="
  postgresql
  postgresql-client
  py3-pillow
  py3-pip
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
  https://sourceforge.net/projects/swig/files/swig/swig-3.0.12/swig-3.0.12.tar.gz
  boost-above-1.56.0.patch
  "
builddir="$srcdir/rdkit-Release_$_pkgver"

prepare() {
  default_prepare
  mkdir -p "$builddir"/build
}

build() {
  # Note that swig 4, which is the stock version for Alpine 3.12, is not supported
  # by RDKit. So we install swig 3.0.12 and remove it later.
  cd "$srcdir"/swig-3.0.12
  ./configure --prefix=/usr
  make -j $(nproc)
  sudo make install

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
  # error: undefined reference to `__isascii'
  # INCHI-API is downloaded by rdkit, so it cannot be patched beforehand.
  sed -i '62d' "$builddir"/External/INCHI-API/src/INCHI_BASE/src/util.c
  sed -i '62i #define __isascii(val) ((unsigned)(val) <= 0x7F)' "$builddir"/External/INCHI-API/src/INCHI_BASE/src/util.c
  make -j $(nproc)

  cd "$srcdir"/swig-3.0.12
  sudo make uninstall
}

check() {
  cd build
  # Install RDKit for testing
  sudo make install

  # Note that pythonTestDirChem test is disabled because of a bug in the source
  # reported here: https://github.com/rdkit/rdkit/issues/2757#issue-516155570
  RDBASE="$builddir" ctest -j $(nproc) -E "testPgSQL|pythonTestDirChem"

  # Test PostgreSQL cartridge
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
  # Set permission and ownership back
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

  # Uninstall RDKit
  sudo rm -rf `cat install_manifest.txt`
  sudo rm -rf install_manifest.txt
}

package() {
  cd build
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
  # This subpackage contains shared libraries, which makes it dependent on architecture.
  pkgdesc="$pkgdesc (for Python3)"
  depends="
    $pkgname=$pkgver-r$pkgrel 
    py3-cairo 
    py3-numpy
    " 

  local pyver="${subpkgname:2:1}"
  mkdir -p "$subpkgdir"/usr/lib
  mv "$pkgdir"/usr/lib/python$pyver* "$subpkgdir"/usr/lib/

  # TODO: Remove, if possible, messages like "so:libRDKitForceField.so.1 (missing)".
  #cp -P "$pkgdir/"/usr/lib/*.so.1 "$subpkgdir"/usr/lib/
}

pgsql() {
  pkgdesc="$pkgdesc (PostgreSQL cartridge)"
  depends="$pkgname=$pkgver-r$pkgrel"

  install -D -m 644 "$builddir"/build/Code/PgSQL/rdkit/rdkit--3.8.sql "$subpkgdir"/usr/share/postgresql/extension/rdkit--3.8.sql
  install -D -m 644 "$builddir"/Code/PgSQL/rdkit/rdkit.control "$subpkgdir"/usr/share/postgresql/extension/rdkit.control
  install -D -m 755 "$builddir"/build/Code/PgSQL/rdkit/librdkit.so "$subpkgdir"/usr/lib/postgresql/librdkit.so
}

java() {
  pkgdesc="$pkgdesc (Java wrapper)"
  depends="py3-$pkgname=$pkgver-r$pkgrel"

  mkdir -p "$subpkgdir"/usr/share/rdkit
  mv "$pkgdir/$builddir"/Code/JavaWrappers "$subpkgdir"/usr/share/rdkit/
  cp "$builddir"/Code/JavaWrappers/gmwrapper/*.jar "$subpkgdir"/usr/share/rdkit/JavaWrappers/gmwrapper/
  local prefix=${builddir#/}
  rm -rf "$pkgdir/${prefix%%/*}"
}

javadoc() {
  pkgdesc="$pkgdesc (Java wrapper documentation)"
  depends=

  mkdir -p "$subpkgdir"/usr/share/doc/rdkit/JavaWrappers/gmwrapper
  cp -R "$builddir"/Code/JavaWrappers/gmwrapper/doc/* "$subpkgdir"/usr/share/doc/rdkit/JavaWrappers/gmwrapper/
}

sha512sums="01a2c41d7efe2e38256d07be4ee751d206dee68ed6331e57cb0607936669ec17454f55363e52e96dd4500c5118c89d5340463202d8e44d48e71b04e4ad59ce61  rdkit-2020.03.2.tar.gz
5eaa2e06d8e4197fd02194051db1e518325dbb074a4c55a91099ad9c55193874f577764afc9029409a41bd520a95154095f26e33ef5add5c102bb2c1d98d33eb  swig-3.0.12.tar.gz
ee863c1c94ff959c0021c0ce7cecf099b8683129070b97184e026dc71d4ad1522d4746b09453848911287c73f272cc5c07eb51aa68bc81f88a53a5adbabc612f  boost-above-1.56.0.patch"
