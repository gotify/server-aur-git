# Maintainer: jmattheis <contact AT jmattheis DOT de>
# Contributor: ml <ml@visu.li>
pkgname=gotify-server
pkgver=2.6.3
pkgrel=1
pkgdesc='A simple server for sending and receiving messages in real-time per WebSocket.'
arch=('x86_64' 'i686' 'aarch64' 'armv7h')
url='https://gotify.net/'
license=('MIT')
depends=('glibc')
makedepends=('git' 'go' 'gzip' 'yarn')
backup=('etc/gotify/config.yml')
install=gotify-server.install
options=(emptydirs)
source=(
  "$pkgname-$pkgver.tar.gz::https://github.com/gotify/server/archive/v${pkgver}.tar.gz"
  'sysusers.d'
  'gotify-server.service'
  'config.patch'
)
sha256sums=('a2ab6d25afa63875e43d1f837dd0dcc36bd4ab154f2b093feff81987ade9b742'
            '39fc913f205bbb102ba42ce3d419f2feb0f9143f14ccffd242b3cd5f51a8c0de'
            '8c3832004ed6f46e01ab69c993773da19b50a45862354165ed065bf3d2147b92'
            'c251cc61b80968f606df9f9840519456fa5ba5f35ced44f73fda49059ed650b6')

prepare() {
  patch -N -p1 -d "server-$pkgver" <config.patch
}

build() {
  local _commit=$(zcat "${pkgname}-${pkgver}.tar.gz" | git get-tar-commit-id)
  cd "server-$pkgver"
  (
    cd ui

    if node --help | grep -q -- "--openssl-legacy-provider"; then
      export NODE_OPTIONS=--openssl-legacy-provider
    fi

    yarn --frozen-lockfile
    NODE_ENV=production yarn --frozen-lockfile build
  )
  export CGO_CFLAGS="$CFLAGS"
  export CGO_LDFLAGS="$LDFLAGS"
  export CGO_CPPFLAGS="$CPPFLAGS"
  export CGO_CXXFLAGS="$CXXFLAGS"
  export GOFLAGS='-buildmode=pie -modcacherw'
  go build \
    -o "$pkgname" \
    -trimpath \
    -ldflags "-X 'main.Version=${pkgver}' \
              -X 'main.Commit=${_commit}' \
              -X 'main.BuildDate=$(date -u -d "@${SOURCE_DATE_EPOCH}" +%F-%T)' \
              -X 'main.Mode=prod'"
}

package() {
  install -Dm644 sysusers.d "$pkgdir/usr/lib/sysusers.d/gotify.conf"
  install -Dm644 gotify-server.service "$pkgdir/usr/lib/systemd/system/gotify-server.service"
  # required for StandardOutput in gotify-server.service
  install -dm755 "$pkgdir/var/log/gotify"

  cd "server-$pkgver"
  install -Dm755 "$pkgname" "$pkgdir/usr/bin/$pkgname"
  install -Dm644 config.example.yml "$pkgdir/etc/gotify/config.yml"
  install -Dm644 LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
