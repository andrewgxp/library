# Expanding LVM Storage on Linux

After increasing the disk size of a Virtual Machine at the hypervisor
level (VMware, Proxmox, etc.), the operating system does **not**
automatically use the new space.

To make the additional storage usable, you must expand each of the
following layers:

Disk → Partition → Physical Volume → Volume Group → Logical Volume →
Filesystem

In this example, we increase an Ubuntu VM from **20GB to 100GB** and
expand the root filesystem to use all available space.

------------------------------------------------------------------------

## Initial Situation

After increasing the VM disk size in VMware to **100GB**, the OS still
shows:

-   Disk (`/dev/sda`) = 100G
-   Partition (`/dev/sda3`) still small
-   Logical Volume (`/dev/ubuntu-vg/ubuntu-lv`) still 19G
-   Root filesystem (`/`) still 19G

Before expansion, `df -h` looks something like this:

``` bash
user@server:~$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              387M  5.4M  382M   2% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   19G   12G  6.1G  66% /
tmpfs                              1.9G     0  1.9G   0% /dev/shm
/dev/sda2                          2.0G  192M  1.6G  11% /boot
```

Even though the disk was expanded, the root filesystem is still only
19G.

We need to expand:

1.  The partition
2.  The LVM Physical Volume (PV)
3.  The Logical Volume (LV)
4.  The filesystem

------------------------------------------------------------------------

# Step 1 --- Verify the Disk Was Expanded

Check the block devices:

``` bash
lsblk
```

Example output:

``` bash
user@server:~$ lsblk
NAME                      SIZE TYPE MOUNTPOINTS
sda                       100G disk
├─sda1                      1M part
├─sda2                      2G part /boot
└─sda3                     38G part
  └─ubuntu--vg-ubuntu--lv  19G lvm  /
```

Notice:

-   `/dev/sda` is now 100G\
-   `/dev/sda3` is still 38G\
-   The LVM logical volume is still 19G

The disk is bigger, but the partition is not.

------------------------------------------------------------------------

# Step 2 --- Expand the Partition

Install `growpart` if it is not already installed:

``` bash
sudo apt install cloud-guest-utils
```

Expand partition 3:

``` bash
sudo growpart /dev/sda 3
```

You should see output similar to:

``` bash
CHANGED: partition=3 start=4198400 old: size=79685632 end=83884031 new: size=205516767 end=209715166
```

Verify the partition grew:

``` bash
lsblk
```

Now `/dev/sda3` should show \~98G:

``` bash
user@server:~$ lsblk
NAME                      SIZE TYPE MOUNTPOINTS
sda                       100G disk
├─sda1                      1M part
├─sda2                      2G part /boot
└─sda3                     98G part
  └─ubuntu--vg-ubuntu--lv  19G lvm  /
```

The partition is now expanded, but LVM still thinks it's smaller
internally.

------------------------------------------------------------------------

# Step 3 --- Resize the LVM Physical Volume

Tell LVM that the underlying partition has grown:

``` bash
sudo pvresize /dev/sda3
```

Example output:

``` bash
Physical volume "/dev/sda3" changed
1 physical volume(s) resized or updated / 0 physical volume(s) not resized
```

Verify free space in the Volume Group:

``` bash
sudo pvs
```

Example:

``` bash
PV         VG        Fmt  Attr PSize   PFree
/dev/sda3  ubuntu-vg lvm2 a--  <98.00g 79.00g
```

`PFree` shows how much unused space is available inside the Volume
Group.

------------------------------------------------------------------------

# Step 4 --- Extend the Logical Volume

Use all available free space:

``` bash
sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
```

Example output:

``` bash
Size of logical volume ubuntu-vg/ubuntu-lv changed from <19.00 GiB to <98.00 GiB.
Logical volume ubuntu-vg/ubuntu-lv successfully resized.
```

At this point, the Logical Volume is larger --- but the filesystem is
still 19G.

------------------------------------------------------------------------

# Step 5 --- Resize the Filesystem

For **ext4** filesystems:

``` bash
sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
```

Example output:

``` bash
resize2fs 1.47.0
Filesystem at /dev/ubuntu-vg/ubuntu-lv is mounted on /; on-line resizing required
The filesystem on /dev/ubuntu-vg/ubuntu-lv is now 25689088 (4k) blocks long.
```

If using XFS instead of ext4:

``` bash
sudo xfs_growfs /
```

------------------------------------------------------------------------

# Step 6 --- Verify the Final Result

``` bash
df -h
```

Example:

``` bash
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   97G   12G   81G  13% /
```

The root filesystem now reflects the expanded size.

------------------------------------------------------------------------

# Notes

-   This procedure assumes:
    -   Linux with an LVM-managed root filesystem\
    -   ext4 (or XFS) filesystem\
    -   Hypervisor disk expansion already completed\
-   Always ensure backups exist before modifying partitions or LVM.\
-   Adjust device paths as needed for your environment (NVMe devices,
    different VG names, etc.).

------------------------------------------------------------------------

## Summary of Commands

``` bash
lsblk
sudo apt install cloud-guest-utils
sudo growpart /dev/sda 3
lsblk
sudo pvresize /dev/sda3
sudo pvs
sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
df -h
```
