<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Managing Kernels and System Boot

This chapter describes the Enterprise Linux boot process and how to configure and use the GRand Unified Bootloader \(GRUB\) version 2 and boot-related kernel parameters.

## About the Boot Process

Understanding the Enterprise Linux boot process can help you troubleshoot problems when booting a system. The boot process involves several files, and errors in these files are the usual cause of boot problems. Boot processes and configuration differ depending on whether the hardware uses UEFI firmware or legacy BIOS to handle system boot.

### About UEFI-Based Booting

On a UEFI-based system running the Enterprise Linux release, the system boot process uses the following sequence:

1. The system's UEFI firmware performs a power-on self-test \(POST\) and then detects and initializes peripheral devices and the hard disk.

2. UEFI searches for a GPT partition with a specific globally unique identifier \(GUID\) that identifies it as the EFI System Partition \(ESP\). This partition contains EFI applications such as boot loaders. In case of the presence of multiple boot devices, the UEFI boot manager uses the appropriate ESP based on the order that's defined in the boot manager. With the `efibootmgr` tool, you can define a different order, if you don't want to use the default definition.

3. The UEFI boot manager checks whether Secure Boot is enabled. If Secure Boot is disabled, the boot manager runs the GRUB 2 bootloader on the ESP.

   Otherwise, the boot manager requests a certificate from the boot loader and validates this against keys stored in the UEFI Secure Boot key database. To handle the certificate validation process, the environment is configured to perform a 2-stage boot process and the `shim.efi` application that's responsible for certification is loaded first before loading the GRUB 2 bootloader. If the certificate is valid, the boot loader runs and, in turn, validates the kernel that it's configured to load.

4. The boot loader loads the `vmlinuz` kernel image file into memory and extracts the contents of the `initramfs` image file into a temporary, memory-based file system \(`tmpfs`\).

5. The kernel loads the driver modules from the `initramfs` file system that are needed to access the root file system.

6. The kernel starts the `systemd` process with a process ID of 1 \(PID 1\). See [About the systemd Service Manager](osmanage-WorkingWithSystemServices.md#).

7. `systemd` runs any additional processes defined for it.

   **Note:**

   Specify any other actions to be processed during the boot process by defining your own `systemd` unit. This method is the preferred approach than using the `/etc/rc.local` file.

### About BIOS-Based Booting

On a BIOS-based system running the Enterprise Linux release, the boot process is as follows:

1. The system's BIOS performs a power-on self-test \(POST\), and then detects and initializes any peripheral devices and the hard disk.

2. The BIOS reads the Master Boot Record \(MBR\) into memory from the boot device. The MBR stores information about the organization of partitions on that device, the partition table, and the boot signature which is used for error detection. The MBR also includes the pointer to the boot loader program \(GRUB 2\). The boot program itself can be on the same device or on another device.

3. The boot loader loads the `vmlinuz` kernel image file into memory and extracts the contents of the `initramfs` image file into a temporary, memory-based file system \(`tmpfs`\).

4. The kernel loads the driver modules from the `initramfs` file system that are needed to access the root file system.

5. The kernel starts the `systemd` process with a process ID of 1 \(PID 1\). See [About the systemd Service Manager](osmanage-WorkingWithSystemServices.md#) for more information.

6. `systemd` runs any additional processes defined for it.

   **Note:**

   Specify any other actions to be processed during the boot process by defining user `systemd` units. This method is the preferred approach than using the `/etc/rc.local` file.

## About the GRUB 2 Bootloader

In addition to Enterprise Linux, GRUB 2 can load and chain-load many proprietary operating systems. GRUB 2 understands the formats of file systems and kernel executable files. Therefore, it can load an arbitrary OS without needing to know the exact location of the kernel on the boot device. GRUB 2 requires only the file name and drive partitions to load a kernel. You can configure this information by using the GRUB 2 menu or by entering it on the command line.

GRUB 2 behavior is based on configuration files. On BIOS-based systems, the configuration file is `/boot/grub2/grub.cfg`. On UEFI-based systems, the configuration file is `/boot/efi/EFI/redhat/grub.cfg`. Each kernel version's boot parameters are stored in independent configuration files in `/boot/loader/entries`. Each kernel configuration is stored with the file name `*machine\_id*-*kernel\_version*.el8.*arch*.conf`.

**Note:**

Don't edit the GRUB 2 configuration file directly.

The `grub2-mkconfig` command generates the configuration file using the template scripts in `/etc/grub.d` and menu-configuration settings taken from the configuration file, `/etc/default/grub`.

The default menu entry is set by the value of the `GRUB_DEFAULT` parameter in `/etc/default/grub`. If `GRUB_DEFAULT` is set to `saved`, you can use the `grub2-set-default` and `grub2-reboot` commands to specify the default entry. The command `grub2-set-default` sets the default entry for all subsequent reboots, while `grub2-reboot` sets the default entry for the next reboot only.

If you specify a numeric value as the value of `GRUB_DEFAULT` or as an argument to either `grub2-reboot` or `grub2-set-default`, GRUB 2 counts the menu entries in the configuration file starting at 0 for the first entry.

For more information about using, configuring, and customizing GRUB 2, see the [GNU GRUB Manual](https://www.gnu.org/software/grub/manual/grub/grub.html), which is also installed as `/usr/share/doc/grub2-tools-2.00/grub.html`.

## About Linux Kernels

The Linux Foundation provides a hub for open source developers to code, manage, and scale different open technology projects. It also manages the Linux Kernel Organization that exists to distribute various versions of the Linux kernel which is at the core of all Linux distributions, including those used by Enterprise Linux. The Linux kernel manages the interactions between the computer hardware and user space applications that run on Enterprise Linux.

OpenELA includes **Red Hat Compatible Kernel** \(RHCK\), which is fully compatible with the Linux kernel that's distributed in a corresponding Red Hat Enterprise Linux \(RHEL\) release. You can use RHCK to ensure full compatibility with applications that run on Red Hat Enterprise Linux.

**Important:** Linux kernels are critical for running applications in the Enterprise Linux user space. Therefore, you must keep the kernel current with the latest bug fixes, enhancements, and security updates provided by OpenELA.

## Managing Kernels in GRUB 2 Using `grubby`

You can use the `grubby` command to view and manage kernels.

Use the following command to display the kernels that are installed and configured on the system:

```
sudo grubby --info=ALL
```

To configure a specific kernel as the default boot kernel, run:

```
sudo grubby --set-default /boot/vmlinuz-4.18.0-80.el8.x86_64
```

You can also use the `grubby` command to update a kernel configuration entry to add or remove kernel boot arguments, for example:

```
sudo grubby --remove-args="rhgb quiet" --args=rd_LUKS_UUID=luks-39fec799-6a6c-4ac1-ac7c-1d68f2e6b1a4 \
--update-kernel /boot/vmlinuz-4.18.0-80.el8.x86_64
```

For more information about the `grubby` command, see the `grubby(8)` manual page.

## Kernel Boot Parameters

The following table describes some commonly used kernel boot parameters.

<table><thead><tr><th>

Option

</th><th>

Description

</th></tr></thead><tbody><tr><td>

`0`, `1`, `2`, `3`, `4`, `5`, or `6`, or `systemd.unit=runlevel*N*.target`

</td><td>

Specifies the nearest `systemd`-equivalent system-state target to match a legacy SysV run level. _N_ can take an integer value between 0 and 6.

Systemd maps system-state targets to mimic the legacy SysV init system. For a description of system-state targets, see [About System-State Targets](osmanage-WorkingWithSystemServices.md#).

</td></tr><tr><td>

`1`, `s`, `S`, `single`, or `systemd.unit=rescue.target`

</td><td>

Specifies the rescue shell. The system boots to single-user mode prompts for the `root` password.

</td></tr><tr><td>

`3` or `systemd.unit=multi-user.target`

</td><td>

Specifies the `systemd` target for multiuser, nongraphical login.

</td></tr><tr><td>

`5` or `systemd.unit=graphical.target`

</td><td>

Specifies the `systemd` target for multiuser, graphical login.

</td></tr><tr><td>

`-b`, `emergency`, or `systemd.unit=emergency.target`

</td><td>

Specifies emergency mode. The system boots to single-user mode and prompts for the `root` password. Fewer services are started than when in rescue mode.

</td></tr><tr><td>

`KEYBOARDTYPE=*kbtype*`

</td><td>

Specifies the keyboard type, which is written to `/etc/sysconfig/keyboard` in the `initramfs`.

</td></tr><tr><td>

`KEYTABLE=*kbtype*`

</td><td>

Specifies the keyboard layout, which is written to `/etc/sysconfig/keyboard` in the `initramfs`.

</td></tr><tr><td>

`LANG=*language*_*territory*.*codeset*`

</td><td>

Specifies the system language and code set, which is written to `/etc/sysconfig/i18n` in the `initramfs`.

</td></tr><tr><td>

`max_loop=*N*`

</td><td>

Specifies the number of loop devices \(`/dev/loop*`\) that are available for accessing files as block devices. The default and maximum values of _N_ are 8 and 255.

</td></tr><tr><td>

`nouptrack`

</td><td>

Disables Ksplice Uptrack updates from being applied to the kernel.

</td></tr><tr><td>

`quiet`

</td><td>

Reduces debugging output.

</td></tr><tr><td>

`rd_LUKS_UUID=*UUID*`

</td><td>

Activates an encrypted Linux Unified Key Setup \(LUKS\) partition with the specified UUID.

</td></tr><tr><td>

`rd_LVM_VG=*vg*/lv_*vol*`

</td><td>

Specifies an LVM volume group and volume to be activated.

</td></tr><tr><td>

`rd_NO_LUKS`

</td><td>

Disables detection of an encrypted LUKS partition.

</td></tr><tr><td>

`rhgb`

</td><td>

Specifies to use the Red Hat graphical boot display to indicate the progress of booting.

</td></tr><tr><td>

`rn_NO_DM`

</td><td>

Disables Device-Mapper \(DM\) RAID detection.

</td></tr><tr><td>

`rn_NO_MD`

</td><td>

Disables Multiple Device \(MD\) RAID detection.

</td></tr><tr><td>

`ro root=/dev/mapper/*vg*-lv_root`

</td><td>

Specifies that the root file system is to be mounted read-only, and specifies the root file system by the device path of its LVM volume \(where _vg_ is the name of the volume group\).

</td></tr><tr><td>

`rw root=UUID=*UUID*`

</td><td>

Specifies that the root \(`/`\) file system is to be mounted read-writable at boot time, and specifies the root partition by its UUID.

</td></tr><tr><td>

`selinux=0`

</td><td>

Disables SELinux.

</td></tr><tr><td>

`SYSFONT=*font*`

</td><td>

Specifies the console font, which is written to `/etc/sysconfig/i18n` in the `initramfs`.

</td></tr><tbody></table>
The kernel boot parameters that were last used to boot a system are recorded in `/proc/cmdline`, for example:

```
sudo cat /proc/cmdline
```

```nocopybutton
BOOT_IMAGE=(hd0,msdos1)/vmlinuz-4.18.0-80.el8.x86_64 root=/dev/mapper/ol-root ro \
crashkernel=1G-4G:192M,4G-64G:256M,64G-:512M resume=/dev/mapper/ol-swap rd.lvm.lv=ol/root \
rd.lvm.lv=ol/swap rhgb quiet
```

For more information, see the `kernel-command-line(7)` manual page.

## Modifying Kernel Boot Parameters Before Booting

To modify boot parameters before booting a kernel, follow these steps:

1. When the GRUB boot menu appears at the beginning of the boot process, use the arrow keys to highlight the required kernel and press the space bar.

2. Press E to edit the boot configuration for the kernel.

3. Use the arrow keys to bring the cursor to the end of the line that starts with `linux`, which is the boot configuration line for the kernel.

4. Modify the boot parameters.

   You can add parameters such as `systemd.target=runlevel1.target`, which instructs the system to boot into the rescue shell.

5. Press Ctrl+X to boot the system.

## Modifying GRUB 2 Default Kernel Boot Parameters

To modify the boot parameters for the GRUB 2 configuration so that these parameters are applied by default at every reboot, follow these steps:

1. Edit `/etc/default/grub` and add parameter settings to the `GRUB_CMDLINE_LINUX` definition, for example:

   ```
   GRUB_CMDLINE_LINUX="vconsole.font=latarcyrheb-sun16 vconsole.keymap=uk 
   crashkernel=1G-4G:192M,4G-64G:256M,64G-:512M  rd.lvm.lv=ol/swap rd.lvm.lv=ol/root biosdevname=0 
   rhgb quiet **systemd.unit=runlevel3.target**"
   ```

   This example adds the parameter `systemd.unit=runlevel3.target` so that the system boots into multiuser, nongraphical mode by default.

2. Rebuild `/boot/grub2/grub.cfg`:

   ```
   sudo grub2-mkconfig -o /boot/grub2/grub.cfg
   ```

   The change takes effect at the next system reboot of all configured kernels.

**Note:**

For systems that boot with UEFI, the `grub.cfg` file is located in the `/boot/efi/EFI/redhat` directory because the boot configuration is stored on a dedicated FAT32-formatted partition.

After the system has successfully booted, the `EFI` folder on that partition is mounted inside the `/boot/efi` directory on the root file system for Enterprise Linux.
