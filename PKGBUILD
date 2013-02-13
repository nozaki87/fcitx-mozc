# Maintainer: Felix Yan <felixonmars@gmail.com>
# Contributor: ponsfoot <cabezon dot hashimoto at gmail dot com>

## Mozc compile option
_bldtype=Release

_zipcoderel=201211 #201212 is broken, for now
_protobuf_ver=2.4.1

_pkgbase=mozc
pkgname=fcitx-mozc
pkgdesc="Fcitx Module of A Japanese Input Method for Chromium OS, Windows, Mac and Linux (the Open Source Edition of Google Japanese Input)"
pkgver=1.6.1187.102
_patchver=${pkgver}.3
pkgrel=6
arch=('i686' 'x86_64')
url="http://code.google.com/p/mozc/"
license=('custom')
depends=('qt' 'fcitx' 'zinnia')
makedepends=('pkg-config' 'python2' 'gtest' 'qt' 'curl' 'fcitx' 'gtk2')
replaces=('mozc-fcitx')
conflicts=('mozc' 'mozc-server' 'mozc-utils-gui' 'mozc-fcitx')
source=(http://mozc.googlecode.com/files/mozc-${pkgver}.tar.bz2
        http://downloads.sourceforge.net/pnsft-aur/ken_all-${_zipcoderel}.zip
        http://downloads.sourceforge.net/pnsft-aur/jigyosyo-${_zipcoderel}.zip
        http://protobuf.googlecode.com/files/protobuf-${_protobuf_ver}.tar.bz2
        http://fcitx.googlecode.com/files/fcitx-mozc-${_patchver}.patch
)

build() {
  cd "$srcdir"
  ln -sf `which python2` ./python
  PATH="${srcdir}:${PATH}"

  # Fix compatibility with google-glog 0.3.3 (symbol conflict)
  #CFLAGS="${CFLAGS} -DFLAGS_log_dir=FLAGS_mozc_log_dir"
  #CXXFLAGS="${CXXFLAGS} -DFLAGS_log_dir=FLAGS_mozc_log_dir"
  CFLAGS="${CFLAGS} -fvisibility=hidden"
  CXXFLAGS="${CXXFLAGS} -fvisibility=hidden"

  cd "${srcdir}/${_pkgbase}-${pkgver}"

  rm unix/fcitx -rf
  patch -Np2 -i ${srcdir}/fcitx-mozc-${_patchver}.patch

  # Generate zip code seed
  msg "Generating zip code seed..."
  python2 dictionary/gen_zip_code_seed.py --zip_code="${srcdir}/KEN_ALL.CSV" --jigyosyo="${srcdir}/JIGYOSYO.CSV" >> data/dictionary_oss/dictionary09.txt
  msg "Done."

  # Copy protobuf to be linked statically
  cp -rf "${srcdir}/protobuf-${_protobuf_ver}" protobuf/files

  msg "Starting make..."

  _targets="server/server.gyp:mozc_server gui/gui.gyp:mozc_tool unix/fcitx/fcitx.gyp:fcitx-mozc"
  _qmnames="qmake-qt4 qmake4 qmake"

  QTDIR=/usr python2 build_mozc.py gyp --channel_dev=0
  python2 build_mozc.py build_tools -c $_bldtype
  python2 build_mozc.py build -c $_bldtype $_targets

  # Extract license part of mozc
  head -n 28 server/mozc_server.cc > LICENSE
}

package() {
  cd "${srcdir}/${_pkgbase}-${pkgver}"
  install -D -m 755 out_linux/${_bldtype}/mozc_server "${pkgdir}/usr/lib/mozc/mozc_server"
  install    -m 755 out_linux/${_bldtype}/mozc_tool   "${pkgdir}/usr/lib/mozc/mozc_tool"
  install -d "${pkgdir}/usr/lib/mozc/documents/"
  install    -m 644 data/installer/*.html "${pkgdir}/usr/lib/mozc/documents/"

  install -D -m 644 LICENSE "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"

  for mofile in out_linux/${_bldtype}/obj/gen/unix/fcitx/po/*.mo
  do
    filename=`basename $mofile`
    lang=${filename/.mo/}
    install -D -m 644 "$mofile" "${pkgdir}/usr/share/locale/$lang/LC_MESSAGES/fcitx-mozc.mo"
  done

  install -D -m 755 out_linux/${_bldtype}/fcitx-mozc.so "${pkgdir}/usr/lib/fcitx/fcitx-mozc.so"
  install -D -m 644 unix/fcitx/fcitx-mozc.conf "${pkgdir}/usr/share/fcitx/addon/fcitx-mozc.conf"
  install -D -m 644 unix/fcitx/mozc.conf "${pkgdir}/usr/share/fcitx/inputmethod/mozc.conf"

  install -d "${pkgdir}/usr/share/fcitx/mozc/icon"
  install -m 644 data/images/product_icon_32bpp-128.png "${pkgdir}/usr/share/fcitx/mozc/icon/mozc.png"
  install -m 644 data/images/unix/ui-alpha_full.png "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-alpha_full.png"
  install -m 644 data/images/unix/ui-alpha_half.png "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-alpha_half.png"
  install -m 644 data/images/unix/ui-direct.png "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-direct.png"
  install -m 644 data/images/unix/ui-hiragana.png "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-hiragana.png"
  install -m 644 data/images/unix/ui-katakana_full.png "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-katakana_full.png"
  install -m 644 data/images/unix/ui-katakana_half.png "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-katakana_half.png"
  install -m 644 data/images/unix/ui-dictionary.png "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-dictionary.png"
  install -m 644 data/images/unix/ui-properties.png "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-properties.png"
  install -m 644 data/images/unix/ui-tool.png "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-tool.png"
}


md5sums=('e5246d17a81d2e942e9e8de0c3240c95'
         'e61df4b5754f3869ca504d269dc9641d'
         '59c5f7e9c734b40197454318f228859f'
         'ed436802019c9e1f40cc750eaf78f318'
         '3c947ef02d9bf3341192ba22916f3605')
