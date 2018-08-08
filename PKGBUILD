# Maintainer: André Silva <emulatorman@hyperbola.info>
# Maintainer: Luke R. <g4jc@hyperbola.info>
# Contributor: Márcio Silva <coadde@hyperbola.info>

# Based on iceweasel-esr and basilisk packages

_pgo=false

_pkgname=UXP
pkgname=iceweasel-uxp
_pkgver=2018.07.18
_appver=1.9
pkgver=52.9.${_pkgver//./}
pkgrel=2
pkgdesc="A new generation of Iceweasel, an XUL-based standalone web browser on the Unified XUL Platform (UXP)."
arch=(i686 x86_64)
license=(MPL GPL LGPL)
depends=(alsa-lib dbus-glib ffmpeg gtk2 gtk3 hunspell icu=59.1 libevent libvpx libxt mime-types mozilla-common nss sqlite startup-notification ttf-font)
makedepends=(autoconf-legacy diffutils gconf imake inetutils libpulse mesa python2 unzip yasm zip)
options=(!emptydirs !makeflags !strip debug)
if $_pgo; then
  makedepends+=(xorg-server-xvfb)
  options+=(!ccache)
fi
optdepends=('networkmanager: Location detection via available WiFi networks'
            'libnotify: Notification integration'
            'speech-dispatcher: Text-to-Speech')
url="https://wiki.hyperbola.info/$pkgname"
replaces=('basilisk' 'firefox' 'firefox-esr' 'icecat' 'iceweasel' 'iceweasel-esr')
conflicts=('basilisk' 'firefox' 'firefox-esr' 'icecat' 'iceweasel' 'iceweasel-esr')
provides=('basilisk' 'firefox' 'firefox-esr' 'icecat' 'iceweasel' 'iceweasel-esr')
source=("$_pkgname-$_pkgver.tar.gz::https://github.com/MoonchildProductions/$_pkgname/archive/v$_pkgver.tar.gz"
        "https://repo.hyperbola.info:50000/other/$pkgname/$pkgname-$_appver.tar.gz"{,.asc}
        mozconfig
        $pkgname.desktop
        $pkgname-install-dir.patch
        vendor.js
        # Application patches
        0001-iceweasel-application-specific-overrides.patch)
sha512sums=('c6db4c5868bbea64f25dc658fdca85ab8a3070dd74b22e141aab33b40c518e9c1a2e980441e43b87fd26708563ec4d5e3a29d3baf0b28cc4dd24a1af490c56a6'
            '800a36bed816a791e2abb83665727f41586b02c912a54fe4082bbaddb53c9f15a019419c6c12f51b84777b76709e0711cc8a6703299297cb2c365dd24daceee0'
            'SKIP'
            '3d7880e9e6499422bbec445c76fe82e24fbf41b295261dd8d2f14676133eddc450ba0db786667b36198aa13bbfb0baed5164c9265523ab4d56d18c7db8ff5a88'
            '42f0003895200da7a311226d6a14245c667a1ed1c643b7a2a0f2676d2b30e0881c6198909eead14777584bb6f81733205503e6a15c78ed581d39d5bfb6b95ec4'
            '30d607ed2e6c0da75930a87191c4a703f47ca918bb33748bd5f62e3f7fd6846605b65aa1f3825da48bb9828188ef784fd3c953a0673c60a28dfa8a74a13195c1'
            'dc1456e2c3dca3c1b3f022c0fe554bd0d46a5c5c4e7ecb1b8f39bfded57bf2e10c9fcfe1df049e1bb2e63dd765f41bc46f2939c41d2325ad1308b4a5d55e5a18'
            'cb088bad8703be7a1d3ad985f269c794819f3208e2b81157c25ba21e8c66222c0eb0c91e831d29f7822837ca3a3d84704c779ea254bc184cba5d78e1305f6854')
validpgpkeys=('B00033B86FDA59BA7D75B89BB88AF90BF64827C5') # Luke R.

prepare() {
  # Move our application into the Unified XUL Platform
  cd "$srcdir/$_pkgname-$_pkgver"
  mv "$srcdir/$pkgname-$_appver" application/$pkgname
  
  # Apply Iceweasel-UXP application specific overrides to UXP
  patch -p1 -i "$srcdir/0001-iceweasel-application-specific-overrides.patch"

  # Install to /usr/lib/iceweasel-uxp
  patch -p1 -i "$srcdir/$pkgname-install-dir.patch"

  # Adapt Iceweasel-UXP version to $pkgver
  sed -i "s|52.9.0_YYYYMMDD|52.9.YYYYMMDD|
          s|MOZ_APP_VERSION[=]52[.]9[.]0_.*|MOZ_APP_VERSION=52.9.${_pkgver//./}|
          s|MOZ_APP_VERSION_DISPLAY[=].*date.*|MOZ_APP_VERSION_DISPLAY=${_pkgver//./}|
         " application/$pkgname/confvars.sh

  # Add missing versionField.textContent in aboutDialog.js
  sed -i 's|let version [=] Services[.]appinfo[.]version[;]|let version = Services.appinfo.version;\n  versionField.textContent = version;|' application/$pkgname/base/content/aboutDialog.js

  # Load our build config
  cp "$srcdir/mozconfig" .mozconfig

  mkdir "$srcdir/path"
  ln -s /usr/bin/python2 "$srcdir/path/python"
}

build() {
  cd "$srcdir/$_pkgname-$_pkgver"

  # _FORTIFY_SOURCE causes configure failures
  CPPFLAGS+=" -O2"

  export PATH="$srcdir/path:$PATH"

  if $_pgo; then
    # Do PGO
    xvfb-run -a -n 95 -s "-extension GLX -screen 0 1280x1024x24" \
      make -f client.mk build MOZ_PGO=1
  else
    make -f client.mk build
  fi
}

package() {
  cd "$srcdir/$_pkgname-$_pkgver"

  make -f client.mk DESTDIR="$pkgdir" INSTALL_SDK= install

  install -Dm644 "$srcdir/vendor.js" "$pkgdir/usr/lib/$pkgname/browser/defaults/preferences/vendor.js"

  for i in 16 32 48; do
    install -Dm644 application/$pkgname/branding/${pkgname%-*}/default$i.png \
      "$pkgdir/usr/share/icons/hicolor/${i}x${i}/apps/$pkgname.png"
  done
  install -Dm644 application/$pkgname/branding/${pkgname%-*}/content/icon64.png \
    "$pkgdir/usr/share/icons/hicolor/64x64/apps/$pkgname.png"
  install -Dm644 application/$pkgname/branding/${pkgname%-*}/content/about-logo.png \
    "$pkgdir/usr/share/icons/hicolor/192x192/apps/$pkgname.png"
  install -Dm644 application/$pkgname/branding/${pkgname%-*}/content/about-logo@2x.png \
    "$pkgdir/usr/share/icons/hicolor/384x384/apps/$pkgname.png"

  install -Dm644 "$srcdir/$pkgname.desktop" \
    "$pkgdir/usr/share/applications/$pkgname.desktop"

  # Use system-provided dictionaries
  rm -rf "$pkgdir/usr/lib/$pkgname/"{dictionaries,hyphenation}
  ln -s /usr/share/hunspell "$pkgdir/usr/lib/$pkgname/dictionaries"
  ln -s /usr/share/hyphen   "$pkgdir/usr/lib/$pkgname/hyphenation"

  # Replace duplicate binary with symlink
  # https://bugzilla.mozilla.org/show_bug.cgi?id=658850
  ln -sf $pkgname "$pkgdir/usr/lib/$pkgname/$pkgname-bin"
  rm "$pkgdir/usr/bin/$pkgname"

  ln -srf "$pkgdir/usr/lib/$pkgname/$pkgname" \
    "$pkgdir/usr/bin/$pkgname"
}