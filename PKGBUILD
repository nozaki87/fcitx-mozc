# Maintainer: Felix Yan <felixonmars@gmail.com>
# Contributor: ponsfoot <cabezon dot hashimoto at gmail dot com>

## Mozc compile option
_bldtype=Release

_zipcoderel=201306
_protobuf_ver=2.5.0

_pkgbase=mozc
pkgname=fcitx-mozc
pkgdesc="Fcitx Module of A Japanese Input Method for Chromium OS, Windows, Mac and Linux (the Open Source Edition of Google Japanese Input)"
pkgver=1.11.1502.102
_patchver=${pkgver}.1
pkgrel=1
arch=('i686' 'x86_64')
url="http://code.google.com/p/mozc/"
license=('custom')
depends=('qt4' 'fcitx' 'zinnia')
makedepends=('pkg-config' 'python2' 'gtest' 'curl' 'gtk2' 'mesa')
replaces=('mozc-fcitx')
conflicts=('mozc' 'mozc-server' 'mozc-utils-gui' 'mozc-fcitx')
source=(http://mozc.googlecode.com/files/mozc-${pkgver}.tar.bz2
        http://downloads.sourceforge.net/pnsft-aur/ken_all-${_zipcoderel}.zip
        http://downloads.sourceforge.net/pnsft-aur/jigyosyo-${_zipcoderel}.zip
        http://protobuf.googlecode.com/files/protobuf-${_protobuf_ver}.tar.bz2
        http://download.fcitx-im.org/fcitx-mozc/fcitx-mozc-${_patchver}.patch
	http://download.fcitx-im.org/fcitx-mozc/fcitx-mozc-icon.tar.gz
	#mozc_1.10.1390.102_remove_warn_as_error.patch
)

build() {
  cd "$srcdir"

  #source /etc/profile.d/qt4.sh

  # Update: Fix qt4 include path too
  # Fix compatibility with google-glog 0.3.3 (symbol conflict)
  #CFLAGS="${CFLAGS} -DFLAGS_log_dir=FLAGS_mozc_log_dir"
  #CXXFLAGS="${CXXFLAGS} -DFLAGS_log_dir=FLAGS_mozc_log_dir"
  CFLAGS="${CFLAGS} -I/usr/include/qt4 -fvisibility=hidden"
  CXXFLAGS="${CXXFLAGS} -I/usr/include/qt4 -fvisibility=hidden"

  cd "${srcdir}/${_pkgbase}-${pkgver}"
  
  # Apply fcitx patch
  rm unix/fcitx -rf
  patch -Np2 -i ${srcdir}/fcitx-mozc-${_patchver}.patch

  # Fix qt4 binary path
  sed -i 's|(qt_dir)/bin|(qt_dir)/lib/qt4/bin|' gui/*.gyp{,i}
  sed -i 's|(qt_dir_env)/bin|(qt_dir_env)/lib/qt4/bin|' gui/*.gyp{,i}

  # Patch for gcc>=4.8
  #patch -p0 -i "${srcdir}/mozc_1.10.1390.102_remove_warn_as_error.patch"

  # Adjust to use python2
  find . -name  \*.py        -type f -exec sed -i -e "1s|python.*$|python2|"  {} +
  find . -regex '.*\.gypi?$' -type f -exec sed -i -e "s|'python'|'python2'|g" {} +

  # Generate zip code seed
  msg "Generating zip code seed..."
  python2 dictionary/gen_zip_code_seed.py --zip_code="${srcdir}/KEN_ALL.CSV" --jigyosyo="${srcdir}/JIGYOSYO.CSV" >> data/dictionary_oss/dictionary09.txt
  msg "Done."

  # Copy protobuf to be linked statically
  cp -rf "${srcdir}/protobuf-${_protobuf_ver}" protobuf/files

  msg "Starting make..."

  _targets="server/server.gyp:mozc_server gui/gui.gyp:mozc_tool unix/fcitx/fcitx.gyp:fcitx-mozc"

  QTDIR=/usr GYP_DEFINES="document_dir=/usr/share/licenses/$pkgname" python2 build_mozc.py gyp --channel_dev=0
  python2 build_mozc.py build_tools -c $_bldtype
  python2 build_mozc.py build -c $_bldtype $_targets

  # Extract license part of mozc
  head -n 29 server/mozc_server.cc > LICENSE
}

package() {
  cd "${srcdir}/${_pkgbase}-${pkgver}"
  install -D -m 755 out_linux/${_bldtype}/mozc_server "${pkgdir}/usr/lib/mozc/mozc_server"
  install    -m 755 out_linux/${_bldtype}/mozc_tool   "${pkgdir}/usr/lib/mozc/mozc_tool"

  install -d "${pkgdir}/usr/share/licenses/$pkgname/"
  install -m 644 LICENSE data/installer/*.html "${pkgdir}/usr/share/licenses/${pkgname}/"

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
  install -m 644 "$srcdir/fcitx-mozc-icons/mozc.png" "${pkgdir}/usr/share/fcitx/mozc/icon/mozc.png"
  install -m 644 "$srcdir/fcitx-mozc-icons/mozc-alpha_full.png" "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-alpha_full.png"
  install -m 644 "$srcdir/fcitx-mozc-icons/mozc-alpha_half.png" "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-alpha_half.png"
  install -m 644 "$srcdir/fcitx-mozc-icons/mozc-direct.png" "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-direct.png"
  install -m 644 "$srcdir/fcitx-mozc-icons/mozc-hiragana.png" "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-hiragana.png"
  install -m 644 "$srcdir/fcitx-mozc-icons/mozc-katakana_full.png" "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-katakana_full.png"
  install -m 644 "$srcdir/fcitx-mozc-icons/mozc-katakana_half.png" "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-katakana_half.png"
  install -m 644 "$srcdir/fcitx-mozc-icons/mozc-dictionary.png" "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-dictionary.png"
  install -m 644 "$srcdir/fcitx-mozc-icons/mozc-properties.png" "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-properties.png"
  install -m 644 "$srcdir/fcitx-mozc-icons/mozc-tool.png" "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-tool.png"
}

sha512sums=('a88c98822d4eeefab2d50b760f3a161deecff5664837d4758652f61996dc7c78fe4ff278f93445810d905b2deb16d50aee02c4a8d64e19a47c7b1389ab03e746'
            '1e2ad1c68c2a1e85cfbc8d171e81cf33d9c1905bcb952eb0f2ff19cf114b569438a75ec88b7737af4446e33374f3f543bea8061a18d415f1e86dd3b7a5207148'
            '181adbee2d5090b3821d7472a35cd8e2b7d0ee1df4ba7e22acc9181b405c7707e67ba314b7d7dc2af006e42febb3ad1b55d244a2447d49649d775274c6785263'
            '5994b3669808b82fef5c860ecad36358c0767f84acac877e7bfcf722e59d972835a955714149bdd4158fbd1328a51d01397a563991d26475351ee72be48142ee'
            'd2935578c2eb76fa9d30f5eb689cbc04b95a38c94082216f752178f82ff63e409acfe22f345b9714db9434243c403f155fa574817e196bb86c4bc7dac1eb5b75'
            '5507c637e5a65c44ccf6e32118b6d16647ece865171b9a77dd3c78e6790fbd97e6b219e68d2e27750e22074eb536bccf8d553c295d939066b72994b86b2f251a')
