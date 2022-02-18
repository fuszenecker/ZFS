# ZFS

Set the L2ARC size to a size that make sense on your hardware, edit `/etc/modprobe.d/zfs.conf`:

```
options zfs zfs_arc_max=536870912
```

This is 512M on my 4G Raspberry Pi.

Enable few features (when e.g. creating a ZFS pool) in order to do some performance increase:

```
zpool create \
    -o compatibility=openzfs-2.1-linux
    -o ashift=12 
    -o autotrim=on \
    -O compression=zstd \
    -O acltype=posix \
    -O atime=on \
    -O relatime=on \
    -O xattr=sa \
    -O normalization=formD \
    zfspool ${DISK}
```

Compatibility: see `/usr/local/share/zfs/compatibility.d/` or `/usr/share/zfs/compatibility.d`.

On older systems, enable ZFS services:

```
sudo systemctl enable zfs-import-cache
sudo systemctl enable zfs-import-scan
sudo systemctl enable zfs-import.target
sudo systemctl enable zfs-mount
sudo systemctl enable zfs-share
sudo systemctl enable zfs-zed
sudo systemctl enable zfs.target
```

On newer systems:

```
sudo systemctl enable zfs.target zfs-import.service zfs-mount.service
```

Enable NFS share on your ZFS filesystem:

```
zfs set sharenfs="rw=192.168.100.204,rw=rpi4.local" zfs/db
```

Check if the share has been exported:

```
fuszenecker@rpi4:/media/zfsd/db $ showmount -e
Export list for rpi4.local:
/media/zfs/db 192.168.100.204
```
