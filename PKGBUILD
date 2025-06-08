# Maintainer: David Cohen <dacohen@pm.me>
# Contributor: Jan Alexander Steffens (heftig) <heftig@archlinux.org>

# This PKGBUILD script and the config file are based on official Arch's linux package
# Refer to config file to get the exact versions it's based on. Any changes to the config
# will be on config.extra file.

pkgbase=linux-edac-git
pkgver=6.15.r13627.119b1e61a769
pkgrel=1
pkgdesc="Linus Torvalds' Mainline Linux"
url="https://www.kernel.org"
arch=(x86_64)
license=(GPL-2.0-only)
_userconfig="/etc/${pkgbase}/config"
_userremote="/etc/${pkgbase}/remote"
_userpatches="/etc/${pkgbase}/patches/patches"
backup=(
"${_userconfig##/}"
"${_userremote##/}"
"${_userpatches##/}"
)
makedepends=(
  bc
  bison
  cpio
  flex
  gettext
  git
  libelf
  pahole
  perl
  python
  rust
  rust-bindgen
  rust-src
  tar
  xz
)
options=(
  !debug
  !strip
)
_srcname=linux-torvalds
source=(
  "$_srcname::git+https://kernel.googlesource.com/pub/scm/linux/kernel/git/torvalds/linux"
  config         # the main kernel config file
  config.extra   # additional configs
  config.user    # user custom config
  remote         # custom remote config
  patches        # user patches config
)
validpgpkeys=(
  'ABAF11C65A2970B130ABE3C479BE3E4300411886'  # Linus Torvalds
  '647F28654894E3BD457199BE38DBBDC86092693E'  # Greg Kroah-Hartman
)
b2sums=('SKIP'
        'a2644043800a384d40677ed957c5c9cbd70600b20f700fa5c3e84684574da7496f8a17b10f2984a15a918ef60211db08996cb189daf3a98cbb8bb5cbad0a8512'
        '249bec61fed688345a0f41245af9e8e189af3149e66a3c0dcc8e833151428232a701a35ed760ef93ceb5f25d9378c44f903f380a7051a65fb9a203c6fb51ebcd'
        'e162f38de7a7dc870dbd9a91b5e5b05460090a259dec4ff07d9f71155521aea907e10420271ad404d1959899e9e1819ad31547401d06c2ed77d04c690c03e3e0'
        'a50d0b4842bb6ad4d522beb71aa29a5b130ad43ea09efcada3aee472346d2f28ed30af3c91139442db3ae885d32a903697641551d5cb2a7fc1371f0b4249815a'
        '04709de86b8803edfc2d12e5fe80d96c172eb817c8cf357990821003b344f54e0dd6f35a0694a2e8012d7d9cbac082ffeaeaaf036301b084fc69361034a91c7f')

export KBUILD_BUILD_HOST=archlinux
export KBUILD_BUILD_USER=$pkgbase
export KBUILD_BUILD_TIMESTAMP="$(date -Ru${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH})"

pkgver() {
  cd "$srcdir/$_srcname"
  if [[ -n "$REMOTE_URL" ]]; then
    # if $REMOTE_URL was created, it's safe to use $REMOTE
    printf "%s" "$(git describe --dirty=-patched --long | sed 's/\([^-]*-\)g/r\1/;s/-/./g' | sed 's/^v//').${REMOTE//[\/-]/.}"
  else
    printf "%s" "$(git describe --dirty=-patched --long | sed 's/\([^-]*-\)g/r\1/;s/-/./g' | sed 's/^v//')"
  fi
}

prepare() {
  cd $_srcname

  if [[ -z ${IGNORE_USER_CUSTOM} ]]; then
    [[ -f "$_userremote" ]] && source "$_userremote" || source "../${_userremote##*/}"
  fi
  if [[ -n "$REMOTE" && -n "$COMMIT" ]]; then
    REMOTE_PREFIX=${source[0]##${_srcname}::git+}
    REMOTE_PREFIX=${REMOTE_PREFIX%%torvalds/linux}
    REMOTE_URL=${REMOTE_PREFIX}${REMOTE}
    echo
    echo "================================================================================"
    echo "Build script detected custom variables \$REMOTE and \$COMMIT are being used:"
    echo "REMOTE_PREFIX: $REMOTE_PREFIX"
    echo "REMOTE_TIP   : $REMOTE"
    echo "COMMIT       : $COMMIT"
    echo "================================================================================"
    echo
    echo "Fetching ${REMOTE_PREFIX}${REMOTE} ${COMMIT}"
    git fetch -t ${REMOTE_URL} ${COMMIT}
    git checkout -f FETCH_HEAD
  fi

  echo "Setting version..."
  echo "-$pkgrel" > localversion.10-pkgrel
  echo "${pkgbase#linux}" > localversion.20-pkgname

  local src
  for src in "${source[@]}"; do
    src="${src%%::*}"
    src="${src##*/}"
    [[ $src = *.patch ]] || continue
    echo "Applying patch $src..."
    patch -Np1 < "../$src"
  done

  if [[ -z ${IGNORE_USER_CUSTOM} ]]; then
    echo
    echo "====================================== User Patches ======================================"
    if [[ -f "$_userpatches" ]]; then
      source "$_userpatches"
      PATCHES_PREFIX=${_userpatches%%patches}
    else
      source "../${_userpatches##*/}"
      PATCHES_PREFIX="../../"
    fi
    for src in "${PATCHES[@]}"; do
      [[ $src = *.patch ]] || continue
      echo " -> Applying user patch $src..."
      patch -Np1 < "${PATCHES_PREFIX}$src"
      _USER_PATCHES="1"
    done
    echo "=========================================================================================="
    echo
    # Let user see the patches were applied before proceed with the build
    [[ ! -z ${_USER_PATCHES} ]] && sleep 5
  fi

  echo "Setting config..."
  cat ../config ../config.extra > .config
  if [[ -z ${IGNORE_USER_CUSTOM} ]]; then
    if [[ -f "$_userconfig" ]]; then
      cat $_userconfig >> .config
    else
      cat ../config.user >> .config
    fi
  fi
  make olddefconfig
  diff -u ../config .config || :

  make -s kernelrelease > version
  echo "Prepared $pkgbase version $(<version)"
}

build() {
  cd $_srcname
  make all
  make -C tools/bpf/bpftool vmlinux.h feature-clang-bpf-co-re=1
}

_package() {
  pkgdesc="The $pkgdesc kernel and modules"
  depends=(
    coreutils
    initramfs
    kmod
  )
  optdepends=(
    'linux-firmware: firmware images needed for some devices'
    'scx-scheds: to use sched-ext schedulers'
    'wireless-regdb: to set the correct wireless channels of your country'
  )
  provides=(
    KSMBD-MODULE
    NTSYNC-MODULE
    VIRTUALBOX-GUEST-MODULES
    WIREGUARD-MODULE
  )
  replaces=(
    virtualbox-guest-modules-arch
    wireguard-arch
  )
  install="${pkgbase}.install"

  cd $_srcname
  local modulesdir="$pkgdir/usr/lib/modules/$(<version)"

  echo "Installing boot image..."
  # systemd expects to find the kernel here to allow hibernation
  # https://github.com/systemd/systemd/commit/edda44605f06a41fb86b7ab8128dcf99161d2344
  install -Dm644 "$(make -s image_name)" "$modulesdir/vmlinuz"

  # Used by mkinitcpio to name the kernel
  echo "$pkgbase" | install -Dm644 /dev/stdin "$modulesdir/pkgbase"

  echo "Installing modules..."
  ZSTD_CLEVEL=19 make INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 \
    DEPMOD=/doesnt/exist modules_install  # Suppress depmod

  # remove build link
  rm -f "$modulesdir"/build

  # install config files
  install -Dm644 $srcdir/config.user "${pkgdir}${_userconfig}"
  install -Dm644 $srcdir/remote "${pkgdir}${_userremote}"
  install -Dm644 $srcdir/patches "${pkgdir}${_userpatches}"
}

_package-headers() {
  pkgdesc="Headers and scripts for building modules for the $pkgdesc kernel"
  depends=(pahole)

  cd $_srcname
  local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

  echo "Installing build files..."
  install -Dt "$builddir" -m644 .config Makefile Module.symvers System.map \
    localversion.* version vmlinux tools/bpf/bpftool/vmlinux.h
  install -Dt "$builddir/kernel" -m644 kernel/Makefile
  install -Dt "$builddir/arch/x86" -m644 arch/x86/Makefile
  cp -t "$builddir" -a scripts
  ln -srt "$builddir" "$builddir/scripts/gdb/vmlinux-gdb.py"

  # required when STACK_VALIDATION is enabled
  install -Dt "$builddir/tools/objtool" tools/objtool/objtool

  # required when DEBUG_INFO_BTF_MODULES is enabled
  install -Dt "$builddir/tools/bpf/resolve_btfids" tools/bpf/resolve_btfids/resolve_btfids

  echo "Installing headers..."
  cp -t "$builddir" -a include
  cp -t "$builddir/arch/x86" -a arch/x86/include
  install -Dt "$builddir/arch/x86/kernel" -m644 arch/x86/kernel/asm-offsets.s

  install -Dt "$builddir/drivers/md" -m644 drivers/md/*.h
  install -Dt "$builddir/net/mac80211" -m644 net/mac80211/*.h

  # https://bugs.archlinux.org/task/13146
  install -Dt "$builddir/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h

  # https://bugs.archlinux.org/task/20402
  install -Dt "$builddir/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
  install -Dt "$builddir/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
  install -Dt "$builddir/drivers/media/tuners" -m644 drivers/media/tuners/*.h

  # https://bugs.archlinux.org/task/71392
  install -Dt "$builddir/drivers/iio/common/hid-sensors" -m644 drivers/iio/common/hid-sensors/*.h

  echo "Installing KConfig files..."
  find . -name 'Kconfig*' -exec install -Dm644 {} "$builddir/{}" \;

  echo "Installing Rust files..."
  install -Dt "$builddir/rust" -m644 rust/*.rmeta
  install -Dt "$builddir/rust" rust/*.so

  echo "Installing unstripped VDSO..."
  make INSTALL_MOD_PATH="$pkgdir/usr" vdso_install \
    link=  # Suppress build-id symlinks

  echo "Removing unneeded architectures..."
  local arch
  for arch in "$builddir"/arch/*/; do
    [[ $arch = */x86/ ]] && continue
    echo "Removing $(basename "$arch")"
    rm -r "$arch"
  done

  echo "Removing documentation..."
  rm -r "$builddir/Documentation"

  echo "Removing broken symlinks..."
  find -L "$builddir" -type l -printf 'Removing %P\n' -delete

  echo "Removing loose objects..."
  find "$builddir" -type f -name '*.o' -printf 'Removing %P\n' -delete

  echo "Stripping build tools..."
  local file
  while read -rd '' file; do
    case "$(file -Sib "$file")" in
      application/x-sharedlib\;*)      # Libraries (.so)
        strip -v $STRIP_SHARED "$file" ;;
      application/x-archive\;*)        # Libraries (.a)
        strip -v $STRIP_STATIC "$file" ;;
      application/x-executable\;*)     # Binaries
        strip -v $STRIP_BINARIES "$file" ;;
      application/x-pie-executable\;*) # Relocatable binaries
        strip -v $STRIP_SHARED "$file" ;;
    esac
  done < <(find "$builddir" -type f -perm -u+x ! -name vmlinux -print0)

  echo "Stripping vmlinux..."
  strip -v $STRIP_STATIC "$builddir/vmlinux"

  echo "Adding symlink..."
  mkdir -p "$pkgdir/usr/src"
  ln -sr "$builddir" "$pkgdir/usr/src/$pkgbase"
}

pkgname=("$pkgbase" "$pkgbase-headers")
for _p in "${pkgname[@]}"; do
  eval "package_$_p() {
    $(declare -f "_package${_p#$pkgbase}")
    _package${_p#$pkgbase}
  }"
done

# vim:set ts=8 sts=2 sw=2 et:
