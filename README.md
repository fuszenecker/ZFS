# ZFS

Set the L2ARC size to a size that make sense on your hardware, edit `/etc/modprobe.d/zfs.conf`:

```
options zfs zfs_arc_max=536870912
```

This is 512M on my 4G Raspberry Pi.

Enable few features (when e.g. creating a ZFS pool) in order to do some performance increase:

```
zpool create \
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

Compatibility:

```
zpool create -o compatibility=grub2 bootpool ${DISK}
```

See `/usr/local/share/zfs/compatibility.d/` or `/usr/share/zfs/compatibility.d`.

All features disabled:

```
sudo zpool create \
    -R /media -m /zfs  \
    -o listsnapshots=on \
    -o autoexpand=off \
    -o ashift=12 \
    -o autotrim=off \
    -o feature@draid=disabled \
    -o feature@device_rebuild=disabled \
    -o feature@livelist=disabled \
    -o feature@log_spacemap=disabled \
    -o feature@bookmark_written=disabled \
    -o feature@redacted_datasets=disabled \
    -o feature@redaction_bookmarks=disabled \
    -o feature@bookmark_v2=disabled \
    -o feature@resilver_defer=disabled \
    -o feature@allocation_classes=disabled \
    -o feature@spacemap_v2=disabled \
    -o feature@zpool_checkpoint=disabled \
    -o feature@obsolete_counts=disabled \
    -o feature@device_removal=disabled \
    -o feature@project_quota=disabled \
    -o feature@encryption=disabled \
    -o feature@userobj_accounting=disabled \
    -o feature@edonr=disabled \
    -o feature@skein=disabled \
    -o feature@sha512=disabled \
    -o feature@large_dnode=disabled \
    -o feature@large_blocks=disabled \
    -o feature@filesystem_limits=disabled \
    -o feature@bookmarks=disabled \
    -o feature@embedded_data=disabled \
    -o feature@extensible_dataset=disabled \
    -o feature@hole_birth=disabled \
    -o feature@enabled_txg=disabled \
    -o feature@spacemap_histogram=disabled \
    -o feature@multi_vdev_crash_dump=disabled \
    -o feature@empty_bpobj=disabled \
    -o feature@async_destroy=disabled \
    -o feature@zstd_compress=enabled \
    -o feature@lz4_compress=disabled \
    -O compression=zstd \
    -O acltype=posix \
    -O atime=on \
    -O relatime=on \
    -O xattr=sa \
    -O normalization=formD \
    zfspool ${DISK}
```

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
