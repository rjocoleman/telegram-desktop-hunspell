# Maintainer: Sven-Hendrik Haase <sh@lutzhaase.com>
# Contributor: hexchain <i@hexchain.org>

# Thanks Nicholas Guriev <guriev-ns@ya.ru> for the patches!
# https://github.com/mymedia2/tdesktop

pkgname=telegram-desktop
pkgver=1.4.3
pkgrel=1
pkgdesc='Official Telegram Desktop client'
arch=('x86_64')
url="https://desktop.telegram.org/"
license=('GPL3')
depends=('ffmpeg' 'hicolor-icon-theme' 'minizip' 'openal' 'qt5-base' 'qt5-imageformats' 'openssl')
makedepends=('cmake' 'git' 'gyp' 'range-v3' 'python' 'libappindicator-gtk3')
optdepends=('libnotify: desktop notifications')
source=(
    "tdesktop::git+https://github.com/telegramdesktop/tdesktop.git#tag=v$pkgver"
    "libtgvoip::git+https://github.com/telegramdesktop/libtgvoip"
    "variant::git+https://github.com/mapbox/variant"
    "Catch::git+https://github.com/philsquared/Catch"
    "GSL::git+https://github.com/Microsoft/GSL.git"
    "crl::git+https://github.com/telegramdesktop/crl.git"
    "xxHash::git+https://github.com/Cyan4973/xxHash.git"
    "tg.protocol"
    "CMakeLists.inj"
    "tdesktop.patch"
    "no-gtk2.patch"
    "libtgvoip.patch"
)
sha512sums=('SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'b87414ceaae19185a8a5749cea1f6d9f3fc3c69b8dd729e3db8790cde00b987c3c827cd30baf0eac579d1884e34aa2f37bb90778c3c0bc9ca211d75a82891b9d'
            'b20674f61ff6378749d1f59a6a0da194d33ccc786bd783f6ed62027924a3a8a8d27c9763bf376480432d6536896b0c7eeb8c495c5b8cefff7cf5fe84da50947e'
            '47c0d1e00401d7899b2ce710c06ceac6caffddbe1a4c85bc407918b43051cba292f6e6131cc0f390a66520e92f0ffa3761f8d973a25986dfe21c54f113062c33'
            '7a37e0ca582145a56a411585aec0bc94889dc18a80cc038d2efa237e19eebf8b67d56825e068be88f7566b08316ce068d7f20c25729caa33d0e9d6c370325025'
            'd60694dc701aa985b0e82a12c9732b945082470441c687b33167a94f94efcf253baf43bb7280ec160ba338485ee5c62de138e4804cae05f27cc5cf4298166d39')

prepare() {
    cd "$srcdir/tdesktop"
    git submodule init
    git config submodule.Telegram/ThirdParty/libtgvoip.url "$srcdir/libtgvoip"
    git config submodule.Telegram/ThirdParty/variant.url "$srcdir/variant"
    git config submodule.Telegram/ThirdParty/GSL.url "$srcdir/GSL"
    git config submodule.Telegram/ThirdParty/Catch.url "$srcdir/Catch"
    git config submodule.Telegram/ThirdParty/crl.url "$srcdir/crl"
    git config submodule.Telegram/ThirdParty/xxHash.url "$srcdir/xxHash"
    git submodule update

    patch -Np1 -i "$srcdir/tdesktop.patch"
    patch -Np1 -i "$srcdir/no-gtk2.patch"

    cd "Telegram/ThirdParty/libtgvoip"
    patch -Np1 -i "$srcdir/libtgvoip.patch"
}

build() {
    cd "$srcdir/tdesktop"
    export LANG=en_US.UTF-8
    export GYP_DEFINES="TDESKTOP_DISABLE_CRASH_REPORTS,TDESKTOP_DISABLE_AUTOUPDATE,TDESKTOP_DISABLE_REGISTER_CUSTOM_SCHEME"
    export EXTRA_FLAGS="-Winvalid-pch"
    export CPPFLAGS="$CPPFLAGS $EXTRA_FLAGS"
    export CXXFLAGS="$CXXFLAGS $EXTRA_FLAGS"
    gyp \
        -Dbuild_defines=${GYP_DEFINES} \
        -Gconfig=Release \
        --depth=Telegram/gyp --generator-output=../.. -Goutput_dir=out Telegram/gyp/Telegram.gyp --format=cmake
    NUM=$((`wc -l < out/Release/CMakeLists.txt` - 2))
    sed -i "$NUM r ../CMakeLists.inj" out/Release/CMakeLists.txt
    cd "$srcdir/tdesktop/out/Release"
    cmake . -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_BUILD_TYPE=Release
    make
}

package() {
    install -dm755 "$pkgdir/usr/bin"
    install -m755 "$srcdir/tdesktop/out/Release/Telegram" "$pkgdir/usr/bin/telegram-desktop"

    install -d "$pkgdir/usr/share/applications"
    install -m644 "$srcdir/tdesktop/lib/xdg/telegramdesktop.desktop" "$pkgdir/usr/share/applications/telegramdesktop.desktop"

    install -d "$pkgdir/usr/share/kservices5"
    install -m644 "$srcdir/tg.protocol" "$pkgdir/usr/share/kservices5/tg.protocol"

    local icon_size icon_dir
    for icon_size in 16 32 48 64 128 256 512; do
        icon_dir="$pkgdir/usr/share/icons/hicolor/${icon_size}x${icon_size}/apps"

        install -d "$icon_dir"
        install -m644 "$srcdir/tdesktop/Telegram/Resources/art/icon${icon_size}.png" "$icon_dir/telegram.png"
    done
}
