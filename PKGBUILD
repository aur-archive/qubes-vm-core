# This is an example PKGBUILD file. Use this as a start to creating your own,
# and remove these comments. For more information, see 'man PKGBUILD'.
# NOTE: Please fill out the license field for your package! If it is unknown,
# then please put 'unknown'.

# Maintainer: Your Name <youremail@domain.com>
pkgname=qubes-vm-core
pkgver=2.1.13
pkgrel=1
epoch=
pkgdesc=""
arch=("x86_64")
url="http://qubes-os.org/"
license=('GPL')
groups=()
depends=(rsyslog qubes-vm-xen sudo net-tools zsh)
makedepends=(qubes-vm-xen)
checkdepends=()
optdepends=()
provides=()
conflicts=()
replaces=()
backup=()
options=()
install=qubes-vm-core.install
changelog=
source=(qubes-ensure-lib-modules.service)
noextract=()
md5sums=() #generate with 'makepkg -g'

verify() {
  version=$1

  TAG=`git tag --points-at=$version | head -n 1`
  echo "Verifying git sources for version $version..."
  git checkout $TAG
  echo "If it fails, your should download Qubes master keys, verify the fingerprint and trust it"
  echo "gpg --recv-keys 0x36879494"
  echo "gpg --list-keys --fingerprint (fingerprint can be found on http://qubes-os.org/trac/wiki/VerifyingSignatures"
  echo "gpg --edit-key 0x36879494 then trust, 5, y, q"
  echo "wget http://keys.qubes-os.org/keys/qubes-developers-keys.asc"
  echo "gpg --import qubes-developers-keys.asc"
  git tag -v $TAG
}

build() {
  echo "Downloading git sources..."
  git clone git://git.qubes-os.org/marmarek/core

  export PYTHON=python2

  cd core
  verify v$pkgver

  sed 's:python :python2 :g' -i misc/Makefile

  # Core libs
  make -C u2mfn
  make -C vchan -f Makefile.linux
  make -C qrexec
  make -C qubes_rpc
  make -C misc

}



package() {
  install -D $srcdir/qubes-ensure-lib-modules.service $pkgdir/usr/lib/systemd/system/qubes-ensure-lib-modules.service

  cd core

  # CORE-LIBS
  install -D -m 0644 vchan/libvchan.h $pkgdir/usr/include/libvchan.h
  install -D -m 0644 u2mfn/u2mfnlib.h $pkgdir/usr/include/u2mfnlib.h
  install -D -m 0644 u2mfn/u2mfn-kernel.h $pkgdir/usr/include/u2mfn-kernel.h

  install -D vchan/libvchan.so $pkgdir/usr/lib/libvchan.so
  install -D u2mfn/libu2mfn.so $pkgdir/usr/lib/libu2mfn.so

  # CORE-VM
  # TODO: Package nautilus-scripts to /home/user/

  mkdir -p $pkgdir/var/lib/qubes

#  install -m 0644 -D misc/fstab $pkgdir/etc/fstab
#  install -d $pkgdir/etc/init.d
#  install vm-init.d/* $pkgdir/etc/init.d/

  install -d $pkgdir/lib/systemd/system $pkgdir/usr/lib/qubes/init
  install -m 0755 vm-systemd/*.sh $pkgdir/usr/lib/qubes/init/
  install -m 0644 vm-systemd/qubes-*.service $pkgdir/lib/systemd/system/
  install -m 0644 vm-systemd/qubes-*.timer $pkgdir/lib/systemd/system/
  install -m 0644 vm-systemd/NetworkManager.service $pkgdir/usr/lib/qubes/init/
  install -m 0644 vm-systemd/cups.service $pkgdir/usr/lib/qubes/init/
  install -m 0644 vm-systemd/ntpd.service $pkgdir/usr/lib/qubes/init/

  install -D -m 0440 misc/qubes.sudoers $pkgdir/etc/sudoers.d/qubes
#  install -F -m 0644 misc/qubes.repo
  install -D -m 0644 misc/serial.conf $pkgdir/usr/lib/qubes/serial.conf
  install -D misc/qubes_serial_login $pkgdir/sbin/qubes_serial_login
  install -d $pkgdir/usr/share/glib-2.0/schemas/
  install -m 0644 misc/org.gnome.settings-daemon.plugins.updates.gschema.override $pkgdir/usr/share/glib-2.0/schemas/
#  install -d $pkgdir/usr/lib/yum-plugins
#  install -m 0644 misc/yum-qubes-hooks.py* $RPM_BUILD_ROOT/usr/lib/yum-plugins/
#  install -D -m 0644 misc/yum-qubes-hooks.conf $RPM_BUILD_ROOT/etc/yum/pluginconf.d/yum-qubes-hooks.conf

  install -d $pkgdir/var/lib/qubes

  install -d $pkgdir/etc/pki/rpm-pgp
  install -m 644 misc/RPM-GPG-KEY-qubes* $pkgdir/etc/pki/rpm-pgp/
  install -D misc/xenstore-watch $pkgdir/usr/bin/xenstore-watch-qubes
  install -d $pkgdir/etc/udev/rules.d
  install -m 0644 misc/qubes_misc.rules $pkgdir/etc/udev/rules.d/50-qubes_misc.rules
  install -m 0644 misc/qubes_block.rules $pkgdir/etc/udev/rules.d/99-qubes_block.rules
  install -m 0644 misc/qubes_usb.rules $pkgdir/etc/udev/rules.d/99-qubes_usb.rules
  install -d $pkgdir/usr/lib/qubes/

  install misc/qubes_download_dom0_updates.sh $pkgdir/usr/lib/qubes/
  install misc/{block_add_change,block_remove,block_cleanup} $pkgdir/usr/lib/qubes
  install misc/{usb_add_change,usb_remove} $pkgdir/usr/lib/qubes/
  install misc/vusb-ctl.py $pkgdir/usr/lib/qubes/
  install misc/qubes_trigger_sync_appmenus.sh $pkgdir/usr/lib/qubes/
#  install -D -m 0644 misc/qubes_trigger_sync_appmenus.action $pkgdir/etc/yum/post-actions
  mkdir -p $pkgdir/usr/lib/qubes

  if [ -r misc/dispvm-dotfiles.${pkgver}.tbz ] ; then
    install misc/dispvm-dotfiles.${pkgver}.tbz $pkgdir/etc/dispvm-dotfiles.tbz
  else
    install misc/dispvm-dotfiles.tbz $pkgdir/etc/dispvm-dotfiles.tbz
  fi
  install misc/dispvm-prerun.sh $pkgdir/usr/lib/qubes/dispvm-prerun.sh

  # Convert module loading to ARCHLINUX
  #install -D misc/qubes_core.modules $pkgdir/etc/sysconfig/modules/qubes_core.modules
  mkdir -p $pkgdir/etc/modules-load.d/
  echo xen-evtchn > $pkgdir/etc/modules-load.d/qubes_core.conf
  echo xen-blkback >> $pkgdir/etc/modules-load.d/qubes_core.conf
  # Note : need to compile pvusb drivers for this last one?
  echo xen-usbfront >> $pkgdir/etc/modules-load.d/qubes_core.conf
  #install -D misc/qubes_misc.modules $pkgdir/etc/sysconfig/modules/qubes_misc.modules
  echo dummy-hcd > $pkgdir/etc/modules-load.d/qubes_misc.conf

  # Note: appears in the gui package but required for qrexec agent to work
  echo u2mfn > $pkgdir/etc/modules-load.d/qubes_u2mfn.conf

  install -m 0644 network/qubes_network.rules $pkgdir/etc/udev/rules.d/99-qubes_network.rules
  sed 's:/sbin/ifconfig:ifconfig:g' -i network/*
  sed 's:/sbin/route:route:g' -i network/*
  sed 's:/sbin/ethtool:ethtool:g' -i network/*
  sed 's:/sbin/ip:ip:g' -i network/*
  sed 's:/bin/grep:grep:g' -i network/*
  
  install network/qubes_setup_dnat_to_ns $pkgdir/usr/lib/qubes
  install network/qubes_fix_nm_conf.sh $pkgdir/usr/lib/qubes
  install network/setup_ip $pkgdir/usr/lib/qubes/
  install network/network-manager-prepare-conf-dir $pkgdir/usr/lib/qubes/
  install -d $pkgdir/etc/dhclient.d
  ln -s /usr/lib/qubes/qubes_setup_dnat_to_ns $pkgdir/etc/dhclient.d/qubes_setup_dnat_to_ns.sh
  install -d $pkgdir/etc/NetworkManager/dispatcher.d/
  install network/{qubes_nmhook,30-qubes_external_ip} $pkgdir/etc/NetworkManager/dispatcher.d/
  install -D network/vif-route-qubes $pkgdir/etc/xen/scripts/vif-route-qubes
  install -D network/iptables $pkgdir/etc/sysconfig/iptables
  install -D network/ip6tables $pkgdir/etc/sysconfig/ip6tables
  mkdir -p $pkgdir/etc/sysconfig/
  install -m 0400 network/iptables $pkgdir/etc/sysconfig/iptables
  install -m 0400 network/ip6tables $pkgdir/etc/sysconfig/ip6tables
#  install -m 0644 network/tinyproxy-qubes-yum.conf $pkgdir/tinyproxy/tinyproxy-qubes-yum.conf
#  install -m 0644 -D network/filter-qubes-yum $RPM_BUILD_ROOT/etc/tinyproxy/filter-qubes-yum

#  install -d $RPM_BUILD_ROOT/etc/yum.conf.d
#  touch $RPM_BUILD_ROOT/etc/yum.conf.d/qubes-proxy.conf

  install -d $pkgdir/usr/sbin
  install network/qubes_firewall $pkgdir/usr/sbin
  install network/qubes_netwatcher $pkgdir/usr/sbin

  install -d $pkgdir/usr/bin

  install qubes_rpc/{qvm-open-in-dvm,qvm-open-in-vm,qvm-copy-to-vm,qvm-run,qvm-mru-entry} $pkgdir/usr/bin
  install qubes_rpc/wrap_in_html_if_url.sh $pkgdir/usr/lib/qubes
  install qubes_rpc/qvm-copy-to-vm.kde $pkgdir/usr/lib/qubes
  install qubes_rpc/qvm-copy-to-vm.gnome $pkgdir/usr/lib/qubes
  install qubes_rpc/{vm-file-editor,qfile-agent,qopen-in-vm} $pkgdir/usr/lib/qubes
  # Install qfile-unpacker as SUID - because it will fail to receive files from other vm
  install -m 4555 qubes_rpc/qfile-unpacker $pkgdir/usr/lib/qubes
  install qubes_rpc/qrun-in-vm $pkgdir/usr/lib/qubes
  install qubes_rpc/sync-ntp-clock $pkgdir/usr/lib/qubes
  install qubes_rpc/prepare-suspend $pkgdir/usr/lib/qubes
  #TODO: find where is kde_service_dir on archlinux
  #install -d $pkgdir/%{kde_service_dir}
  #install -m 0644 qubes_rpc/{qvm-copy.desktop,qvm-dvm.desktop} $pkgdir/%{kde_service_dir}
  install -d $pkgdir/etc/qubes_rpc
  install -m 0644 qubes_rpc/{qubes.Filecopy,qubes.OpenInVM,qubes.VMShell,qubes.SyncNtpClock} $pkgdir/etc/qubes_rpc
  install -m 0644 qubes_rpc/{qubes.SuspendPre,qubes.SuspendPost,qubes.GetAppmenus} $pkgdir/etc/qubes_rpc
  install -m 0644 qubes_rpc/qubes.WaitForSession $pkgdir/etc/qubes_rpc

  install qrexec/qrexec_agent $pkgdir/usr/lib/qubes
  install qrexec/qrexec_client_vm $pkgdir/usr/lib/qubes
  install qrexec/qubes_rpc_multiplexer $pkgdir/usr/lib/qubes

  install misc/meminfo-writer $pkgdir/usr/lib/qubes
  install -d $pkgdir/mnt/removable
  #install -d $pkgdir/var/lib/qubes/dom0-updates

  install -D -m 0644 misc/xorg-preload-apps.conf $pkgdir/etc/X11/xorg-preload-apps.conf

  install -d $pkgdir/var/run/qubes
  install -d $pkgdir/home_volatile/user

}

md5sums=('4b92e669d1a4ca74eb3fa9fa7b44ad6a')

# vim:set ts=2 sw=2 et:
