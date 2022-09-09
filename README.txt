FQDN=demo
IP_OF_FQDN=`dig $FQDN +short`

DIR=/srv/live-build-rescue-`date +"%Y%m%d%H%M%S"`
mkdir $DIR
cd $DIR
lb config --debootstrap-options "--exclude=nano,iptables,dmidecode,info" --apt-source-archives false --cache false --chroot-filesystem squashfs --apt-indices false --cache-packages false --config https://github.com/gratuxri/live-build-rescue.git::master --archive-areas "main contrib non-free" --apt-recommends false --firmware-chroot false --firmware-binary false --updates true --backports false --win32-loader false --loadlin false --security true  --initsystem systemd  -b netboot -a amd64 -k amd64 --linux-packages linux-image --bootappend-live vconsole.keymap=de-latin1-nodeadkeys log_buf_len=1M quickreboot silent splash lang=de locales=de_DE.UTF-8 keyboard-layouts=de consoleblank=0 quiet kernel.sysrq=1 keep_bootcon sysrq_always_enabled toram live-config live-config.timezone=Europe/Berlin 
lb build
cd `dirname $DIR`
ln -s `basename $DIR` lb-rescue
cd /var/www/html
ln -s /srv/lb-rescue/binary/live .
cat <<EOF>myhw
#!ipxe
dhcp
kernel http://$FQDN/live/vmlinuz boot=live components fetch=http://$IP_OF_FQDN/live/filesystem.squashfs noswap vconsole.keymap=de-latin1-nodeadkeys log_buf_len=1M quickreboot silent splash toram lang=de locales=de_DE.UTF-8 keyboard-layouts=de consoleblank=0 quiet kernel.sysrq=1 keep_bootcon sysrq_always_enabled initrd=initrd.img rootwait=120 live-config
initrd http://$FQDN/live/initrd.img
boot
EOF
