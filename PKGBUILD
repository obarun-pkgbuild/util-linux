# Maintainer: Eric Vidal <eric@obarun.org>
# based on the original https://projects.archlinux.org/svntogit/packages.git/tree/trunk?h=packages/util-linux&id=6b9f34ed258a54c4d4a27f2edabaaf5bc5e616df
# 						Maintainer: Tom Gundersen <teg@jklm.no>
# 						Maintainer: Dave Reisner <dreisner@archlinux.org>
# 						Contributor: judd <jvinet@zeroflux.org>

pkgbase=util-linux
pkgname=(util-linux libutil-linux)
_pkgmajor=2.32
pkgver=${_pkgmajor}
pkgrel=4
pkgdesc="Miscellaneous system utilities for Linux"
url="https://www.kernel.org/pub/linux/utils/util-linux/"
arch=('x86_64')
makedepends=('python' 'libcap-ng')
license=('GPL2')
options=('strip' 'debug')
validpgpkeys=('B0C64D14301CC6EFAEDF60E4E4B71D5EEC39C284')  # Karel Zak
source=("https://www.kernel.org/pub/linux/utils/util-linux/v$_pkgmajor/${pkgbase}-$pkgver.tar.xz"
        pam-{login,common,su}
        '60-rfkill.rules'
        'util-linux.sysusers'
        '0001-fstrim-cleanup-uncludes.patch'
        '0002-libmount_include_sys_mount_h_only_if_necessary.patch')
sha256sums=('6c7397abc764e32e8159c2e96042874a190303e77adceb4ac5bd502a272a4734'
            '993a3096c2b113e6800f2abbd5d4233ebf1a97eef423990d3187d665d3490b92'
            'fc6807842f92e9d3f792d6b64a0d5aad87995a279153ab228b1b2a64d9f32f20'
            '51eac9c2a2f51ad3982bba35de9aac5510f1eeff432d2d63c6362e45d620afc0'
            '7423aaaa09fee7f47baa83df9ea6fef525ff9aec395c8cbd9fe848ceb2643f37'
            '0abcfa0d8d5b48561ddcb663b14877d0d3ad7a04074c19d2232c95e25608e67a'
            'b1c0db069f60c086351f7a6fb7ff1add1c41e9ed7c1b65bd67e6604943fc4c75'
            'c65d8354fc2773035be8ac071541f4709763366d22a8a8f1bc2cb3401dcfdf69')
validpgpkeys=('6DD4217456569BA711566AC7F06E8FDE7B45DAAC') # Eric Vidal

prepare() {
  cd "$pkgbase-$pkgver"

  patch -Np1 -i ../0001-fstrim-cleanup-uncludes.patch
  patch -Np1 -i ../0002-libmount_include_sys_mount_h_only_if_necessary.patch
}

build() {
  cd "${pkgbase}-$pkgver"

  ./configure --prefix=/usr \
              --libdir=/usr/lib \
              --bindir=/usr/bin \
              --localstatedir=/var \
              --enable-fs-paths-extra=/usr/bin \
              --enable-raw \
              --enable-vipw \
              --enable-newgrp \
              --enable-chfn-chsh \
              --enable-write \
              --enable-mesg \
              --with-python=3 \
              --with-systemdsystemunitdir=no \
			  --with-systemd=no 
			  #--enable-libmount-force-mountinfo \ #recommended for systemd
              #--enable-socket-activation \ #??? option not available 
			  	
  make
}

package_util-linux() {
  conflicts=('eject' 'zramctl' 'rfkill')
  provides=('eject' 'zramctl' 'rfkill')
  replaces=('zramctl' 'rfkill')
  depends=('pam' 'shadow' 'coreutils' 'libutil-linux' 'eudev-obarun' 'libcap-ng')
  optdepends=('python: python bindings to libmount'
			  'uuid-s6serv: uuid s6 service')
  groups=('base' 'base-devel')
  backup=(etc/pam.d/chfn
          etc/pam.d/chsh
          etc/pam.d/login
          etc/pam.d/su
          etc/pam.d/su-l)

  cd "${pkgbase}-$pkgver"

  make DESTDIR="$pkgdir" install

  # setuid chfn and chsh
  chmod 4755 "$pkgdir"/usr/bin/{newgrp,ch{sh,fn}}

  # install PAM files for login-utils
  install -Dm644 "$srcdir/pam-common" "$pkgdir/etc/pam.d/chfn"
  install -m644 "$srcdir/pam-common" "$pkgdir/etc/pam.d/chsh"
  install -m644 "$srcdir/pam-login" "$pkgdir/etc/pam.d/login"
  install -m644 "$srcdir/pam-su" "$pkgdir/etc/pam.d/su"
  install -m644 "$srcdir/pam-su" "$pkgdir/etc/pam.d/su-l"

  # adjust for usrmove
  # TODO(dreisner): fix configure.ac upstream so that this isn't needed
  cd "$pkgdir"
  mv {,usr/}sbin/* usr/bin
  rmdir sbin usr/sbin

  ### create libutil-linux split
  ### runtime libs are shipped as part of libutil-linux
  rm "$pkgdir"/usr/lib/lib*.{a,so}*
  
  ### install pacopts applysys
  install -Dm644 "$srcdir/util-linux.sysusers" \
	"$pkgdir/usr/lib/sysusers.d/util-linux.conf"

  install -Dm644 "$srcdir/60-rfkill.rules" \
    "$pkgdir/usr/lib/udev/rules.d/60-rfkill.rules"
}

package_libutil-linux() {
  pkgdesc="util-linux runtime libraries"
  depends=('glibc')
  provides=("libutil-linux=$pkgver" 'libblkid.so' 'libfdisk.so' 'libmount.so' 'libsmartcols.so' 'libuuid.so')
  	
  make -C "${pkgbase}-$pkgver" DESTDIR="$pkgdir" install-usrlib_execLTLIBRARIES
}
