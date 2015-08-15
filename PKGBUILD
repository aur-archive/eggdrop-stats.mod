# Maintainer: Jiri Barous (amunak) <amunak at amunak dot net>

pkgname=eggdrop-stats.mod
pkgver=1.4.0.dev20
pkgrel=2
_eggver=1.6.21
pkgdesc="Statistics mod for Eggdrop."
arch=('i686' 'x86_64')
url="http://www.kreativrauschen.com/stats.mod/"
license=('GPL2')
depends=("eggdrop>=${_eggver}")
makedepends=('hardened-cc')
source=("http://ftp.eggheads.org/pub/eggdrop/source/1.6/eggdrop${_eggver}.tar.bz2"
		"http://www.kreativrauschen.com/stats.mod/download/stats.mod.${pkgver}.tar.gz"
		'dlopen.c')
backup=("etc/${pkgname}.conf")
options=('!makeflags')
sha256sums=('75bd5573a609eac3940c0b6ca8251c3f38ea5d54b520e1cad93c650b4bc21754'
			'0d81ee8576fc40625bc6e6fe7d72be33b3f9e666c016b3abe023957ffb546dd1'
			'c6e8a113c570c8bdde23741d2d8c25e28fea42e158a09f375e3710d9ba336d44')

# Want multiple *unique* installations? Change the pkgname, it will just work.
_usrdir='/usr'
_sharedir="${_usrdir}/share"
_modulesdir="${_usrdir}/lib/eggdrop"
_scriptsdir="${_sharedir}/eggdrop/scripts"
_langsdir="${_sharedir}/eggdrop/language"
readonly -a _usrdir _sharedir _modulesdir _scriptsdir _langsdir

build() {
	cd "${srcdir}/eggdrop${_eggver}"
	cp -r "${srcdir}/stats.mod" src/mod

	# Force hardened build unless CC is already set.
	if [[ ! $CC ]]; then
		export CC='/usr/bin/hgcc'
	fi

	# Don't complain about language files on startup.
	sed -i "s|\"./language\"|\"${_langsdir}\"|g" src/eggdrop.h

	./configure# --with-tcllib='/usr/lib/libtcl8.6.so' --with-tclinc='/usr/include/tcl.h'
	make config

	# Include LDFLAGS.
	sed -i "s|-L/usr/lib|${LDFLAGS}|g" Makefile
	make
}

check() {
	cd "${srcdir}/eggdrop${_eggver}"

	# If this fails, theres a rather good chance something is broken.
	gcc -O2 "${srcdir}/dlopen.c" -o dlopen -ldl

	if ! env LD_PRELOAD='' ./dlopen stats.so ; then
		error 'Sanity check failed: stats.so cannot be loaded. Proceeding anyway..'
	fi

	rm dlopen
}

package() {
	readonly eggtmp="${pkgdir}/tmp"

	# This is ugly..

	cd "${srcdir}/eggdrop${_eggver}"
	make DEST="$eggtmp" install

	mkdir -p -m 0755 "${pkgdir}/etc" \
		"${pkgdir}/${_usrdir}/lib" \
		"${pkgdir}/${_sharedir}/eggdrop/"{language,scripts} \
		"${pkgdir}/${_modulesdir}"

	install -D "$eggtmp/modules-${_eggver}/stats.so" "${pkgdir}/${_modulesdir}"

	install -Dm 0644 src/mod/stats.mod/language/stats.*.lang "${pkgdir}${_langsdir}"
	install -Dm 0644 src/mod/stats.mod/addons/*.tcl "${pkgdir}${_scriptsdir}"

	sed -e '2d' \
		-e "s@scripts/@${_scriptsdir}/@g" \
		-e "s@modules/@${_modulesdir}/@g" \
		-e "s@language/@${_langsdir}/@g" \
		src/mod/stats.mod/stats.conf > "${pkgdir}/etc/${pkgname}.conf"

	rm -r "$eggtmp"
}
