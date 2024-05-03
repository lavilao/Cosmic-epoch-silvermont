# Maintainer: soloturn <soloturn@gmail.com>
# Co-Maintainer: Mark Wagie <mark dot wagie at proton dot me>

pkgname=cosmic-epoch-git
pkgver=r123.2eadc4e
pkgrel=1
pkgdesc="Cosmic desktop environment from System76's Pop!_OS written in Rust utilizing Iced inspired by GNOME"
arch=('x86_64' 'aarch64')
url="https://github.com/pop-os/cosmic-epoch"
license=('GPL-3.0-or-later AND MPL-2.0 AND CC-BY-SA-4.0')
depends=(
  'accountsservice'
  'archlinux-appstream-data'
  'cage'
  'fontconfig'
  'geoclue'
  'greetd'
  'gtk4'
  'iso-codes'
  'libinput'
  'libglvnd'
  'libpipewire'
  'libpulse'
  'libseat.so'
  'libxkbcommon'
  'org.freedesktop.secrets'
  'pop-icon-theme-git'
  'pop-launcher-git'
  'systemd-libs'
  'wayland'
  'xdg-desktop-portal'
  'xdg-utils'
)
makedepends=(
  'cargo'
  'clang'
  'flatpak'
  'git'
  'intltool'
  'just'
  'mold'
  'nasm'
)
optdepends=(
  'flatpak: Flatpak packages support for COSMIC Store'
  'otf-fira-mono: Recommended Mono font'
  'otf-fira-sans: Recommended Sans font'
  'packagekit: package manager integration module for COSMIC Store'
)

_submodules=(
  cosmic-applets
  cosmic-applibrary
  cosmic-bg
  cosmic-comp
  cosmic-edit
  cosmic-files
  cosmic-greeter
  cosmic-launcher
  cosmic-notifications
  cosmic-osd
  cosmic-panel
  cosmic-randr
  cosmic-screenshot
  cosmic-session
  cosmic-settings
  cosmic-settings-daemon
  cosmic-term
  cosmic-workspaces-epoch
  xdg-desktop-portal-cosmic
)

provides=(
  'cosmic-epoch'
  'cosmic-icons'
  'cosmic-store'
  'cosmic-workspaces'
  "${_submodules[@]}"
)
conflicts=(
  'cosmic-epoch'
  'cosmic-icons'
  'cosmic-store'
  'cosmic-workspaces'
  "${_submodules[@]}"
)
backup=('etc/cosmic-comp/config.ron')
options=('!lto')
source=(
  'git+https://github.com/pop-os/cosmic-epoch.git'
  'git+https://github.com/pop-os/cosmic-applets.git'
  'git+https://github.com/pop-os/cosmic-applibrary.git'
  'git+https://github.com/pop-os/cosmic-bg.git'
  'git+https://github.com/pop-os/cosmic-comp.git'
  'git+https://github.com/pop-os/cosmic-edit.git'
  'git+https://github.com/pop-os/cosmic-files.git'
  'git+https://github.com/pop-os/cosmic-greeter.git'
  'git+https://github.com/pop-os/cosmic-icons.git'
  'git+https://github.com/pop-os/cosmic-launcher.git'
  'git+https://github.com/pop-os/cosmic-notifications.git'
  'git+https://github.com/pop-os/cosmic-osd.git'
  'git+https://github.com/pop-os/cosmic-panel.git'
  'git+https://github.com/pop-os/cosmic-randr.git'
  'git+https://github.com/pop-os/cosmic-screenshot.git'
  'git+https://github.com/pop-os/cosmic-session.git'
  'git+https://github.com/pop-os/cosmic-settings.git'
  'git+https://github.com/pop-os/cosmic-settings-daemon.git'
  'git+https://github.com/pop-os/cosmic-store.git'
  'git+https://github.com/pop-os/cosmic-term.git'
  'git+https://github.com/pop-os/cosmic-workspaces-epoch.git'
  'git+https://github.com/pop-os/xdg-desktop-portal-cosmic.git'
  'git+https://github.com/jackpot51/appstream.git'
  'justfile.diff'
)
sha256sums=('SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            '5a642677504184fd2b4957aa3ce6596d426083dd17f5bba2217e882975d599d9')

pkgver() {
  cd cosmic-epoch
  printf "r%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
}

prepare() {
  cd cosmic-epoch
  export CARGO_HOME="$srcdir/cargo-home"
  export RUSTUP_TOOLCHAIN=stable

  for submodule in "${_submodules[@]}"; do
    git submodule init
    git config submodule."${submodule#*::}".url "$srcdir/${submodule%::*}"
    git config submodule.cosmic-icons.url "$srcdir/cosmic-icons"
    git -c protocol.file.allow=always submodule update

    git config submodule.cosmic-store.url "$srcdir/cosmic-store"
    pushd cosmic-store
    git config submodule.appstream.url "$srcdir/appstream"
    git -c protocol.file.allow=always submodule update
    popd

    pushd "${submodule#*::}"
    cargo fetch --target "$CARCH-unknown-linux-gnu"
    popd

    pushd cosmic-store
    cargo fetch --target "$CARCH-unknown-linux-gnu"
    popd

  done

  # Use mold linker instead of lld
  sed -i 's/lld/mold/g' cosmic-notifications/justfile

  # libexec > lib
  # see discussion: https://github.com/pop-os/cosmic-epoch/issues/87
  sed -i 's|libexecdir = $(prefix)/libexec|libexecdir = $(libdir)|g' \
    xdg-desktop-portal-cosmic/Makefile
  sed -i 's|libexec|lib|g' cosmic-session/{Justfile,src/main.rs} \
    cosmic-settings-daemon/{Makefile,src/main.rs}
  sed -i 's|libexec|lib/polkit-1|g' cosmic-osd/Makefile \
    cosmic-osd/src/subscriptions/polkit_agent_helper.rs

  # Fix justfile
  patch -p1 justfile < ../justfile.diff
}

build() {
  cd cosmic-epoch
#  CFLAGS+=" -ffat-lto-objects"  ## cosmic-edit fails
  export CARGO_HOME="$srcdir/cargo-home"
  export RUSTUP_TOOLCHAIN=stable
  # note, consider rust build time optimisations: 
  # https://matklad.github.io/2021/09/04/fast-rust-builds.html, 
  # later. for now, ignore warnings, and build with lower priority 
  # to not block user installing this pkg. to speed up build, use "mold" linker, see 
  # https://stackoverflow.com/questions/67511990/how-to-use-the-mold-linker-with-cargo
  RUSTFLAGS="-A warnings -C link-arg=-fuse-ld=mold"
  nice just build
}

package() {
  cd cosmic-epoch
  just rootdir="$pkgdir" install

  # Keybinding config
  # https://github.com/pop-os/cosmic-epoch#cosmic-comp
  install -Dm644 cosmic-comp/config.ron -t "$pkgdir/etc/cosmic-comp/"
}
