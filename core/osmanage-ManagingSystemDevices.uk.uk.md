<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Managing System Devices

This chapter describes how the system uses device files and how the Udev device manager dynamically creates or removes device node files.

## About Device Files

The `/dev` directory contains _device files_ or _device nodes_ that provide access to peripheral devices such as hard disks, to resources on peripheral devices such as disk partitions, and pseudo devices such as a random number generator.

The `/dev` directory has several subdirectory hierarchies, each of which holds device files that relate to a certain type of device. However, the contents of these subdirectories are implemented as symbolic links to corresponding files in `/dev`. Thus, the files can be accessed either through the linked file in `/dev` or the corresponding file in the subdirectory.

Using the `ls -l /dev` command lists files, some of which are flagged as being either type `b` \(for _block_\) or type `c` \(for _character_\). These devices have an associated pair of numbers that identify the device to the system.

```
ls -l /dev
```

```nocopybutton
total 0
crw-r--r--. 1 root root     10, 235 Aug 20 08:36 autofs
drwxr-xr-x. 2 root root         240 Sep 20 07:37 block
drwxr-xr-x. 2 root root         100 Aug 20 08:36 bsg
drwxr-xr-x. 3 root root          60 Nov  4  2019 bus
lrwxrwxrwx. 1 root root           3 Aug 20 08:36 cdrom -> sr0
drwxr-xr-x. 2 root root        2720 Sep 20 07:37 char
crw-------. 1 root root      5,   1 Aug 20 08:36 console
lrwxrwxrwx. 1 root root          11 Aug 20 08:36 core -> /proc/kcore
drwxr-xr-x. 3 root root          60 Nov  4  2019 cpu
crw-------. 1 root root     10,  62 Aug 20 08:36 cpu_dma_latency
drwxr-xr-x. 7 root root         140 Aug 20 08:36 disk
brw-rw----. 1 root disk    253,   0 Aug 20 08:36 dm-0
brw-rw----. 1 root disk    253,   1 Aug 20 08:36 dm-1
brw-rw----. 1 root disk    253,   2 Aug 20 08:36 dm-2
lrwxrwxrwx. 1 root root          13 Aug 20 08:36 fd -> /proc/self/fd
crw-rw-rw-. 1 root root      1,   7 Aug 20 08:36 full
crw-rw-rw-. 1 root root     10, 229 Aug 20 08:36 fuse
crw-------. 1 root root     10, 228 Aug 20 08:36 hpet
drwxr-xr-x. 2 root root           0 Aug 20 08:36 hugepages
crw-------. 1 root root     10, 183 Aug 20 08:36 hwrng
lrwxrwxrwx. 1 root root          12 Aug 20 08:36 initctl -> /run/initctl
drwxr-xr-x. 3 root root         220 Aug 20 08:36 input
crw-r--r--. 1 root root      1,  11 Aug 20 08:36 kmsg
lrwxrwxrwx. 1 root root          28 Aug 20 08:36 log -> /run/systemd/journal/dev-log
brw-rw----. 1 root disk      7,   0 Sep 23 01:28 loop0
crw-rw----. 1 root disk     10, 237 Sep 20 07:37 loop-control
drwxr-xr-x. 2 root root         120 Aug 20 08:36 mapper
crw-------. 1 root root     10, 227 Aug 20 08:36 mcelog
crw-r-----. 1 root kmem      1,   1 Aug 20 08:36 mem
crw-------. 1 root root     10,  59 Aug 20 08:36 memory_bandwidth
drwxrwxrwt. 2 root root          40 Nov  4  2019 mqueue
drwxr-xr-x. 2 root root          60 Aug 20 08:36 net
crw-------. 1 root root     10,  61 Aug 20 08:36 network_latency
crw-------. 1 root root     10,  60 Aug 20 08:36 network_throughput
crw-rw-rw-. 1 root root      1,   3 Aug 20 08:36 null
crw-------. 1 root root     10, 144 Aug 20 08:36 nvram
drwxr-xr-x. 2 root root         100 Aug 20 08:36 ol_ca-virtdoc-oltest1
crw-r-----. 1 root kmem      1,   4 Aug 20 08:36 port
crw-------. 1 root root    108,   0 Aug 20 08:36 ppp
crw-rw-rw-. 1 root tty       5,   2 Oct  7 08:10 ptmx
drwxr-xr-x. 2 root root           0 Aug 20 08:36 pts
crw-rw-rw-. 1 root root      1,   8 Aug 20 08:36 random
drwxr-xr-x. 2 root root          60 Nov  4  2019 raw
lrwxrwxrwx. 1 root root           4 Aug 20 08:36 rtc -> rtc0
crw-------. 1 root root    251,   0 Aug 20 08:36 rtc0
brw-rw----. 1 root disk      8,   0 Aug 20 08:36 sda
brw-rw----. 1 root disk      8,   1 Aug 20 08:36 sda1
brw-rw----. 1 root disk      8,   2 Aug 20 08:36 sda2
brw-rw----. 1 root disk      8,  16 Aug 20 08:36 sdb
brw-rw----. 1 root disk      8,  17 Aug 20 08:36 sdb1
crw-rw----. 1 root cdrom    21,   0 Aug 20 08:36 sg0
```

Block devices support random access to data, seeking media for data, and typically buffers data while data is being written or read. Examples of block devices include hard disks, CD-ROM drives, flash memory, and other addressable memory devices.

Character devices support the streaming of data to or from a device. The data isn't typically buffered nor is random access granted to data on a device. The kernel writes data to or reads data from a character device 1 byte at a time. Examples of character devices include keyboards, mice, terminals, pseudo terminals, and tape drives. `tty0` and `tty1` are character device files that correspond to terminal devices so users can log in from serial terminals or terminal emulators.

Pseudo terminals secondary devices emulate real terminal devices to interact with software. For example, a user might log in to a terminal device such as `/dev/tty1`, which then uses the pseudo terminal primary device, `/dev/pts/ptmx`, to interact with an underlying pseudo terminal device. The character device files for pseudo terminal secondary and primary devices are located in the `/dev/pts` directory, as shown in the following example:

```
ls -l /dev/pts
```

```nocopybutton
total 0
crw--w----. 1 guest tty  136, 0 Mar 17 10:11 0
crw--w----. 1 guest tty  136, 1 Mar 17 10:53 1
crw--w----. 1 guest tty  136, 2 Mar 17 10:11 2
c---------. 1 root  root   5, 2 Mar 17 08:16 ptmx
```

Some device entries, such as `stdin` for the standard input, are symbolically linked through the `self` subdirectory of the `proc` file system. The pseudo-terminal device file to which they actually point depends on the context of the process.

```
ls -l /proc/self/fd/[012]
```

```nocopybutton
lrwx------. 1 root root 64 Oct  7 08:23 /proc/self/fd/0 -> /dev/pts/0
lrwx------. 1 root root 64 Oct  7 08:23 /proc/self/fd/1 -> /dev/pts/0
lrwx------. 1 root root 64 Oct  7 08:23 /proc/self/fd/2 -> /dev/pts/0

```

Character devices, such as `null`, `random`, `urandom`, and `zero` are examples of pseudo devices that provide access to virtual functionality implemented in software rather than to physical hardware.

`/dev/null` is a data sink. Data that you write to `/dev/null` effectively disappears but the write operation succeeds. Reading from `/dev/null` returns `EOF` \(end-of-file\).

`/dev/zero` is a data source of an unlimited number of 0-value bytes.

`/dev/random` and `/dev/urandom` are data sources of streams of pseudo random bytes. To maintain high-entropy output, `/dev/random` blocks if its entropy pool doesn't contain sufficient bits of noise. `/dev/urandom` doesn't block and, thereforem, the entropy of its output might not be as consistently high as that of `/dev/random`. However, neither `/dev/random` nor `/dev/urandom` are considered to be truly random enough for the purposes of secure cryptography such as military-grade encryption.

You can find out the size of the entropy pool and the entropy value for `/dev/random` from virtual files under `/proc/sys/kernel/random`:

```
cat /proc/sys/kernel/random/poolsize
```

```nocopybutton
4096
```

```
cat /proc/sys/kernel/random/entropy_avail
```

```nocopybutton
3467
```

For more information, see the `null(4)`, `pts(4)`, and `random(4)` manual pages.

## About the Udev Device Manager

The Udev device manager dynamically creates or removes device node files at boot time . When creating a device node, `udev` reads the device’s `/sys` directory for attributes such as the label, serial number, and bus device number.

Udev can use persistent device names to guarantee consistent naming of devices across reboots, regardless of their order of discovery. Persistent device names are especially important when using external storage devices.

The configuration file for `udev` is `/etc/udev/udev.conf`, in which you can define the `udev_log` logging priority, which can be set to `err`, `info` and `debug`. Note that the default value is `err`.

For more information, see the `udev(7)` manual page.

## About Udev Rules

`udev` service \(`systemd-udevd`\) reads the rules files at system start-up and stores the rules in memory. If the kernel discovers a new device or an existing device goes offline, the kernel sends an event action \(_uevent_\) notification to `udev`, which matches the in-memory rules against the device attributes in the `/sys` directory to identify the device.

Multiple rules files exist in different directories. However, you only need to know about `/etc/udev/rules.d/*.rules` files because these are the only rules files that you can modify. See [Modifying Udev Rules](osmanage-ManagingSystemDevices.md#).

Udev processes the rules files in lexical order, regardless of the directory of the rule files. Rules files in `/etc/udev/rules.d` override rules files of the same name in other locations.

The following rules are extracted from the file `/lib/udev/rules.d/50-udev- default.rules` and illustrate the syntax of udev rules:

```
# do not edit this file, it will be overwritten on update

SUBSYSTEM=="block", SYMLINK{unique}+="block/%M:%m"
SUBSYSTEM!="block", SYMLINK{unique}+="char/%M:%m"

KERNEL=="pty[pqrstuvwxyzabcdef][0123456789abcdef]", GROUP="tty", MODE="0660"
KERNEL=="tty[pqrstuvwxyzabcdef][0123456789abcdef]", GROUP="tty", MODE="0660"
...
# mem
KERNEL=="null|zero|full|random|urandom", MODE="0666"
KERNEL=="mem|kmem|port|nvram",  GROUP="kmem", MODE="0640"
...
# block
SUBSYSTEM=="block", GROUP="disk"
...
# network
KERNEL=="tun",                  MODE="0666"
KERNEL=="rfkill",               MODE="0644"

# CPU
KERNEL=="cpu[0-9]*",            MODE="0444"
...
# do not delete static device nodes
ACTION=="remove", NAME=="", TEST=="/lib/udev/devices/%k", \
    OPTIONS+="ignore_remove"
ACTION=="remove", NAME=="?*", TEST=="/lib/udev/devices/$name", \
    OPTIONS+="ignore_remove"
```

A rule either assigns a value to a key or it tries to find a match for a key by comparing its current value with the specified value. The following table shows the assignment and comparison operators that you can use.

<table><thead><tr><th>

Operator

</th><th>

Description

</th></tr></thead><tbody><tr><td>

`=`

</td><td>

Assign a value to a key, overwriting any previous value.

</td></tr><tr><td>

`+=`

</td><td>

Assign a value by appending it to the key's current list of values.

</td></tr><tr><td>

`:=`

</td><td>

Assign a value to a key. This value cannot be changed by any further rules.

</td></tr><tr><td>

`==`

</td><td>

Match the key's current value against the specified value for equality.

</td></tr><tr><td>

`!=`

</td><td>

Match the key's current value against the specified value for equality.

</td></tr><tbody></table>
You can use the following shell-style pattern-matching characters in values.

<table><thead><tr><th>

Character

</th><th>

Description

</th></tr></thead><tbody><tr><td>

`?`

</td><td>

Matches a single character.

</td></tr><tr><td>

`*`

</td><td>

Matches any number of characters, including zero.

</td></tr><tr><td>

`[]`

</td><td>

Matches any single character or character from a range of characters specified within the brackets. For example, `tty[sS][0-9]` would match `ttys7` or `ttyS7`.

</td></tr><tbody></table>
The following table describes commonly used match keys in rules.

<table><thead><tr><th>

Match Key

</th><th>

Description

</th></tr></thead><tbody><tr><td>

`ACTION`

</td><td>

Matches the name of the action that led to an event. For example, `ACTION="add"` or `ACTION="remove"`.

</td></tr><tr><td>

`ENV{*key*}`

</td><td>

Matches a value for the device property _key_. For example, `ENV{DEVTYPE}=="disk"`.

</td></tr><tr><td>

`KERNEL`

</td><td>

Matches the name of the device that is affected by an event. For example, `KERNEL=="dm-*"` for disk media.

</td></tr><tr><td>

`NAME`

</td><td>

Matches the name of a device file or network interface. For example, `NAME="?*"` for any name that consists of one or more characters.

</td></tr><tr><td>

`SUBSYSTEM`

</td><td>

Matches the subsystem of the device that is affected by an event. For example, `SUBSYSTEM=="tty"`.

</td></tr><tr><td>

`TEST`

</td><td>

Tests wheter the specified file or path exists; for example, `TEST=="/lib/udev/devices/$name"`, where `$name` is the name of the currently matched device file.

</td></tr><tbody></table>
Other match keys include `ATTR{*filename*}`, `ATTRS{*filename*}`, `DEVPATH`, `DRIVER`, `DRIVERS`, `KERNELS`, `PROGRAM`, `RESULT`, `SUBSYSTEMS`, and `SYMLINK`.

The following table describes commonly used assignment keys in rules.

<table><thead><tr><th>

Assignment Key

</th><th>

Description

</th></tr></thead><tbody><tr><td>

`ENV{*key*}`

</td><td>

Specifies a value for the device property _key_, such as `GROUP="disk"`.

</td></tr><tr><td>

`GROUP`

</td><td>

Specifies the group for a device file, such as `GROUP="disk"`.

</td></tr><tr><td>

`IMPORT{*type*}`

</td><td>

Specifies a set of variables for the device property, depending on _type_:

- **`cmdline`**

Import a single property from the boot `kernel` command line. For simple flags, `udev` sets the value of the property to 1. For example, `IMPORT{cmdline}="nodmraid"`.

- **`db`**

Interpret the specified value as an index into the device database and import a single property, which must have already been set by an earlier event. For example, `IMPORT{db}="DM_UDEV_LOW_PRIORITY_FLAG"`.

- **`file`**

Interpret the specified value as the name of a text file and import its contents, which must be in environmental key format. For example, `IMPORT{file}="keyfile"`.

- **`parent`**

Interpret the specified value as a key-name filter and import the stored keys from the database entry for the parent device. For example `IMPORT{parent}="ID_*"`.

- **`program`**

Run the specified value as an external program and imports its result, which must be in environmental key format. For example `IMPORT{program}="usb_id --export %p"`.

</td></tr><tr><td>

`MODE`

</td><td>

Specifies the permissions for a device file, such as `MODE="0640"`.

</td></tr><tr><td>

`NAME`

</td><td>

Specifies the name of a device file, such as `NAME="em1"`.

</td></tr><tr><td>

`OPTIONS`

</td><td>

Specifies rule and device options, such as `OPTIONS+="ignore_remove"`, which means that the device file isn't removed if the device is removed.

</td></tr><tr><td>

`OWNER`

</td><td>

Specifies the owner for a device file, such as `GROUP="root"`.

</td></tr><tr><td>

`RUN`

</td><td>

Specifies a command to be run after the device file has been created, such as `RUN+="/usr/bin/eject $kernel"`, where `$kernel` is the kernel name of the device.

</td></tr><tr><td>

`SYMLINK`

</td><td>

Specifies the name of a symbolic link to a device file, such as `SYMLINK+="disk/by-uuid/$env{ID_FS_UUID_ENC}"`, where `$env{}` is substituted with the specified device property.

</td></tr><tbody></table>
Other assignment keys include `ATTR{*key*}`, `GOTO`, `LABEL`, `RUN`, and `WAIT_FOR`.

The following table describes the string substitutions that are commonly used with the `GROUP`, `MODE`, `NAME`, `OWNER`, `PROGRAM`, `RUN`, and `SYMLINK` keys.

<table><thead><tr><th>

String Substitution

</th><th>

Description

</th></tr></thead><tbody><tr><td>

`$attr{*file*}` or

`%s{*file*}`

</td><td>

Specifies the value of a device attribute from a file under `/sys`, such as `ENV{MATCHADDR}="$attr{address}"`.

</td></tr><tr><td>

`$devpath` or

`%p`

</td><td>

The device path of the device in the `sysfs` file system under `/sys`, such as `RUN+="keyboard-force-release.sh $devpath common-volume-keys"`.

</td></tr><tr><td>

`$env{*key*}` or

`%E{*key*}`

</td><td>

Specifies the value of a device property, such as `SYMLINK+="disk/by-id/md-name-$env{MD_NAME}-part%n"`.

</td></tr><tr><td>

`$kernel` or

`%k`

</td><td>

Specifies the kernel name for the device.

</td></tr><tr><td>

`$major` or

`%M`

</td><td>

Specifies the major number of a device, such as `IMPORT{program}="udisks-dm-export %M %m"`.

</td></tr><tr><td>

`$minor` or

`%m`

</td><td>

Specifies the minor number of a device, such as `RUN+="$env{LVM_SBIN_PATH}/lvm pvscan --cache --major $major --minor $minor"`.

</td></tr><tr><td>

`$name`

</td><td>

Specifies the device file of the current device, such as `TEST=="/lib/udev/devices/$name"`.

</td></tr><tbody></table>
Udev expands the strings specified for `RUN` immediately before its program is run, which is after udev has finished processing all other rules for the device. For the other keys, `udev` expands the strings while it's processing the rules.

For more information, see the `udev(7)` manual page.

## Querying Udev and Sysfs

You can use the `udevadm` command to query the `udev` database and `sysfs`.

To query the `sysfs` device path relative to `/sys` that corresponds to the device file `/dev/sda`:

```
udevadm info --query=path --name=/dev/sda
```

```nocopybutton
/devices/pci0000:00/0000:00:0d.0/host0/target0:0:0/0:0:0:0/block/sda
```

To query the symbolic links that point to `/dev/sda`, use the following command:

```
udevadm info --query=symlink --name=/dev/sda
```

```nocopybutton
block/8:0
disk/by-id/ata-VBOX_HARDDISK_VB6ad0115d-356e4c09
disk/by-id/scsi-SATA_VBOX_HARDDISK_VB6ad0115d-356e4c09
disk/by-path/pci-0000:00:0d.0-scsi-0:0:0:0
```

To query the properties of `/dev/sda`, use the following command:

```
udevadm info --query=property --name=/dev/sda
```

```nocopybutton'
UDEV_LOG=3
DEVPATH=/devices/pci0000:00/0000:00:0d.0/host0/target0:0:0/0:0:0:0/block/sda
MAJOR=8
MINOR=0
DEVNAME=/dev/sda
DEVTYPE=disk
SUBSYSTEM=block
ID_ATA=1
ID_TYPE=disk
ID_BUS=ata
ID_MODEL=VBOX_HARDDISK
ID_MODEL_ENC=VBOX\x20HARDDISK\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20...
ID_REVISION=1.0
ID_SERIAL=VBOX_HARDDISK_VB579a85b0-bf6debae
ID_SERIAL_SHORT=VB579a85b0-bf6debae
ID_ATA_WRITE_CACHE=1
ID_ATA_WRITE_CACHE_ENABLED=1
ID_ATA_FEATURE_SET_PM=1
ID_ATA_FEATURE_SET_PM_ENABLED=1
ID_ATA_SATA=1
ID_ATA_SATA_SIGNAL_RATE_GEN2=1
ID_SCSI_COMPAT=SATA_VBOX_HARDDISK_VB579a85b0-bf6debae
ID_PATH=pci-0000:00:0d.0-scsi-0:0:0:0
ID_PART_TABLE_TYPE=dos
LVM_SBIN_PATH=/sbin
UDISKS_PRESENTATION_NOPOLICY=0
UDISKS_PARTITION_TABLE=1
UDISKS_PARTITION_TABLE_SCHEME=mbr
UDISKS_PARTITION_TABLE_COUNT=2
UDISKS_ATA_SMART_IS_AVAILABLE=0
DEVLINKS=/dev/block/8:0 /dev/disk/by-id/ata-VBOX_HARDDISK_VB579a85b0-bf6debae ...
```

To query the entire information for `/dev/sda`, use the following command:

```
udevadm info --query=all --name=/dev/sda
```

```nocopybutton
P: /devices/pci0000:00/0000:00:0d.0/host0/target0:0:0/0:0:0:0/block/sda
N: sda
W: 37
S: block/8:0
S: disk/by-id/ata-VBOX_HARDDISK_VB579a85b0-bf6debae
S: disk/by-id/scsi-SATA_VBOX_HARDDISK_VB579a85b0-bf6debae
S: disk/by-path/pci-0000:00:0d.0-scsi-0:0:0:0
E: UDEV_LOG=3
E: DEVPATH=/devices/pci0000:00/0000:00:0d.0/host0/target0:0:0/0:0:0:0/block/sda
E: MAJOR=8
E: MINOR=0
E: DEVNAME=/dev/sda
E: DEVTYPE=disk
E: SUBSYSTEM=block
E: ID_ATA=1
E: ID_TYPE=disk
E: ID_BUS=ata
E: ID_MODEL=VBOX_HARDDISK
E: ID_MODEL_ENC=VBOX\x20HARDDISK\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20...
E: ID_SERIAL=VBOX_HARDDISK_VB579a85b0-bf6debae
E: ID_SERIAL_SHORT=VB579a85b0-bf6debae
E: ID_ATA_WRITE_CACHE=1
E: ID_ATA_WRITE_CACHE_ENABLED=1
E: ID_ATA_FEATURE_SET_PM=1
E: ID_ATA_FEATURE_SET_PM_ENABLED=1
E: ID_ATA_SATA=1
E: ID_ATA_SATA_SIGNAL_RATE_GEN2=1
E: ID_SCSI_COMPAT=SATA_VBOX_HARDDISK_VB579a85b0-bf6debae
E: ID_PATH=pci-0000:00:0d.0-scsi-0:0:0:0
E: ID_PART_TABLE_TYPE=dos
E: LVM_SBIN_PATH=/sbin
E: UDISKS_PRESENTATION_NOPOLICY=0
E: UDISKS_PARTITION_TABLE=1
E: UDISKS_PARTITION_TABLE_SCHEME=mbr
E: UDISKS_PARTITION_TABLE_COUNT=2
E: UDISKS_ATA_SMART_IS_AVAILABLE=0
E: DEVLINKS=/dev/block/8:0 /dev/disk/by-id/ata-VBOX_HARDDISK_VB579a85b0-bf6debae ...
```

To display all of the properties of `/dev/sda`, as well as the parent devices that `udev` has found in `/sys`, use the following command:

```
udevadm info --attribute-walk --name=/dev/sda
```

```nocopybutton'
...
  looking at device '/devices/pci0000:00/0000:00:0d.0/host0/target0:0:0/0:0:0:0/block/sda':
    KERNEL=="sda"
    SUBSYSTEM=="block"
    DRIVER==""
    ATTR{range}=="16"
    ATTR{ext_range}=="256"
    ATTR{removable}=="0"
    ATTR{ro}=="0"
    ATTR{size}=="83886080"
    ATTR{alignment_offset}=="0"
    ATTR{capability}=="52"
    ATTR{stat}=="   20884    15437  1254282   338919     5743     8644   103994   109005 ...
    ATTR{inflight}=="       0        0"

  looking at parent device '/devices/pci0000:00/0000:00:0d.0/host0/target0:0:0/0:0:0:0':
    KERNELS=="0:0:0:0"
    SUBSYSTEMS=="scsi"
    DRIVERS=="sd"
    ATTRS{device_blocked}=="0"
    ATTRS{type}=="0"
    ATTRS{scsi_level}=="6"
    ATTRS{vendor}=="ATA     "
    ATTRS{model}=="VBOX HARDDISK   "
    ATTRS{rev}=="1.0 "
    ATTRS{state}=="running"
    ATTRS{timeout}=="30"
    ATTRS{iocounterbits}=="32"
    ATTRS{iorequest_cnt}=="0x6830"
    ATTRS{iodone_cnt}=="0x6826"
    ATTRS{ioerr_cnt}=="0x3"
    ATTRS{modalias}=="scsi:t-0x00"
    ATTRS{evt_media_change}=="0"
    ATTRS{dh_state}=="detached"
    ATTRS{queue_depth}=="31"
    ATTRS{queue_ramp_up_period}=="120000"
    ATTRS{queue_type}=="simple"

  looking at parent device '/devices/pci0000:00/0000:00:0d.0/host0/target0:0:0':
    KERNELS=="target0:0:0"
    SUBSYSTEMS=="scsi"
    DRIVERS==""

  looking at parent device '/devices/pci0000:00/0000:00:0d.0/host0':
    KERNELS=="host0"
    SUBSYSTEMS=="scsi"
    DRIVERS==""

  looking at parent device '/devices/pci0000:00/0000:00:0d.0':
    KERNELS=="0000:00:0d.0"
    SUBSYSTEMS=="pci"
    DRIVERS=="ahci"
    ATTRS{vendor}=="0x8086"
    ATTRS{device}=="0x2829"
    ATTRS{subsystem_vendor}=="0x0000"
    ATTRS{subsystem_device}=="0x0000"
    ATTRS{class}=="0x010601"
    ATTRS{irq}=="21"
    ATTRS{local_cpus}=="00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000003"
    ATTRS{local_cpulist}=="0-1"
    ATTRS{modalias}=="pci:v00008086d00002829sv00000000sd00000000bc01sc06i01"
    ATTRS{numa_node}=="-1"
    ATTRS{enable}=="1"
    ATTRS{broken_parity_status}=="0"
    ATTRS{msi_bus}==""
    ATTRS{msi_irqs}==""

  looking at parent device '/devices/pci0000:00':
    KERNELS=="pci0000:00"
    SUBSYSTEMS==""
    DRIVERS==""
```

The command starts at the device that's specified by the device path and walks the chain of parent devices. For every device that the command finds, the command displays the possible attributes for the device and its parent devices by using the match key format for `udev` rules.

For more information, see the `udevadm(8)` manual page.

## Modifying Udev Rules

The order in which rules are evaluated is important. Udev processes rules in lexical order. If you want to add custom rules, you need `udev` to locate and evaluate these rules before the default rules.

The following example illustrates how to implement a `udev` rules file that adds a symbolic link to the disk device `/dev/sdb`.

1. Create a rule file under `/etc/udev/rules.d` with a file name such as `10-local.rules` that udev reads before any other rules file.

   The following rule in `10-local.rules` creates the symbolic link `/dev/my_disk`, which points to `/dev/sdb`:

   ```
   KERNEL=="sdb", ACTION=="add", SYMLINK="my_disk"
   ```

   Listing the device files in `/dev` shows that `udev` hasn't yet applied the rule:

   ```
   ls /dev/sd* /dev/my_disk
   ```

   ```nocopybutton
   ls: cannot access /dev/my_disk: No such file or directory
   /dev/sda  /dev/sda1  /dev/sda2  /dev/sdb
   ```

2. To simulate how `udev` applies its rules to create a device, you can use the `udevadm test` command with the device path of `sdb` listed under the `/sys/class/block` hierarchy, for example:

   ```
   udevadm test /sys/class/block/sdb
   ```

   ```nocopybutton
   calling: test
   version ...
   This program is for debugging only, it does not run any program
   specified by a RUN key. It may show incorrect results, because
   some values may be different, or not available at a simulation run.
   ...
   LINK 'my_disk' /etc/udev/rules.d/10-local.rules:1
   ...
   creating link '/dev/my_disk' to '/dev/sdb'
   creating symlink '/dev/my_disk' to 'sdb
   ...
   ACTION=add
   DEVLINKS=/dev/disk/by-id/ata-VBOX_HARDDISK_VB186e4ce2-f80f170d 
     /dev/disk/by-uuid/a7dc508d-5bcc-4112-b96e-f40b19e369fe 
     /dev/my_disk
   ...
   ```

3. Restart the `systemd-udevd` service:

   ```
   sudo systemctl restart systemd-udevd
   ```

   After `udev` processes the rules files, the symbolic link `/dev/my_disk` has been added:

   ```
   ls -F /dev/sd* /dev/my_disk
   ```

   ```nocopybutton
   /dev/my_disk@  /dev/sda  /dev/sda1  /dev/sda2  /dev/sdb
   ```

4. \(Optional\) To undo the changes, remove `/etc/udev/rules.d/10-local.rules` and `/dev/my_disk`, then run `systemctl restart systemd-udevd` again.
