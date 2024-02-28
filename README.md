#### Tested on physical machines

- rpi3b+ -->> SERIAL_NUMBER=76d2a334
- rpi4b  -->> SERIAL_NUMBER=ac8854ec

Using a Dell wireless laptop to serve tftp and nfs
#### Forwards ...

Go to https://www.raspberrypi.com/software/operating-systems/ and dowload extract your image.

Run these commands ...

```
sudo kpartx -va 2023-12-11-raspios-bookworm-arm64-lite.img

sudo mkdir -v /pxe

sudo mount -v /dev/mapper/loop0p2 /pxe
sudo mount -v /dev/mapper/loop0p1 /pxe/boot/firmware

sudo mkdir -v /srv/tftp/$SERIAL_NUMBER

sudo mount -v -o bind /pxe/boot/firmware /srv/tftp/$SERIAL_NUMBER

sudo systemctl daemon-reload # only needed when changing config

sudo systemctl start dnsmasq

sudo exportfs -a
```

In the meantime, run on another screen to watch if the system tries to download files. If it works, then you will see failure messages, but then success messages. It takes a few minutes over a wifi nfs mount, he he.

```
journalctl -fex
```

If it succeeds on the server machine, then you will see message on the client machine where the control is transfered. 

Depending on your configuration, dnsmasq won't be needed anymore.


> [!NOTE]
> Make sure to avoid any confusion and make backups of these files before working on live systems : 
- /etc/fstab 
- /boot/cmdline.txt 
- /etc/exports 
- /etc/dnsmasq.conf

> Edit /etc/fstab /boot/cmdline.txt /etc/exports & /etc/dnsmasq.conf

**/boot/cmdline.txt:**

```
CMDLINE="console=serial0,115200 console=tty1 root=/dev/nfs nfsroot=192.168.0.206:/pxe,vers=3 rw ip=dhcp rootwait elevator=deadline"
```

**/etc/exports:**
 
```/srv/tftp *(rw,sync,no_subtree_check,no_root_squash)
/pxe/boot/firmware *(rw,sync,no_subtree_check,no_root_squash)
/pxe *(rw,sync,no_subtree_check,no_root_squash)
```

**/etc/fstab :**

```FSTAB="192.168.0.206:/pxe / nfs defaults,vers=3 0 0
192.168.0.206:/pxe/boot/firmware /boot/firmware nfs defaults,vers=3 0 0"
```

**/etc/dnsmasq.conf :**

```
interface=*
dhcp-range=192.168.0.10,192.168.0.50,12h
log-dhcp
enable-tftp
tftp-root=/srv/tftp
pxe-service=0,"Raspberry Pi Boot"
```



#### Backwards ...
```
sudo systemctl stop dnsmasq
sudo umount -v /srv/tftp/$SERIAL_NUMBER
sudo umount -v /pxe/boot/firmware
sudo umount-v  /pxe

sudo rmdir -v /pxe

sudo kpartx -vd 2023-12-11-raspios-bookworm-arm64-lite.img
```