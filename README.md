# ZFS

## Install

On Ubuntu:

```
sudo apt install zfsutils-linux
```

## Memory usage

### Calculation

```
MAX = (RAM SIZE - 1 GB) * 3/4
MIN = MAX / 32
```

In practice: 62.5 MB ... 2 GB. Average: `RAM SIZE / 8 = 512 MB`-

### Setup 

Set the ARC size to a size that make sense on your hardware, edit `/etc/modprobe.d/zfs.conf`:

```
options zfs zfs_arc_max=536870912
```

or

```
echo "536870912" > /sys/module/zfs/parameters/zfs_arc_max
```

This is 512M on my 4G Raspberry Pi. 256 M is `268435456`.

### Statistics

```
arc_summary -s arc
arc_summary -s archits
```

## Creating pool and filesystem

Enable few features (when e.g. creating a ZFS pool) in order to do some performance increase:

```
zpool create \
    -o compatibility=openzfs-2.0-freebsd \
    -o ashift=12 \
    -o autotrim=off \
    -O compression=lz4 \
    -O acltype=posix \
    -O atime=off \
    -O relatime=on \
    -O xattr=sa \
    -O normalization=formD \
    -m /media/zfspool \
    zfspool ${DISK}
```

Compatibility: see `/usr/local/share/zfs/compatibility.d/` or `/usr/share/zfs/compatibility.d`. Good candidates are:

* grub2: for **boot** pools, when the kernel and initrd is placed on this file system
* openzfs-2.0-linux: ubuntu-20.04 + ZSTD
* openzfs-2.1-linux: ubuntu-22.04 = ubuntu-20.04 + DRAID + ZSTD

## Importing with right names

```
zpool import -d /dev/disk/by-partlabel
```

```
zfs create zfspool/zfs1
```

Trimming:

```
zpool trim -w zfspool
```

## Starting services with systemd

On newer systems:

```
systemctl enable zfs-import-cache zfs-import-scan zfs-import.target zfs-mount zfs-share zfs-volume-wait zfs-volumes.target zfs-zed zfs.target
systemctl start zfs-import-cache zfs-import-scan zfs-import.target zfs-mount zfs-share zfs-volume-wait zfs-volumes.target zfs-zed zfs.target
```

## Encryption

Creating a filesystem:

```
zfs create -o encryption=aes-256-gcm -o keylocation=prompt -o keyformat=passphrase zfspool/encrypted
```

Loading the key and mounting the filesystem:

```
zpool import -l zfspool
```

or manually:

```
zfs load-key -r zfspool/encrypted
zfs get keystatus zfspool/encrypted
zfs mount zfspool/encrypted
```

## Sharing with NFS

Enable NFS share on your ZFS filesystem (service principal for NFS server: `nfs/${server}@${REALM}`):

```
zfs set sharenfs="rw=192.168.100.204,rw=rpi4.local" zfspool/shared
zfs set sharenfs="rw=@192.168.100.0/24,sync,sec=krb5:krb5i:krb5p,root_squash,no_subtree_check" zfspool/shared
```

or 

```
zfs set sharenfs=on zfspool/shared
exportfs -o sync,sec=krb5:krb5i:krb5p,root_squash,no_subtree_check 192.168.100.0/24:/media/zfs/shared
```

Check if the share has been exported:

```
exportfs -s
showmount -e
```

## DKMS

If you have a root ZFS:

```
yay -S zfs-dkms-git zfs-utils-git
```

If you don't have a root ZFS:

```
yay -S zfs-dkms zfs-utils
```

Alternatively, you can setup DKMS manually, however, it is not recommended.

```
cd /home/fuszenecker/zfs
sudo ln -s /home/fuszenecker/zfs /usr/src/zfs-2.1.3
scripts/dkms.mkconf -n zfs -v 2.1.3 -f dkms.conf
sudo dkms add -m zfs -v 2.1.3
sudo dkms build -m zfs -v 2.1.3
sudo dkms install -m zfs -v 2.1.3
sudo dkms remove -m zfs -v 2.1.3 --all
sudo dkms status
```

## Trim and Timers

```
sudo systemctl edit fstrim.timer
```

You might want to add:

```
[Unit]
Description=Discard unused blocks once a day

[Timer]
OnCalendar=Daily
```

But it is more important to add the command to trigger trim:

```
sudo systemctl edit fstrim
```

By adding:

```
[Service]
ExecStart=/usr/bin/zpool trim zfspool
```
