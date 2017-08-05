# Maintainer: Eric Vidal <eric@obarun.org>
# based on the original https://projects.archlinux.org/svntogit/packages.git/tree/trunk?h=packages/util-linux&id=6b9f34ed258a54c4d4a27f2edabaaf5bc5e616df
# 						Maintainer: Tom Gundersen <teg@jklm.no>
# 						Maintainer: Dave Reisner <dreisner@archlinux.org>
# 						Contributor: judd <jvinet@zeroflux.org>

pkgbase=util-linux
pkgname=(util-linux libutil-linux)
_pkgmajor=2.30
pkgver=${_pkgmajor}.1
pkgrel=3
pkgdesc="Miscellaneous system utilities for Linux"
url="https://www.kernel.org/pub/linux/utils/util-linux/"
arch=('x86_64')
makedepends=('python' 'libcap-ng')
optdepends=('uuid-s6serv: uuid s6 service'
			'uuid-runitserv: uuid runit service')
license=('GPL2')
options=('strip' 'debug')
validpgpkeys=('B0C64D14301CC6EFAEDF60E4E4B71D5EEC39C284')  # Karel Zak
source=("https://www.kernel.org/pub/linux/utils/util-linux/v$_pkgmajor/${pkgbase}-$pkgver.tar.xz"
        pam-{login,common,su}
        'util-linux.sysusers')
md5sums=('5e5ec141e775efe36f640e62f3f8cd0d'
         '4368b3f98abd8a32662e094c54e7f9b1'
         'a31374fef2cba0ca34dfc7078e2969e4'
         'fa85e5cce5d723275b14365ba71a8aad'
         'dfc9904f67ebc54bb347ca3cc430ef2b')
validpgpkeys=('6DD4217456569BA711566AC7F06E8FDE7B45DAAC') # Eric Vidal


build() {
  cd "${pkgbase}-$pkgver"

  ./configure --prefix=/usr \
              --libdir=/usr/lib \
              --bindir=/usr/bin \
              --localstatedir=/run \
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
  conflicts=('eject' 'zramctl')
  provides=('eject' 'zramctl')
  replaces=('zramctl')
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
}

package_libutil-linux() {
  pkgdesc="util-linux runtime libraries"
  depends=('glibc')
  provides=("libutil-linux=$pkgver" 'libblkid.so' 'libfdisk.so' 'libmount.so' 'libsmartcols.so' 'libuuid.so')
  	
  make -C "${pkgbase}-$pkgver" DESTDIR="$pkgdir" install-usrlib_execLTLIBRARIES
}