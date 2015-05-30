# Maintainer: Felix Yan <felixonmars@archlinux.org>
# Contributor: ponsfoot <cabezon dot hashimoto at gmail dot com>
# Contributor: UTUMI Hirosi <utuhiro78 at yahoo dot co dot jp>

## Mozc compile option
_bldtype=Release

_zipcode_rel=201504
_protobuf_rev=172019c40bf548908ab09bfd276074c929d48415
_gyp_rev=cdf037c1edc0ba3b5d25f8e3973661efe00980cc
_jsoncpp_rev=11086dd6a7eba04289944367ca82cea71299ed70
_japanese_usage_dictionary_rev=10
_mozc_rev=2ceaee96e4acabb39c45f071fc39df485870da96

_pkgbase=mozc
pkgname=fcitx-mozc
pkgdesc="Fcitx Module of A Japanese Input Method for Chromium OS, Windows, Mac and Linux (the Open Source Edition of Google Japanese Input)"
pkgver=2.17.2097.102
_fcitx_patchver=2.16.2037.102.2
pkgrel=1
arch=('i686' 'x86_64')
url="https://github.com/google/mozc"
license=('custom')
depends=('qt4' 'fcitx' 'zinnia')
makedepends=('pkg-config' 'python2' 'curl' 'gtk2' 'mesa' 'subversion' 'ninja' 'git' 'clang')
replaces=('mozc-fcitx')
conflicts=('mozc' 'mozc-server' 'mozc-utils-gui' 'mozc-fcitx')
source=(mozc::git+https://github.com/google/mozc.git#commit=${_mozc_rev}
        jsoncpp::git+https://github.com/open-source-parsers/jsoncpp.git#commit=${_jsoncpp_rev}
        japanese_usage_dictionary::svn+http://japanese-usage-dictionary.googlecode.com/svn/trunk#revision=$_japanese_usage_dictionary_rev
        gyp::git+https://chromium.googlesource.com/external/gyp#commit=${_gyp_rev}
        protobuf::git+https://github.com/google/protobuf.git#commit=${_protobuf_rev}
        http://downloads.sourceforge.net/pnsft-aur/x-ken-all-${_zipcode_rel}.zip
        http://downloads.sourceforge.net/pnsft-aur/jigyosyo-${_zipcode_rel}.zip
        http://download.fcitx-im.org/fcitx-mozc/fcitx-mozc-${_fcitx_patchver}.patch
        http://download.fcitx-im.org/fcitx-mozc/fcitx-mozc-icon.tar.gz)
sha512sums=('SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            '60c053fd3ffec28e0e2eca33834e4cc212127e032b30d0638dcb7ce3faa45c7c87ceae9f98eb03f0ec0e3d1429d2c105f4590e30b39e0818cfcf50646584c459'
            'fb1ad9d279e34a1e55b6dc182ad503abd250d4773ce488a12c4b07bf67eccda65ae7d83b41db6489afd867e303468448dc13d8797ed3bbc038de60ff744f0b9f'
            '22b885859588bb8e0efd354d153da461a654203729c723156a419bf33fae473e3f7165964aa3cb3b5c969f97c2727f9d87b0d587330e4eeab67f07d4458542a3'
            '5507c637e5a65c44ccf6e32118b6d16647ece865171b9a77dd3c78e6790fbd97e6b219e68d2e27750e22074eb536bccf8d553c295d939066b72994b86b2f251a')

prepare() {
  cd mozc/src

  # Apply fcitx patch
  rm unix/fcitx -rf
  patch -Np2 -i "$srcdir/fcitx-mozc-${_fcitx_patchver}.patch"

  # Fix qt4 binary path
  sed -i 's|(qt_dir)/bin|(qt_dir)/lib/qt4/bin|' gui/*.gyp{,i}
  sed -i 's|(qt_dir_env)/bin|(qt_dir_env)/lib/qt4/bin|' gui/*.gyp{,i}

  # Adjust to use python2
  find . -name  \*.py        -type f -exec sed -i -e "1s|python.*$|python2|"  {} +
  find . -regex '.*\.gypi?$' -type f -exec sed -i -e "s|'python'|'python2'|g" {} +

  # Generate zip code seed
  msg "Generating zip code seed..."
  python2 dictionary/gen_zip_code_seed.py --zip_code="${srcdir}/x-ken-all.csv" --jigyosyo="${srcdir}/JIGYOSYO.CSV" >> data/dictionary_oss/dictionary09.txt
  msg "Done."

  # Copy third party deps
  cd "$srcdir"
  for dep in jsoncpp gyp protobuf japanese_usage_dictionary
  do
    cp -a $dep mozc/src/third_party/
  done
}

build() {
  # Update: Fix qt4 include path too
  # Fix compatibility with google-glog 0.3.3 (symbol conflict)
  CFLAGS="${CFLAGS} -I/usr/include/qt4 -fvisibility=hidden"
  CXXFLAGS="${CXXFLAGS} -I/usr/include/qt4 -fvisibility=hidden"

  cd mozc/src

  _targets="server/server.gyp:mozc_server gui/gui.gyp:mozc_tool unix/fcitx/fcitx.gyp:fcitx-mozc"

  QTDIR=/usr GYP_DEFINES="document_dir=/usr/share/licenses/$pkgname" python2 build_mozc.py gyp
  python2 build_mozc.py build -c $_bldtype -j 8 $_targets

  # Extract license part of mozc
  head -n 29 server/mozc_server.cc > LICENSE
}

package() {
  cd mozc/src
  install -D -m 755 out_linux/${_bldtype}/mozc_server "${pkgdir}/usr/lib/mozc/mozc_server"
  install    -m 755 out_linux/${_bldtype}/mozc_tool   "${pkgdir}/usr/lib/mozc/mozc_tool"

  install -d "${pkgdir}/usr/share/licenses/$pkgname/"
  install -m 644 LICENSE data/installer/*.html "${pkgdir}/usr/share/licenses/${pkgname}/"

  for mofile in out_linux/${_bldtype}/gen/unix/fcitx/po/*.mo
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
