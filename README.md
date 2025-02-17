# ZFS

## Install

On Ubuntu:

```
sudo apt install zfsutils-linux zfs-initramfs
```

## Memory usage

### Calculation

```
MAX = (RAM SIZE - 1 GB) * 3/4
MIN = MAX / 32
```

In practice: 62.5 MB ... 2 GB. Average: `RAM SIZE / 8 = 512 MB`.

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
    -o compatibility=openzfs-2.2 \
    -o ashift=12 \
    -o autotrim=off \
    -O compression=lz4 \
    -O acltype=posix \
    -O atime=off \
    -O relatime=off \
    -O xattr=sa \
    -O normalization=formD \
    -O listsnapshots=on \
    -m /media/zfspool \
    zfspool ${DISK}
```

Compatibility: see `/usr/local/share/zfs/compatibility.d/` or `/usr/share/zfs/compatibility.d`. Good candidates are:

* `grub2`: for **boot** pools, when the kernel and initrd is placed on this file system (not recommended)
* `openzfs-2.0-linux`: `ubuntu-20.04` + `zstd_compress`
* `openzfs-2.1-linux` = `ubuntu-22.04`: `openzfs-2.0-linux` + `draid`
* `openzfs-2.2-linux`: `openzfs-2.1-linux` + `blake3` + `block_cloning`
* `openzfs-2.3-linux`: `openzfs-2.2-linux` + `fast_dedup` + `longname` + `raidz_expansion`

## Importing with right names

```
zpool import -d /dev/disk/by-partlabel
```

```
zfs create zfspool/zfs1
```

## Starting services with systemd

On newer systems:

```
systemctl enable zfs-import-cache.service zfs-load-module.service zfs-mount.service zfs-share.service zfs-volume-wait.service zfs-zed.service zfs-import.target zfs-volumes.target zfs.target
systemctl start zfs-import-cache.service zfs-load-module.service zfs-mount.service zfs-share.service zfs-volume-wait.service zfs-zed.service zfs-import.target zfs-volumes.target zfs.target
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

## ZFS root

Kernel command line:

```
... root=ZFS=zfsroot rootfstype=zfs rootwait
```

`fatab`:

```
zfsroot    /    zfs    discard    0 1
```
