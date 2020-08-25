# Contributor: daverona <egkimatwork@gmail.com>
# Maintainer: daverona <egkimatwork@gmail.com>
pkgname=rdkit
pkgver=2020.03.5
_pkgver=2020_03_5
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
  swig
  "
checkdepends="
  gfortran 
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
  cd build
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
}

check() {
  cd build
  # Install check dependencies which cannot be specified in $checkdepends
  #local tmpprev=$(mktemp)
  #local tmpcurr=$(mktemp)
  #local tmpdiff=$(mktemp)
  #pip3 freeze | sort >> "$tmpprev"
  #sudo pip3 install --verbose wheel "pandas==1.1.0"
  #pip3 freeze | sort >> "$tmpcurr"
  #comm -3 "$tmpprev" "$tmpcurr" | sed "s|^\t||" >> "$tmpdiff"
  #rm -rf "$tmpprev" "$tmpcurr"
  sudo make install
 
  # TODO: Do pythonTestDirChem after installing pandas
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

  # Uninstall check dependencies
  sudo rm -rf `cat install_manifest.txt`
  sudo rm -rf install_manifest.txt
  #sudo pip3 uninstall --yes --requirement "$tmpdiff"
  #rm -rf "$tmpdiff"
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

sha512sums="a95d100280fb9d1fb95fbf54bf47c259c234f931bfe857feba87bd3e9304753c64c4c4c8d52a336d2543a5635c0c6b60661dea32fca866278fcce0fc0e0152d2  rdkit-2020.03.5.tar.gz
4dfdb72aa75f8d77f95cc458f48ca2207e77f5cf2e758ff0750ddc68a32c34301376957c12a442f9f199736c07111d8c2d7cda790051d54eece717c7c35faecc  boost-above-1.56.0.patch"
