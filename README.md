# ZFS

## Memory usage

Set the L2ARC size to a size that make sense on your hardware, edit `/etc/modprobe.d/zfs.conf`:

```
options zfs zfs_arc_max=536870912
```

This is 512M on my 4G Raspberry Pi.

## Creating pool and filesystem

Enable few features (when e.g. creating a ZFS pool) in order to do some performance increase:

```
zpool create \
    -o compatibility=openzfs-2.1-linux \
    -o ashift=12 \
    -o autotrim=on \
    -O compression=zstd \
    -O acltype=posix \
    -O atime=on \
    -O relatime=on \
    -O xattr=sa \
    -O normalization=formD \
    -m /media/zfspool
    zfspool ${DISK}
```

Compatibility: see `/usr/local/share/zfs/compatibility.d/` or `/usr/share/zfs/compatibility.d`. Good candidates are:

* grub2 (for root pools)
* zol-0.8 (= ubuntu-20.04)
* openzfs-2.0-linux (everything, except DRAID)
* openzfs-2.1-linux (everything)

## Importing with right names

```
zpool import -d /dev/disk/by-partlabel
```

## Starting services with systemd

On newer systems:

```
systemctl enable zfs.target zfs-import.service zfs-mount.service
systemctl start zfs.target zfs-import.service zfs-mount.service
```

On older systems, enable ZFS services:

```
systemctl enable zfs-import-cache
systemctl enable zfs-import-scan
systemctl enable zfs-import.target
systemctl enable zfs-mount
systemctl enable zfs-share
systemctl enable zfs-zed
systemctl enable zfs.target
```

## Encryption

Creating a filesystem:

```
zfs create -o encryption=aes-256-gcm -o keylocation=prompt -o keyformat=passphrase zfspool/encrypted
```

Loading the key and mounting the filesystem:

```
zfs load-key -r zfspool/encrypted
zfs get keystatus zfspool/encrypted
zfs mount zfspool/encrypted
```

## Sharing with NFS

Enable NFS share on your ZFS filesystem:

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
