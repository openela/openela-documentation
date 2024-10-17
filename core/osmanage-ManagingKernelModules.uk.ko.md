<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Managing Kernel Modules

This chapter describes how to load, unload, and modify the behavior of kernel modules.

## About Kernel Modules

The boot loader loads the kernel into memory. You can add new code to the kernel by including the source files in the kernel source tree and recompiling the kernel. Kernel modules provide device drivers that enable the kernel to access new hardware, support different file system types, and extend its functionality in other ways. The modules can be dynamically loaded and unloaded on demand. To avoid wasting memory on unused device drivers, Enterprise Linux supports loadable kernel modules \(LKMs\), which enable a system to run with only the device drivers and kernel code that are required to be loaded into memory.

Kernel modules can be signed to protect the system from running malicious code at boot time. When UEFI Secure Boot is enabled, only kernel modules that contain the correct signature information can be loaded.

## Listing Information About Loaded Modules

The `lsmod` command lists the modules that are loaded into the kernel:

```
lsmod
```

```nocopybutton
Module                  Size  Used by
udp_diag               16384  0
ib_core               311296  0
tcp_diag               16384  0
inet_diag              24576  2 tcp_diag,udp_diag
nfsv3                  49152  0
nfs_acl                16384  1 nfsv3
...
dm_mirror              24576  0
dm_region_hash         20480  1 dm_mirror
dm_log                 20480  2 dm_region_hash,dm_mirror
...
```

The output shows the module name, the amount of memory it uses, the number of processes using the module and the names of other modules on which it depends. The module `dm_log`, for example, depends on the `dm_region_hash` and `dm_mirror` modules. The example also shows that two processes are using all three modules.

Show detailed information about a module by using the `modinfo` command:

```
modinfo ahci
```

```nocopybutton
filename:       /lib/modules/5.4.17-2136.306.1.3.el8OpenELA CA Server
sig_key:        22:07:CB:47:59:F3:50:A0:A2:FA:24:CE:B4:00:53:4E:C5:1D:C6:2A
sig_hashalgo:   sha512
signature:      2F:AE:AF:6D:56:92:69:C4:77:AB:E1:3D:41:09:AF:A6:FC:1D:3B:A2:
                9C:23:79:6F:17:82:D5:A3:9B:61:64:32:72:9B:98:C9:8C:89:73:FB:
                A4:86:4F:B5:7D:DF:84:8E:05:26:4F:22:CB:02:41:38:7B:7C:CB:C2:
                ...
                9F:FD:94:8F:35:9B:2A:89:3E:E1:17:40:49:79:30:8B:92:4D:3A:9A:
                F4:C7:82:8D:26:BE:6D:FB:71:C6:E5:FD
parm:           marvell_enable:Marvell SATA via AHCI (1 = enabled) (int)
parm:           mobile_lpm_policy:Default LPM policy for mobile chipsets (int)

...
```

The output would include the following information:

- **`filename`**

  Absolute path of the kernel object file.

- **`version`**

  Version number of the module. Note that the version number might not be updated for patched modules and might be missing or removed in newer kernels.

- **`license`**

  License information for the module.

- **`description`**

  Short description of the module.

- **`author`**

  Author credit for the module.

- **`srcversion`**

  Hash of the source code used to create the module.

- **`alias`**

  Internal alias names for the module.

- **`depends`**

  Comma-separated list of any modules on which this module depends.

- **`retpoline`**

  A flag indicating that the module is built that includes a mitigation against the Spectre security vulnerability.

- **`intree`**

  A flag indicating that the module is built from the kernel in-tree source and isn't tainted.

- **`vermagic`**

  Kernel version that was used to compile the module, which is checked against the current kernel when the module is loaded.

- **`sig_id`**

  The method used to store signing keys that might have been used to sign a module for Secure Boot, typically PKCS\#7

- **`signer`**

  The name of the signing key used to sign a module for Secure Boot.

- **`sig_key`**

  The signature key identifier for the key used to sign the module.

- **`sig_hashalgo`**

  The algorithm used to generate the signature hash for a signed module.

- **`signature`**

  The signature data for a signed module.

- **`parm`**

  Module parameters and descriptions.

Modules are loaded into the kernel from kernel object files \(`/lib/modules/*kernel\_version*/kernel/*ko*`\). To display the absolute path of a kernel object file, specify the `-n` option, for example:

```
modinfo -n parport
```

```nocopybutton
/lib/modules/5.4.17-2136.306.1.3.el8
```

For more information, see the `lsmod(5)` and `modinfo(8)` manual pages.

## Loading and Unloading Modules

The `modprobe` command loads kernel modules, for example:

```
sudo modprobe nfs
sudo lsmod | grep nfs
```

```nocopybutton
nfs                   266415  0 
lockd                  66530  1 nfs
fscache                41704  1 nfs
nfs_acl                 2477  1 nfs
auth_rpcgss            38976  1 nfs
sunrpc                204268  5 nfs,lockd,nfs_acl,auth_rpcgss
```

Include the `-v` \(verbose\) option to show whether any additional modules are loaded to resolve dependencies.

```
sudo modprobe -v nfs
```

```nocopybutton
insmod /lib/modules/4.18.0-80.el8.x86_64/kernel/net/sunrpc/auth_gss/auth_rpcgss.ko 
insmod /lib/modules/4.18.0-80.el8.x86_64/kernel/fs/nfs_common/nfs_acl.ko 
insmod /lib/modules/4.18.0-80.el8.x86_64/kernel/fs/fscache/fscache.ko 
...
```

**Note:**

The `modprobe` command does not reload modules that are already loaded. You must first unload a module before you can load it again.

Use the `-r` option to unload kernel modules:

```
sudo modprobe -rv nfs
```

```nocopybutton
rmmod /lib/modules/4.18.0-80.el8.x86_64/kernel/fs/nfs/nfs.ko
rmmod /lib/modules/4.18.0-80.el8.x86_64/kernel/fs/lockd/lockd.ko
rmmod /lib/modules/4.18.0-80.el8.x86_64/kernel/fs/fscache/fscache.ko
...
```

Modules are unloaded in reverse order in which they were first loaded. Modules aren't unloaded if a process or another loaded module requires them.

For more information, see the `modprobe(8)` and `modules.dep(5)` manual pages.

## About Module Parameters

To modify a module's behavior, specify parameters for the module in the `modprobe` command:

```
sudo modprobe *module\_name* *parameter*=*value* ...
```

Separate multiple parameter and value pairs with spaces. Array values are represented by a comma-separated list, for example:

```
sudo modprobe foo arrayparm=1,2,3,4
```

Alternatively, change the values of some parameters for loaded modules and built-in drivers by writing the new value to a file under `/sys/module/*module\_name*/parameters`, for example:

```
echo 0 | sudo tee /sys/module/ahci/parameters/skip_host_reset
```

Configuration files \(`/etc/modprobe.d/*.conf`\) specify module options, create module aliases, and override the usual behavior of `modprobe` for modules with special requirements. The `/etc/modprobe.conf` file that was used with earlier versions of `modprobe` is also valid if it exists. Entries in the `/etc/modprobe.conf` and `/etc/modprobe.d/*.conf` files use the same syntax.

The following are commonly used commands in `modprobe` configuration files:

- **`alias`**

  Creates an alternative name for a module. The alias can include shell wildcards. To create an alias for the `sd-mod` module:

  ```
  alias block-major-8-* sd_mod
  ```

- **`blacklist`**

  Ignore a module's internal alias that's displayed by the `modinfo` command. This command is typically used in the following conditions:

  - The associated hardware isn't required.

  - Two or more modules both support the same devices.

  - A module invalidly claims to support a device.

  For example, to demote the alias for the frame-buffer driver `cirrusfb`, type:

  ```
  blacklist cirrusfb
  ```

  The `/etc/modprobe.d/blacklist.conf` file prevents hotplug scripts from loading a module so that a different driver binds the module instead regardless of which driver happens to be probed first. If it doesn't already exist, you must create it.

- **`install`**

  Runs a shell command instead of loading a module into the kernel. For example, load the module `snd-emu10k1-synth` instead of `snd-emu10k1`:

  ```
  install snd-emu10k1 /sbin/modprobe --ignore-install snd-emu10k1 && /sbin/modprobe snd-emu10k1-synth
  ```

- **`options`**

  Defines options for a module. For exampe, to define the `nohwcrypt` and `qos` options for the `b43` module, type:

  ```
  options b43 nohwcrypt=1 qos=0
  ```

- **`remove`**

  Runs a shell command instead of unloading a module. To unmount `/proc/fs/nfsd` before unloading the `nfsd` module, type:

  ```
  remove nfsd { /bin/umount /proc/fs/nfsd > /dev/null 2>&1 || :; } ;
  /sbin/modprobe -r --first-time --ignore-remove nfsd
  ```

  For more information, see the `modprobe.conf(5)` manual page.

## Specifying Modules To Be Loaded at Boot Time

The system loads most modules automatically at boot time. You can also add modules to be loaded by creating a configuration file for the module in the `/etc/modules-load.d` directory. The file name must have the extension `.conf`.

For example to force the `bnxt_en.conf` to load at boot time, run the following command:

```
echo *bnxt\_en* | sudo tee /etc/modules-load.d/*bnxt\_en*.conf
```

Changes to the `/etc/modules-load.d` directory persist across reboots.

## Preventing Modules From Loading at Boot Time

You can prevent modules from loading at boot time by adding a deny rule in a configuration file in the `/etc/modprobe.d` directory and then rebuilding the initial ramdisk used to load the kernel at boot time.

1. Create a configuration file to prevent the module from loading. For example:

   ```
   sudo tee /etc/modprobe.d/*bnxt\_en*-deny.conf <<'EOF'
   #DENY *bnxt\_en*
   blacklist *bnxt\_en*
   install *bnxt\_en* /bin/false
   EOF
   ```

2. Rebuild the initial ramdisk image:

   ```
   sudo dracut -f -v
   ```

3. Reboot the system for the changes to take effect:

   ```
   sudo reboot
   ```

**Warning:**

Disabling modules can have unintended consequences and can prevent a system from booting properly or from being fully functional after boot. As a best practice, create a backup ramdisk image before making changes and ensure that the configuration is correct.

## About Weak Update Modules

External modules, such as drivers that are installed by using a driver update disk or that are installed from an independent package, are typically installed in the `/lib/modules/*kernel-version*/extra` directory. Modules that are stored in this directory are are preferred over any matching modules that are included with the kernel when these modules are being loaded. Installed external drivers and modules can override existing kernel modules to resolve hardware issues. For each kernel update, these external modules must be made available to each compatible kernel so that potential boot issues resulting from driver incompatibilities with the affected hardware can be avoided.

Because the requirement to load the external module with each compatible kernel update is system critical, a mechanism exists for external modules to be loaded as weak update modules for compatible kernels.

You make weak update modules available by creating symbolic links to compatible modules in the `/lib/modules/*kernel-version*/weak-updates` directory. The package manager handles this process automatically when it detects driver modules that are installed in the `/lib/modules/*kernel-version*/extra` directories for any compatible kernels.

For example, if a newer kernel is compatible with a module that was installed for the previous kernel, an external module \(such as `kmod-kvdo`\) is automatically added as a symbolic link in the `weak-updates` directory as part of the installation process, as shown in the following command output:

```
ls -l /lib/modules/4.18.0-80.el8.x86_64/weak-updates/kmod-kvdo/uds
```

```nocopybutton
lrwxrwxrwx. 1 root root 68 Oct  8 07:57 uds.ko -> 
/lib/modules/4.18.0-80.0.0.0.1.el8.x86_64/extra/kmod-kvdo/uds/uds.ko
```

```
ls -l /lib/modules/4.18.0-80.el8.x86_64/weak-updates/kmod-kvdo/vdo
```

```nocopybutton
lrwxrwxrwx. 1 root root 68 Oct  8 07:57 uds.ko -> 
/lib/modules/4.18.0-80.0.0.0.1.el8.x86_64/extra/kmod-kvdo/uds/uds.ko
```

Thesymbolic link enables the external module to loaded for kernel updates.

Weak updates are beneficial and ensure that no extra work is required to carry an external module through kernel updates. Any potential driver-related boot issues after kernel upgrades are prevented, thus provides a more predictable running of a system and its hardware.

In certain cases, you might remove weak update modules in place of a newer kernel, for example, in the case where an issue with a shipped driver has been resolved in a newer kernel. In this case, you might prefer to use the new driver rather than the external module that you installed as part of a driver update.

To remove weak update modules, use the following command:

```
rm -rf /lib/modules/*4.18.0-80.el8.x86\_64*/weak-updates/*kmod-kvdo/*
```

Running this command manually removes the symbolic links for each kernel.

Alternatively, you can use the `weak-modules` command, which safely removes the specified weak update module for the compatible kernels or the command removes the weak update modules for the current kernel. You can use the `weak-modules` command similarly to add weak update modules.

You can also use the `weak-modules` command with the `dry-run` option to test the results without making actual changes, for example:

```
weak-modules --remove-kernel --dry-run --verbose
rm -rf kmod-kvdo
```
