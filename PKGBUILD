# Maintainer: Felix Yan <felixonmars@archlinux.org>
# Contributor: ponsfoot <cabezon dot hashimoto at gmail dot com>
# Contributor: UTUMI Hirosi <utuhiro78 at yahoo dot co dot jp>

## Mozc compile option
_bldtype=Release

_mozc_rev=280e38fe3d9db4df52f0713acf2ca65898cd697a
_japanese_usage_dictionary_rev=e5b3425575734c323e1d947009dd74709437b684
_gyp_rev=920ee58c3d3109dea3cd37d88054014891a93db7
_protobuf_rev=a428e42072765993ff674fda72863c9f1aa2d268
_zipcode_rel=201611

_pkgbase=mozc
pkgname=fcitx-mozc
pkgdesc="Fcitx Module of A Japanese Input Method for Chromium OS, Windows, Mac and Linux (the Open Source Edition of Google Japanese Input)"
pkgver=2.20.2673.102
_fcitx_patchver=2.18.2612.102.1
pkgrel=1
arch=('i686' 'x86_64')
url="https://github.com/google/mozc"
license=('custom')
depends=('qt5-base' 'fcitx' 'zinnia')
makedepends=('pkg-config' 'python2' 'curl' 'gtk2' 'mesa' 'subversion' 'ninja' 'git' 'clang')
replaces=('mozc-fcitx')
conflicts=('mozc' 'mozc-server' 'mozc-utils-gui' 'mozc-fcitx')
source=(git+https://github.com/google/mozc.git#commit=${_mozc_rev}
        japanese_usage_dictionary::git+https://github.com/hiroyuki-komatsu/japanese-usage-dictionary.git#commit=${_japanese_usage_dictionary_rev}
        git+https://chromium.googlesource.com/external/gyp#commit=${_gyp_rev}
        git+https://github.com/google/protobuf.git#commit=${_protobuf_rev}
        http://downloads.sourceforge.net/pnsft-aur/x-ken-all-${_zipcode_rel}.zip
        http://downloads.sourceforge.net/pnsft-aur/jigyosyo-${_zipcode_rel}.zip
        http://download.fcitx-im.org/fcitx-mozc/fcitx-mozc-${_fcitx_patchver}.patch
        http://download.fcitx-im.org/fcitx-mozc/fcitx-mozc-icon.tar.gz)
sha512sums=('SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'c48a4ac3237a367672009e67ccc34416cbbd5373f8ce7e47d28e5be85dddbb216d3eb27d2439cde4fdcf826df92ad13694e113761cb927def77944f14757aab1'
            'a6c328ebf4805a38ed17f782d70179d37072a0d4c6dd50fef0aeefff0e073acd606a006c261c6a79d381038e3928533f76104362d5bd443d93794ecffe45d923'
            '725c464727f230707b90975705ef1565f1178f9144e66c5a688696a4ef520a6fec1d11f7aa68ee05f82aa9417060b1cccf49a155e30734871a525eaa4c76addb'
            '5507c637e5a65c44ccf6e32118b6d16647ece865171b9a77dd3c78e6790fbd97e6b219e68d2e27750e22074eb536bccf8d553c295d939066b72994b86b2f251a')

prepare() {
  cd mozc/src

  # Apply fcitx patch
  rm unix/fcitx -rf
  patch -Np2 -i "$srcdir/fcitx-mozc-${_fcitx_patchver}.patch"

  # Adjust to use python2
  find . -name  \*.py        -type f -exec sed -i -e "1s|python.*$|python2|"  {} +
  find . -regex '.*\.gypi?$' -type f -exec sed -i -e "s|'python'|'python2'|g" {} +

  # Generate zip code seed
  msg "Generating zip code seed..."
  PYTHONPATH="$PWD:$PYTHONPATH" python2 dictionary/gen_zip_code_seed.py --zip_code="${srcdir}/x-ken-all.csv" --jigyosyo="${srcdir}/JIGYOSYO.CSV" >> data/dictionary_oss/dictionary09.txt
  msg "Done."

  # Copy third party deps
  cd "$srcdir"
  for dep in gyp protobuf japanese_usage_dictionary
  do
    cp -a $dep mozc/src/third_party/
  done
}

build() {
  # Fix compatibility with google-glog 0.3.3 (symbol conflict)
  CFLAGS="${CFLAGS} -fvisibility=hidden"
  CXXFLAGS="${CXXFLAGS} -fvisibility=hidden"

  cd mozc/src

  _targets="server/server.gyp:mozc_server gui/gui.gyp:mozc_tool unix/fcitx/fcitx.gyp:fcitx-mozc"

  QTDIR=/usr GYP_DEFINES="document_dir=/usr/share/licenses/$pkgname use_libzinnia=1" python2 build_mozc.py gyp
  python2 build_mozc.py build -c $_bldtype $_targets

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
