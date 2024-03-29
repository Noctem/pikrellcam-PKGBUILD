# Maintainer: David Christenson

_pkgname=pikrellcam
pkgname="${_pkgname}-git"
pkgver=4.3.r137.08ed792
pkgrel=1
pkgdesc="Raspberry Pi motion vector detection program with OSD web interface"
arch=('armv6h' 'armv7h' 'aarch64')
license=('GPL3')
install="${pkgname}.install"
url="https://billw2.github.io/pikrellcam/pikrellcam.html"
makedepends=('git' 'make' 'patch' 'coreutils')
depends=('nginx' 'ffmpeg' 'php-fpm' 'gpac' 'imagemagick' 'lame' 'raspberrypi-firmware' 'alsa-lib' 'sudo')
optdepends=('mpack: for emailing previews'
            'python: for some of the example scripts'
            'servoblaster-git: for controlling servos')
provides=('pikrellcam')
conflicts=('pikrellcam')
backup=('etc/nginx/sites-available/pikrellcam'
        'srv/http/pikrellcam/www/.htpasswd')
source=('git+https://github.com/billw2/pikrellcam'
        '50-vcsm.rules'
        'conformance.patch'
        'htpasswd'
        'pikrellcam.nginx'
        'pikrellcam.service'
        'pikrellcam.sysuser'
        'pikrellcam.tmp'
        'user.php')
sha256sums=('SKIP'
            'cf165a161848cda32e89cfd48722b64d19cca2f58ade93a5c9c59a23f7b1521b'
            '954b37fd5e6302875a087e2d54f01e513c755aa6c24d26be8b6dc8d6d72d6509'
            'e91161b96d890b37f5970c4acfd99f1f62991d1cb1c018e1f4c267ce185ef4b4'
            'c1ec8f8d25b7ee2a1770f9ded6cf7089454d96c505e76b25c285125a4c536c3a'
            'aa04ff7e986749beae3a7ffbc0c03a449fec9ee6e40c4e53c57e8b263a2629d6'
            'e92ef70d2b1b697fe7add6abb727ec692e341589bfe5d3173b4b855328e37495'
            '24f7bf8f483c130b4b9c26e27566a22901cfe8707ca705343bd871a502fbdaaa'
            '4307747339b96bdd1b41e50abb4b60c4212d4c6a1216cca740cb83605d932ccf')

pkgver() {
  cd "${srcdir}/${_pkgname}/src"
  local _ver=$(grep -oP '(?<=#define\tPIKRELLCAM_VERSION\t")\d+\.\d+' pikrellcam.h)
  [[ -n "$_ver" ]] || _ver='4.3'
  printf '%s.r%s.%s' "$_ver" $(git rev-list --count HEAD) $(git rev-parse --short HEAD)
}

prepare() {
  cd "${srcdir}/${_pkgname}"
  rm -f etc/nginx* install-pikrellcam.sh pikrellcam scripts-dist/_upgrade*
  patch -p1 -i ../conformance.patch

  local _ver=$(grep -oP '(?<=#define\tPIKRELLCAM_VERSION\t")\d+\.\d+.\d+' src/pikrellcam.h)
  if ! fgrep -q $_ver www/config.php; then
    sed -i "/VERSION/c\	define\(\"VERSION\", \"$_ver\"\);" www/config.php
  fi
}

build() {
  cd "${srcdir}/${_pkgname}/src"
  make
}

package(){
  cd "${srcdir}/${_pkgname}"
  local _dst="${pkgdir}/var/lib/pikrellcam"
  local _srv="${pkgdir}/srv/http/pikrellcam"

  # create sysuser
  install -Dm644 ../"${_pkgname}.sysuser" "${pkgdir}/usr/lib/sysusers.d/pikrellcam.conf"

  install -dm755 "${_dst}/config"

  # create web directories and symlinks
  install -dm775 "${_srv}/www"
  install -dm755 -g http "$_srv"/{www/images,www/js-css}
  install -dm775 -g http "$_srv"/{media,media/archive,media/loop,media/stills,media/thumbs,media/videos}
  ln -s ../media "$_srv"/www/media
  ln -s ../media/archive "$_srv"/www/archive
  ln -s ../media/loop "$_srv"/www/loop

  # install web files
  install -Dm640 -g http ../htpasswd "${_srv}/www/.htpasswd"

  # install binary
  install -Dm755 src/build/pikrellcam "${pkgdir}/usr/bin/pikrellcam"

  # install systemd service
  install -Dm644 "../${_pkgname}.service" "${pkgdir}/usr/lib/systemd/system/${_pkgname}.service"

  # install udev rule for vcsm
  install -Dm644 "../50-vcsm.rules" "${pkgdir}/etc/udev/rules.d/50-vcsm.rules"

  # install sudoers entry
  install -dm750 "${pkgdir}/etc/sudoers.d"
  install -Dm440 etc/pikrellcam.sudoers "${pkgdir}/etc/sudoers.d/pikrellcam"

  # install tmpfiles.d entry
  install -Dm644 "../${_pkgname}.tmp" "${pkgdir}/usr/lib/tmpfiles.d/${_pkgname}.conf"

  # install nginx conf
  install -dm755 "${pkgdir}/etc/nginx/sites-enabled"
  install -Dm644 "../${_pkgname}.nginx" "${pkgdir}/etc/nginx/sites-available/pikrellcam"
  ln -s ../sites-available/pikrellcam "${pkgdir}/etc/nginx/sites-enabled/pikrellcam.conf"

  # install example scripts (will be copied to scripts by install script)
  install -dm755 "${_dst}/scripts-dist"
  install -Dm755 scripts-dist/* "${_dst}/scripts-dist/"
  chmod 644 "${_dst}/scripts-dist/Readme"
  install -dm755 "${_dst}/scripts"

  # copy web files
  for file in www/*; do
    [[ -f "$file" ]] && install -Dm644 -g http "$file" "${_srv}/www/"
  done
  install -Dm644 -g http www/images/* "${_srv}/www/images/"
  install -Dm644 -g http www/js-css/* "${_srv}/www/js-css/"
}
