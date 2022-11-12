# Maintainer: Levente Polyak <anthraxx[at]archlinux[dot]org>
# Maintainer: Frederik Schwan <freswa at archlinux dot org>
# Contributor: Guillaume ALAUX <guillaume@archlinux.org>

# TODO add test, see about packaging jtreg and using it here

pkgbase=java-openjdk-wakefield
pkgname=('jre-openjdk-wakefield-headless' 'jre-openjdk-wakefield' 'jdk-openjdk-wakefield' 'openjdk-wakefield-src' 'openjdk-wakefield-doc')
_majorver=19
_minorver=0
# _securityver=1
# _updatever=10
# pkgver=${_majorver}.${_minorver}.${_securityver}.u${_updatever}
pkgver=${_majorver}
pkgrel=1
arch=('x86_64')
url='https://openjdk.org/projects/wakefield/'
license=('custom')
makedepends=('java-environment>=11' 'cpio' 'unzip' 'zip' 'libelf' 'libcups' 'libx11'
	'libxrender' 'libxtst' 'libxt' 'libxext' 'libxrandr' 'alsa-lib' 'pandoc'
	'graphviz' 'freetype2' 'libjpeg-turbo' 'giflib' 'libpng' 'lcms2'
	'libnet' 'bash' 'harfbuzz' 'gcc-libs' 'glibc' 'wayland')
options=(!lto)
source=(git+https://github.com/openjdk/wakefield.git#branch=pure_wl_toolkit
	freedesktop-java.desktop
	freedesktop-jconsole.desktop
	freedesktop-jshell.desktop)
sha256sums=('SKIP'
	'abf5527f32d6ef7dbc07bbeb1012d40cbba4e98ef39b0afc0bc9222d33802284'
	'3f11df00c5b8024b6ef5a8af8d3ff31ab1762b7ddb89f8503c068fc4fbfd7204'
	'810e7a532b2a43d38a58213c2542f2b835c67419e5f5561b7efd89d349825258')

case "${CARCH}" in
x86_64) _JARCH='x86_64' ;;
i686) _JARCH='x86' ;;
esac

_jvmdir=/usr/lib/jvm/java-${_majorver}-openjdk-wakefield
_jdkdir=wakefield
_imgdir=${_jdkdir}/build/linux-${_JARCH}-server-release/images

_nonheadless=(lib/libawt_xawt.{so,debuginfo}
	lib/libjawt.{so,debuginfo}
	lib/libjsound.{so,debuginfo}
	lib/libsplashscreen.{so,debuginfo})

build() {
	cd ${_jdkdir}

	NUM_PROC_OPT=''
	MAKEFLAG_J=$(echo ${MAKEFLAGS} | sed -En 's/.*-j([0-9]+).*/\1/p')
	if [ -n "${MAKEFLAG_J}" ]; then
		# http://hg.openjdk.java.net/jdk10/jdk10/file/85e6cb013b98/make/InitSupport.gmk#l105
		echo "Removing '-j${MAKEFLAG_J}' from MAKEFLAGS to prevent build fail. Passing it directly to ./configure."
		export MAKEFLAGS=${MAKEFLAGS/-j${MAKEFLAG_J}/}
		NUM_PROC_OPT="--with-num-cores=${MAKEFLAG_J}"
	fi

	# Avoid optimization of HotSpot to be lowered from O3 to O2
	local _CFLAGS="${CFLAGS//-O2/-O3} ${CPPFLAGS} -fcommon"
	local _CXXFLAGS="${CXXFLAGS//-O2/-O3} ${CPPFLAGS} -fcommon"
	local _LDFLAGS=${LDFLAGS}
	if [[ ${CARCH} = i686 ]]; then
		echo "Removing '-fno-plt' from CFLAGS and CXXFLAGS to prevent build fail with this architecture"
		_CFLAGS=${CFLAGS/-fno-plt/}
		_CXXFLAGS=${CXXFLAGS/-fno-plt/}
	fi

	# TODO: Should be rechecked for the next releases
	# compiling with -fexceptions leads to:
	# /usr/bin/ld: /build/java-openjdk/src/jdk17u-jdk-17.0.3-2/build/linux-x86_64-server-release/hotspot/variant-server/libjvm/objs/zPhysicalMemory.o: in function `ZList<ZMemory>::~ZList()':
	# /build/java-openjdk/src/jdk17u-jdk-17.0.3-2/src/hotspot/share/gc/z/zList.hpp:54: undefined reference to `ZListNode<ZMemory>::~ZListNode()'
	# collect2: error: ld returned 1 exit status
	_CFLAGS=${CFLAGS/-fexceptions/}
	_CXXFLAGS=${CXXFLAGS/-fexceptions/}

	# CFLAGS, CXXFLAGS and LDFLAGS are ignored as shown by a warning
	# in the output of ./configure unless used like such:
	#  --with-extra-cflags="${CFLAGS}"
	#  --with-extra-cxxflags="${CXXFLAGS}"
	#  --with-extra-ldflags="${LDFLAGS}"
	# See also paragraph "Configure Control Variables from "jdk${_majorver}-${_git_tag}/common/doc/building.md
	unset CFLAGS
	unset CXXFLAGS
	unset LDFLAGS

	bash configure \
		--with-version-pre="" \
		--with-version-opt="" \
		--with-stdc++lib=dynamic \
		--with-extra-cflags="${_CFLAGS}" \
		--with-extra-cxxflags="${_CXXFLAGS}" \
		--with-extra-ldflags="${_LDFLAGS}" \
		--with-libjpeg=system \
		--with-giflib=system \
		--with-libpng=system \
		--with-lcms=system \
		--with-zlib=system \
		--with-harfbuzz=system \
		--with-jvm-features=zgc,shenandoahgc \
		--enable-unlimited-crypto \
		--disable-warnings-as-errors \
		--with-wayland=/usr \
		${NUM_PROC_OPT}
	#--disable-javac-server

	make images legacy-jre-image docs

	# https://bugs.openjdk.java.net/browse/JDK-8173610
	find "../${_imgdir}" -iname '*.so' -exec chmod +x {} \;
}

check() {
	cd ${_jdkdir}
	# TODO package jtreg
	# make -k check
}

package_jre-openjdk-wakefield-headless() {
	pkgdesc="Wakefield OpenJDK Java ${_majorver} headless runtime environment"
	depends=('java-runtime-common>=3' 'ca-certificates-utils' 'nss' 'libjpeg-turbo' 'libjpeg.so'
		'lcms2' 'liblcms2.so' 'libnet' 'freetype2' 'libfreetype.so' 'harfbuzz' 'libharfbuzz.so'
		'glibc' 'gcc-libs')
	optdepends=('java-rhino: for some JavaScript support')
	provides=("java-runtime-headless=${_majorver}" "java-runtime-headless-openjdk-wakefield=${_majorver}" "jre${_majorver}-openjdk-wakefield-headless=${pkgver}-${pkgrel}")
	backup=(etc/${pkgbase}/logging.properties
		etc/${pkgbase}/management/jmxremote.access
		etc/${pkgbase}/management/jmxremote.password.template
		etc/${pkgbase}/management/management.properties
		etc/${pkgbase}/net.properties
		etc/${pkgbase}/security/java.policy
		etc/${pkgbase}/security/java.security
		etc/${pkgbase}/security/policy/README.txt
		etc/${pkgbase}/security/policy/limited/default_US_export.policy
		etc/${pkgbase}/security/policy/limited/default_local.policy
		etc/${pkgbase}/security/policy/limited/exempt_local.policy
		etc/${pkgbase}/security/policy/unlimited/default_US_export.policy
		etc/${pkgbase}/security/policy/unlimited/default_local.policy
		etc/${pkgbase}/sound.properties)
	install=install_jre-openjdk-wakefield-headless.sh

	cd ${_imgdir}/jre

	install -dm 755 "${pkgdir}${_jvmdir}"

	cp -a bin lib \
		"${pkgdir}${_jvmdir}"

	for f in "${_nonheadless[@]}"; do
		rm "${pkgdir}${_jvmdir}/${f}"
	done

	cp ../jdk/release "${pkgdir}${_jvmdir}"
	cp ../jdk/lib/modules "${pkgdir}${_jvmdir}/lib"

	# Conf
	install -dm 755 "${pkgdir}/etc"
	cp -r conf "${pkgdir}/etc/${pkgbase}"
	ln -s /etc/${pkgbase} "${pkgdir}/${_jvmdir}/conf"

	# Legal
	install -dm 755 "${pkgdir}/usr/share/licenses"
	cp -r legal "${pkgdir}/usr/share/licenses/${pkgbase}"
	ln -s ${pkgbase} "${pkgdir}/usr/share/licenses/${pkgname}"
	ln -s /usr/share/licenses/${pkgbase} "${pkgdir}/${_jvmdir}/legal"

	# Man pages
	for f in bin/*; do
		f=$(basename "${f}")
		_man=../jdk/man/man1/"${f}.1"
		test -f "${_man}" && install -Dm 644 "${_man}" "${pkgdir}/usr/share/man/man1/${f}-openjdk-wakefield${_majorver}.1"
	done
	ln -s /usr/share/man "${pkgdir}/${_jvmdir}/man"

	# Link JKS keystore from ca-certificates-utils
	rm -f "${pkgdir}${_jvmdir}/lib/security/cacerts"
	ln -sf /etc/ssl/certs/java/cacerts "${pkgdir}${_jvmdir}/lib/security/cacerts"
}

package_jre-openjdk-wakefield() {
	pkgdesc="Wakefield OpenJDK Java ${_majorver} full runtime environment"
	depends=("jre${_majorver}-openjdk-wakefield-headless=${pkgver}-${pkgrel}" 'giflib' 'libgif.so'
		'glibc' 'gcc-libs' 'libpng')
	optdepends=('alsa-lib: for basic sound support'
		'gtk2: for the Gtk+ 2 look and feel - desktop usage'
		'gtk3: for the Gtk+ 3 look and feel - desktop usage')
	provides=("java-runtime=${_majorver}" "java-runtime-openjdk-wakefield=${_majorver}" "jre${_majorver}-openjdk-wakefield=${pkgver}-${pkgrel}")
	install=install_jre-openjdk-wakefield.sh

	cd ${_imgdir}/jre

	install -dm 755 "${pkgdir}${_jvmdir}"

	for f in "${_nonheadless[@]}"; do
		install -Dm 644 ${f} "${pkgdir}${_jvmdir}/${f}"
	done

	# Licenses
	install -dm 755 "${pkgdir}/usr/share/licenses"
	ln -s ${pkgbase} "${pkgdir}/usr/share/licenses/${pkgname}"
}

package_jdk-openjdk-wakefield() {
	pkgdesc="Wakefield OpenJDK Java ${_majorver} development kit"
	depends=("jre${_majorver}-openjdk-wakefield=${pkgver}-${pkgrel}" 'java-environment-common=3'
		'hicolor-icon-theme' 'libelf' 'glibc' 'gcc-libs')
	provides=("java-environment=${_majorver}" "java-environment-openjdk-wakefield=${_majorver}" "jdk${_majorver}-openjdk-wakefield=${pkgver}-${pkgrel}")
	install=install_jdk-openjdk-wakefield.sh

	cd ${_imgdir}/jdk

	install -dm 755 "${pkgdir}${_jvmdir}"

	cp -a bin demo include jmods lib \
		"${pkgdir}${_jvmdir}"

	rm "${pkgdir}${_jvmdir}/lib/src.zip"

	# Remove files held by JRE
	pushd ../jre
	for d in bin lib; do
		find ${d} ! -type d -exec rm "${pkgdir}${_jvmdir}/{}" \;
	done
	popd
	find "${pkgdir}${_jvmdir}/lib" -type d -empty -delete

	# Conf files all belong to JRE

	# Legal
	install -dm 755 "${pkgdir}/usr/share/licenses"
	cp -r legal "${pkgdir}/usr/share/licenses/${pkgbase}"
	pushd ../jre/legal
	find . ! -type d -exec rm "${pkgdir}/usr/share/licenses/${pkgbase}/{}" \;
	popd
	find "${pkgdir}/usr/share/licenses" -type d -empty -delete
	ln -s ${pkgbase} "${pkgdir}/usr/share/licenses/${pkgname}"

	# Man pages
	for f in bin/*; do
		f=$(basename "${f}")
		_man=man/man1/"${f}.1"
		test -f "../jre/bin/${f}" && continue
		test -f "${_man}" && install -Dm 644 "${_man}" "${pkgdir}/usr/share/man/man1/${f}-openjdk-wakefield${_majorver}.1"
	done

	# Icons
	for s in 16 24 32 48; do
		install -Dm 644 \
			"${srcdir}/${_jdkdir}/src/java.desktop/unix/classes/sun/awt/X11/java-icon${s}.png" \
			"${pkgdir}/usr/share/icons/hicolor/${s}x${s}/apps/${pkgbase}.png"
	done

	# Desktop files
	for f in jconsole java jshell; do
		install -Dm 644 \
			"${srcdir}/freedesktop-${f}.desktop" \
			"${pkgdir}/usr/share/applications/${f}-${pkgbase}.desktop"
	done
}

package_openjdk-wakefield-src() {
	pkgdesc="Wakefield OpenJDK Java ${_majorver} sources"
	# Depends on JDK to get license files
	depends=("jdk${_majorver}-openjdk-wakefield=${pkgver}-${pkgrel}")
	provides=("openjdk-wakefield${_majorver}-src=${pkgver}-${pkgrel}")

	install -Dm 644 -t "${pkgdir}${_jvmdir}/lib" ${_imgdir}/jdk/lib/src.zip

	install -dm 755 "${pkgdir}/usr/share/licenses"
	ln -s ${pkgbase} "${pkgdir}/usr/share/licenses/${pkgname}"
}

package_openjdk-wakefield-doc() {
	pkgdesc="Wakefield OpenJDK Java ${_majorver} documentation"
	# Depends on JDK to get license files
	depends=("jdk${_majorver}-openjdk-wakefield=${pkgver}-${pkgrel}")
	provides=("openjdk-wakefield${_majorver}-doc=${pkgver}-${pkgrel}")

	install -dm 755 "${pkgdir}/usr/share/doc"
	cp -r ${_imgdir}/docs "${pkgdir}/usr/share/doc/${pkgbase}"

	install -dm 755 "${pkgdir}/usr/share/licenses"
	ln -s ${pkgbase} "${pkgdir}/usr/share/licenses/${pkgname}"
}

# vim: ts=2 sw=2 et:
