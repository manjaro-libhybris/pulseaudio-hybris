pkgname="pulseaudio-hybris"
pkgdesc="A featureful, general-purpose sound server"
pkgver=14.2
pkgrel=2
arch=("aarch64" "i686" "x86_64" "armv7h")
url="http://pulseaudio.org/"
license=("GPL" "LGPL")
install=pulseaudio.install
depends=(lib{ltdl,soxr,asyncns,xtst,sndfile} "rtkit" "speexdsp" "tdb" "orc"
         "webrtc-audio-processing" jack2 "lirc" bluez{,-libs} "sbc"
         python-{pyqt5,dbus,pyqt5-sip} "fftw" dconf libwrap gst-plugins-base-libs)
makedepends=("git" lib{asyncns,xtst,tool,soxr,sndfile} "attr" "rtkit" "speexdsp"
             "tdb" jack2 bluez{,-libs} "intltool"  "sbc" "lirc" "fftw"
             "orc" "gtk3" "webrtc-audio-processing" "check" "meson" "valgrind"
             "libwrap" "doxygen")
backup=(etc/pulse/{daemon.conf,default.pa,system.pa,client.conf})
provides=(pulseaudio{,-{zeroconf,lirc,jack,bluetooth,equalizer}} libpulse  libpulse{,-{simple,mainloop-glib}}.so)
conflicts=(pulseaudio-zeroconf pulseaudio-lirc pulse-audio-jack pulseaudio-bluetooth pulseaudio-equalizer libpulse{,-{simple,mainloop-glib}}.so pipewire-pulse)
options=(!emptydirs)
source=("git+https://github.com/droidian/pulseaudio"
        "pulseaudio.install")
sha256sums=('SKIP'
            '1d4890b10fadb9208c3fefbbed4aca1f22e63a0f102f4c598dc573a55e724cb2')

prepare() {
  mv pulseaudio pulseaudio-hybris
  cd $pkgbase
  git checkout -b bookworm
  git checkout bc1a4be445b88c91442c5b76a5f3389170984ac9
}

build() {
    arch-meson pulseaudio-hybris build \
    -D stream-restore-clear-old-devices=true \
    -D pulsedsp-location='/usr/\$LIB/pulseaudio' \
    -D udevrulesdir=/usr/lib/udev/rules.d \
    -D orc=disabled

  ninja -C build
}

check() {
  meson test -C build --print-errorlogs
  ninja -C build test-daemon
}
 
package() {    

    DESTDIR="$pkgdir" meson install -C build

    cd "$pkgdir"

    # Assumes that any volume adjustment is intended by the user, who can control
    # each app's volume. Misbehaving clients can trigger earsplitting volume
    # jumps. App volumes can diverge wildly and cause apps without their own
    # volume control to fall below sink volume; a sink-only volume control will
    # suddenly be unable to make such an app loud enough.
    sed -e '/flat-volumes/iflat-volumes = no' \
        -i etc/pulse/daemon.conf

    # Superseded by socket activation
    sed -e '/autospawn/iautospawn = no' \
        -i etc/pulse/client.conf

    # Disable cork-request module, can result in e.g. media players unpausing
    # when there's a Skype call incoming
    sed -e 's|/usr/bin/pactl load-module module-x11-cork-request|#&|' \
    -i usr/bin/start-pulseaudio-x11

    # Required by qpaeq
    sed -e '/Load several protocols/aload-module module-dbus-protocol' \
        -i etc/pulse/default.pa

    rm -r etc/dbus-1
}
