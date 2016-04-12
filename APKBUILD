# Maintainer: Caleb Champlin <caleb.champlin@gmail.com>

pkgname=nginx-pagespeed
pkgver=1.10.33.7
pkgrel=0
pkgdesc="Google Pagespeed module for NGINX"
url="https://developers.google.com/speed/pagespeed/module/"
arch="x86_64"
license="custom"
depends="nginx libuuid apr apr-util libjpeg-turbo icu icu-libs"

_nginx_pkg="nginx"
_nginx_ver=1.9.11

_libpng_lib="libpng12"
_libpng_ver="1.2.56"
_libpng_pkg="libpng"

_ngx_pagespeed_pkg="ngx_pagespeed"
_ngx_pagespeed_ver=$pkgver
_ngx_pagespeed_type="beta"

_mod_ps="modpagespeed"
makedepends="apr-dev apr-util-dev zlib-dev linux-headers openssl-dev libjpeg-turbo-dev icu-dev gperf"

source="mod-pagespeed-$pkgver.tar.bz2::https://dl.google.com/dl/linux/mod-pagespeed/tar/beta/mod-pagespeed-beta-$pkgver-r0.tar.bz2
        $_nginx_pkg-$_nginx_ver.tar.gz::http://nginx.org/download/$_nginx_pkg-$_nginx_ver.tar.gz
	$_libpng_pkg-$_libpng_ver.tar.gz::ftp://ftp.simplesystems.org/pub/libpng/png/src/$_libpng_lib/$_libpng_pkg-$_libpng_ver.tar.gz
	$_ngx_pagespeed_pkg-$_ngx_pagespeed_ver.tar.gz::https://github.com/pagespeed/ngx_pagespeed/archive/v$_ngx_pagespeed_ver-$_ngx_pagespeed_type.tar.gz
	pthread_nonrecursive_np.patch
	stack_trace_posix.patch
	libpng_cflags.patch
	automatic_makefile.patch
	rename_c_symbols.patch"


_pkgdir="$srcdir"/$_mod_ps-$pkgver
_builddir="$srcdir"/$_mod_ps-$pkgver/src
_pngdir="$srcdir"/$_libpng_pkg-$_libpng_ver
_ngx_pagespeeddir="$srcdir"/$_ngx_pagespeed_pkg-$_ngx_pagespeed_ver-$_ngx_pagespeed_type
_nginxdir="$srcdir"/$_nginx_pkg-$_nginx_ver
prepare() {
	cd "$_pkgdir"
	for i in $source; do
		case $i in
		*.patch) msg $i; patch -p1 -i "$srcdir"/$i || return 1;;
		esac
	done
}

build() {
	_buildpng
	cd "$_pkgdir"
	./generate.sh -D use_system_libs=1 -D _GLIBCXX_USE_CXX11_ABI=0 -D use_system_icu=1 || return 1
	cd "$_builddir"
	make BUILDTYPE=Release CXXFLAGS=" -I/usr/include/apr-1 -I$_pngdir -fPIC -D_GLIBCXX_USE_CXX11_ABI=0" CFLAGS=" -I/usr/include/apr-1 -I$_pngdir -fPIC -D_GLIBCXX_USE_CXX11_ABI=0" || return 1
	cd "$_builddir"/pagespeed/automatic
	make psol BUILDTYPE=Release CXXFLAGS=" -I/usr/include/apr-1 -I$_pngdir -fPIC -D_GLIBCXX_USE_CXX11_ABI=0" CFLAGS=" -I/usr/include/apr-1 -I$_pngdir -fPIC -D_GLIBCXX_USE_CXX11_ABI=0" || return 1
	mkdir -p $_ngx_pagespeeddir/psol
        mkdir -p $_ngx_pagespeeddir/psol/lib/Release/linux/x64
        mkdir -p $_ngx_pagespeeddir/psol/include/out/Release
	cp -r "$_builddir"/out/Release/obj $_ngx_pagespeeddir/psol/include/out/Release/
	cp -r "$_builddir"/net $_ngx_pagespeeddir/psol/include/
	cp -r "$_builddir"/testing $_ngx_pagespeeddir/psol/include/
	cp -r "$_builddir"/pagespeed $_ngx_pagespeeddir/psol/include/
	cp -r "$_builddir"/third_party $_ngx_pagespeeddir/psol/include/
	cp -r "$_builddir"/tools $_ngx_pagespeeddir/psol/include/
	cp -r "$_builddir"/pagespeed/automatic/pagespeed_automatic.a $_ngx_pagespeeddir/psol/lib/Release/linux/x64 || return 1
	cd $_nginxdir
	LD_LIBRARY_PATH=$pkgdir/usr/lib ./configure --with-ipv6 \
	            --with-file-aio \
		    --with-pcre-jit \
		    --with-http_dav_module \
		    --with-http_ssl_module \
		    --with-http_stub_status_module \
		    --with-http_gzip_static_module \
		    --with-http_v2_module \
		    --with-http_auth_request_module \
		    --with-mail \
		    --with-mail_ssl_module \
		    --add-dynamic-module=$_ngx_pagespeeddir \
		    --with-cc-opt="-fPIC -I /usr/include/apr-1" \
		    --with-ld-opt="-luuid -lapr-1 -laprutil-1 -licudata -licuuc -L$pkgdir/usr/lib -lpng12 -lturbojpeg -ljpeg" || return 1
	make || return 1
} 

package() {
  cd $_nginxdir
  make DESTDIR="$pkgdir" install || return 1

  # We don't want to include all of nginx only the pagespeed dynamic library
  rm -rf "$pkgdir"/usr/local/nginx/sbin
  rm -rf "$pkgdir"/usr/local/nginx/conf
  rm -rf "$pkgdir"/usr/local/nginx/html
  rm -rf "$pkgdir"/usr/local/nginx/logs

  # We only want libpng12 libraries, none of the extra stuff
  rm -rf "$pkgdir"/usr/bin
  rm -rf "$pkgdir"/usr/share
  rm -rf "$pkgdir"/usr/include
  rm "$pkgdir"/usr/lib/libpng.*
  rm "$pkgdir"/usr/lib/pkgconfig/libpng.pc

  cd $_ngx_pagespeeddir
  install -Dm644 LICENSE "$pkgdir"/usr/share/licenses/$pkgname/LICENSE
  install -Dm644 README.md "$pkgdir"/usr/share/doc/$pkgname/README.md
}


_buildpng() {
  cd $_pngdir
  ./configure --build=$CBUILD --host=$CHOST --prefix=/usr --enable-shared --with-libpng-compat || return 1
  make || return 1
  # TODO we ideally don't want to be including libpng12 with our package, but there isn't a clean way around it right now
  make DESTDIR="$pkgdir" install || return 1
}
md5sums="4fa4a4aa4fd5fe7f8232aba39d53169e  mod-pagespeed-1.10.33.7.tar.bz2
76eb5853a1190e0cfc691aa21c545de3  nginx-1.9.11.tar.gz
9508fc59d10a1ffadd9aae35116c19ee  libpng-1.2.56.tar.gz
0313bea9d1edeaad336855e16e34e306  ngx_pagespeed-1.10.33.7.tar.gz
ce007d26a1fcf15bbbaae8d4583e751e  pthread_nonrecursive_np.patch
ecfa57c89df68167231f35e73fc49230  stack_trace_posix.patch
c424e4619cd186f11c033c4209da116f  libpng_cflags.patch
2d4ac7f331a8d9aa22889e50dabc53d5  automatic_makefile.patch
90a5918f86ebb7855a807d9cc12fbdb7  rename_c_symbols.patch"
sha256sums="7d1c572d696a237b746423f6702ed2933f40549dfe2d3f6e7354d461756b5849  mod-pagespeed-1.10.33.7.tar.bz2
6a5c72f4afaf57a6db064bba0965d72335f127481c5d4e64ee8714e7b368a51f  nginx-1.9.11.tar.gz
0366f932883084a6972efa8eadfaa5584e94a7c9c8682a49590759951518801f  libpng-1.2.56.tar.gz
09fb9826d4199a3f88645bd4e4b02e51364f0d719b221dcf7b0f680f2d9240fe  ngx_pagespeed-1.10.33.7.tar.gz
d72e4cc7a9e3b666bcd21116d82f21af180f8c1a232c2c577daf30dc5ab24bb1  pthread_nonrecursive_np.patch
79a02c4fddc874e07f56f2621c06054eec64aaacea7dafcd0f6a7fa91c2b698c  stack_trace_posix.patch
f2681cebcd17f9673fafa14b50f33afca8e9835a2a7122893de66bbcf7dd6955  libpng_cflags.patch
23fe6a32e478ccd13bca805fcb49ac4835ff9d1444319bffaad92e2697ef2ed9  automatic_makefile.patch
2ae97d5a255f36c0f6bc007e8ca552be8ed4bd6d17a15f5b20ee5dc872652cf3  rename_c_symbols.patch"
sha512sums="38e616fbb10e961f1356c4c2fa41866c3fe357f9846db7dc55aa05edda23dce308c628038b7ebdccf082136d37bb63027bad0a99ad34291a7774806061bba510  mod-pagespeed-1.10.33.7.tar.bz2
1ea79b8ade066faa4facdb13631b97c3228cc91f512dc98c7b1b153a489936f2bc13017ea67097cd303581720fd9bc3a212744ac0a03a1ea514e56321b407caa  nginx-1.9.11.tar.gz
f29082281fb25184cca7897635557bc281e4f4dc91e7bf7a7458d14e8684c9c839ec24b88144bcdaeec2d0b0a0d7c11787f2cd41a5a3fc70523801125b330e28  libpng-1.2.56.tar.gz
024fe41f0237a29b6a2411257136c4375a23fd6b447eae23f5358b871023aef64af251925ab21692a1d73d0671b6180a6b6d372f0b1df96d10abf0c4abcdcf98  ngx_pagespeed-1.10.33.7.tar.gz
f1ad8e682872a52c90dcbfc7947e3156f787670d3ea817e48bf06132e0929ae4ac5eb0f1661b2ba4b85d39d15eadb7d9cc2adb311299b7feb6924ae3536e4d20  pthread_nonrecursive_np.patch
e0992c815d2447263b78180195bf51da8cd766ab1dc8791ec6e586b88961b62a0919fbce2c04a25b33018f6ac433361944711fabdda6b740f4cde174906d39d7  stack_trace_posix.patch
ad6ca92f3f635b3c55cbcc011b1dd4e62a3ce43dc9ee39a0d4e2718f60e77541d74843d2561fd46d0048e2d8967744909c8c4e6369aafe8463e0301f3e87fc76  libpng_cflags.patch
9c66f2b758795a520b232510366533bb7e0cfb748a5d8e0ffcea73f6908b1f55b42cad63b1a0e6a4e535775d69f89d3427e0c9f7100e5a3b0053e9c12d2cd36d  automatic_makefile.patch
c3ef1558264d16f31a5b29d16dca85cd862154dbd653d2bc90f82bcc7ae41d85d921fefa3765242aa656b6ae50edbdcb80b607f23af109f8ffc7be683bda2c5a  rename_c_symbols.patch"
