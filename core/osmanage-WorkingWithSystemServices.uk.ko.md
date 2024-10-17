<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Managing System Services With systemd

The`systemd` daemon is the system initialization and service manager in Enterprise Linux. This chapter describes how to use `systemd`to manage system processes, services and `systemd` targets.

## About the systemd Service Manager

The `systemd` daemon is the first process that starts after a system boots and is the final process that's running when the system shuts down. `systemd` controls the final stages of booting and prepares the system for use. It also speeds up booting by loading services concurrently.

`systemd` reads its configuration from files in the `/etc/systemd` directory. For example, the `/etc/systemd/system.conf` file controls how `systemd` handles system initialization.

The `systemd` daemon starts services during the boot process by reading the symbolic link `/etc/systemd/system/default.target`. The following example shows the value of `/etc/systemd/system/default.target` on a system configured to boot to a multiuser mode without a graphical user interface, a target called `multi-user.target`:

```
sudo ls -l /etc/systemd/system/default.target
```

```nocopybutton
 /etc/systemd/system/default.target -> /usr/lib/systemd/system/multi-user.target 
```

**Note:**

You can use a kernel boot parameter to override the default system target. See [Kernel Boot Parameters](osmanage-WorkingWiththeGRUB2BootloaderandConfiguringBootServices.md#).

### systemd Units

`systemd` organizes the different types of resources it manages into units. Most units are configured in unit configuration files that enable you to configure these units according to system needs. In addition to the files, you can also use `systemd` runtime commands to configure the units.

The following list describes some system units that you can manage on an Enterprise Linux system by using `systemd`:

- **Services**

  Service unit configuration files have the filename format _service\_name_.`service`, for example `sshd.service`, `crond.service`, and `httpd.service`.

  Service units start and control daemons and the processes of which the daemons consist.

  The following example shows how you might start the `systemd` service unit for the Apache HTTP server, `httpd.service`:

  ```
  sudo systemctl start httpd.service
  ```

- **Targets**

  Target unit configuration files have the filename format _target\_name_.`target`, for example `graphical.target`.

  Targets are similar to runlevels. A system reaches different targets during the boot process as resources get configured. For example, a system reaches `network-pre.target` before it reaches the target `network-online.target`.

  Many target units have dependencies. For example, the activation of`graphical.target` \(for a graphical session\) fails unless `multi-user.target` \(for multiuser system\) is also active.

- **File System Mount Points**

  Mount unit configuration files have the filename format _mount\_point\_name_.`mount`.

  Mount units enable you to mount filesystems at boot time. For example, you can run the following command to mount the temporary file system \(`tmpfs`\) on `/tmp` at boot time:

  ```
  sudo systemctl enable tmp.mount
  ```

- **Devices**

  Device unit configuration files have the filename format _device\_unit\_name_.`device`.

  Device units are named after the `/sys` and `/dev` paths they control. For example, the device `/dev/sda5` is exposed in systemd as `dev-sda5.device`.

  Device units enable you to implement device-based activation.

- **Sockets**

  Socket unit configuration files have the filename format _socket\_unit\_name_.`socket`.

  Each "\*.`socket`" file needs a corresponding "\*.`service`" file to configure the service to start on incoming traffic on the socket.

  Socket units enable you to implement socket-based activation.

- **Timers**

  Timer unit configuration files have the filename format _timer\_unit\_name_.`timer`.

  Each "\*.`timer`" file needs a corresponding "\*.`service`" file to configure the service to start at a configured timer event. A `Unit` configuration entry can be used to specify a service that's named differently to the timer unit, if required.

  Timer units can control when service units are run and can act as an alternative to using the cron daemon. Timer units can be configured for calendar time events, monotonic time events, and can be run asynchronously.

Paths to `systemd` unit configuration files vary depending on their purpose and whether `systemd` is running in 'user' or 'system' mode. For example, configuration for units that are installed from packages might be available in `/usr/lib/systemd/system` or in `/usr/local/lib/systemd/system`, while a user mode configuration unit is likely to be stored in `$HOME/.config/systemd/user`. See the `systemd.unit(5)` manual page for more information.

See [About System-State Targets](osmanage-WorkingWithSystemServices.md#).

## About System-State Targets

By using system-state targets, you can control `systemd` so that it starts only the services that are required for a specific purpose. For example, you set the default target to `multi-user.target` on a production server so that the graphical user interface isn't used when the system boots. In a case where you need to troubleshoot or perform diagnostics, you might consider setting the target to `rescue.target`, where only `root` logs onto the system to run the minimum number of services.

Each run level defines the services that `systemd` stops or starts. As an example, `systemd` starts network services for `multi-user.target` and the X Window System for `graphical.target`, and stops both services for `rescue.target`.

[Table 1](osmanage-WorkingWithSystemServices.md#ol-systemctltgt) shows the commonly used system-state targets and the equivalent runlevel targets.

<table><thead><tr><th>

System-State Targets

</th><th>

Equivalent Runlevel Targets

</th><th>

Description

</th></tr></thead><tbody><tr><td>

`graphical.target`

</td><td>

`runlevel5.target`

</td><td>

Set up a multiuser system with networking and display manager.

</td></tr><tr><td>

`multi-user.target`

</td><td>

`runlevel2.target`

`runlevel3.target`

`runlevel4.target`

</td><td>

Set up a nongraphical multiuser system with networking.

</td></tr><tr><td>

`poweroff.target`

</td><td>

`runlevel0.target`

</td><td>

Shut down and power off the system.

</td></tr><tr><td>

`reboot.target`

</td><td>

`runlevel6.target`

</td><td>

Shut down and reboot the system.

</td></tr><tr><td>

`rescue.target`

</td><td>

`runlevel1.target`

</td><td>

Set up a rescue shell.

</td></tr><tbody></table>
Note that `runlevel*` targets are implemented as symbolic links.

For more information, see the `systemd.target(5)` manual page.

### Displaying Default and Active System-State Targets

To display the default system-state target, use the `systemctl get-default` command:

```
sudo systemctl get-default
```

```nocopybutton
graphical.target
```

To display the active targets on a system, use the `systemctl list-units --type target` command:

```
sudo systemctl list-units --type target [--all]
```

```nocopybutton
UNIT                   LOAD   ACTIVE SUB    DESCRIPTION                
basic.target           loaded active active Basic System               
cryptsetup.target      loaded active active Local Encrypted Volumes    
getty.target           loaded active active Login Prompts              
graphical.target       loaded active active Graphical Interface        
local-fs-pre.target    loaded active active Local File Systems (Pre)   
local-fs.target        loaded active active Local File Systems         
multi-user.target      loaded active active Multi-User System          
network-online.target  loaded active active Network is Online          
network-pre.target     loaded active active Network (Pre)              
network.target         loaded active active Network                    
nfs-client.target      loaded active active NFS client services        
nss-user-lookup.target loaded active active User and Group Name Lookups
paths.target           loaded active active Paths                      
remote-fs-pre.target   loaded active active Remote File Systems (Pre)  
remote-fs.target       loaded active active Remote File Systems        
rpc_pipefs.target      loaded active active rpc_pipefs.target          
rpcbind.target         loaded active active RPC Port Mapper            
slices.target          loaded active active Slices                     
sockets.target         loaded active active Sockets                    
sound.target           loaded active active Sound Card                 
sshd-keygen.target     loaded active active sshd-keygen.target         
swap.target            loaded active active Swap                       
sysinit.target         loaded active active System Initialization      
timers.target          loaded active active Timers                     

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.

24 loaded units listed. Pass --all to see loaded but inactive units, too.
To show all installed unit files use 'systemctl list-unit-files'.
```

The output for a system with the `graphical` target active shows that this target depends on other active targets, including `network` and `sound` to support networking and sound.

Use the `--all` option to include inactive targets in the list.

For more information, see the `systemctl(1)` and `systemd.target(5)` manual pages.

**Note:**

Target is only one of `systemd` types of units. To display all the types of units, use the following command:

```
sudo systemctl -t help
```

```nocopybutton
Available unit types:
service
mount
swap
socket
target
device
automount
timer
path
slice
scope
```

### Changing Default and Active System-State Targets

Use the `systemctl set-default` command to change the default system-state target:

```
sudo systemctl set-default multi-user.target
```

```nocopybutton
Removed /etc/systemd/system/default.target. 
Created symlink /etc/systemd/system/default.target → /usr/lib/systemd/system/multi-user.target
```

**Note:**

This command changes the target to which the default target is linked, but doesn't change the state of the system.

To change the current active system target, use the `systemctl isolate` command, for example:

```
sudo systemctl isolate multi-user.target
```

For more information, see the `systemctl(1)` manual page.

## Shutting Down, Suspending, and Rebooting the System

<table><thead><tr><th>

systemctl Command

</th><th>

Description

</th></tr></thead><tbody><tr><td>

`systemctl halt`

</td><td>

Halt the system.

</td></tr><tr><td>

`systemctl hibernate`

</td><td>

Put the system into hibernation.

</td></tr><tr><td>

`systemctl hybrid-sleep`

</td><td>

Put the system into hibernation and suspend its operation.

</td></tr><tr><td>

`systemctl poweroff`

</td><td>

Halt and power off the system.

</td></tr><tr><td>

`systemctl reboot`

</td><td>

Reboot the system.

</td></tr><tr><td>

`systemctl suspend`

</td><td>

Suspend the system.

</td></tr><tbody></table>
For more information, see the `systemctl(1)` manual page.

## Managing Services

Services in an Enterprise Linux system are managed by the `systemctl *subcommand*` command.

Examples of subcommands are `enable`, `disable`, `stop`, `start`, `restart`, reload, and `status`.

For more information, see the `systemctl(1)` manual page.

### Starting and Stopping Services

To start a service, use the `systemctl start` command:

```
sudo systemctl start *sshd*
```

To stop a service, use the `systemctl stop` command:

```
sudo systemctl stop *sshd*
```

Changing the state of a service only lasts while the system remains at the same state. If you stop a service and then change the system-state target to one in which the service is configured to run \(for example, by rebooting the system\), the service restarts. Similarly, starting a service doesn't enable the service to start following a reboot. See [Enabling and Disabling Services](osmanage-WorkingWithSystemServices.md#).

### Enabling and Disabling Services

You can use the `systemctl` command to enable or disable a service from starting when the system boots, for example:

```
sudo systemctl enable *httpd*
```

```nocopybutton
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
```

The `enable` command activates a service by creating a symbolic link for the lowest-level system-state target at which the service should start. In the previous example, the command creates the symbolic link `httpd.service` for the `multi-user` target.

Disabling a service removes the symbolic link:

```
sudo systemctl disable *httpd*
```

```nocopybutton
Removed /etc/systemd/system/multi-user.target.wants/httpd.service.
```

To check whether a service is enabled, use `is-enabled` subcommand as shown in the following examples:

```
sudo systemctl is-enabled *httpd*
```

```nocopybutton
disabled
```

```
sudo systemctl is-enabled *sshd*
```

```nocopybutton
enabled
```

After running the `systemctl disable` command, the service can still be started or stopped by user accounts, scripts, and other processes. However, if you need to ensure that the service might be started inadvertently, for example, by a conflicting service, then use the `systemctl mask` command as follows:

```
sudo systemctl mask *httpd*
```

```nocopybutton
Created symlink from '/etc/systemd/system/multi-user.target.wants/httpd.service' to '/dev/null'
```

The `mask` command sets the the service reference to `/dev/null`. If you try to start a service that has been masked, you will receive an error as shown in the following example:

```
sudo systemctl start *httpd*
```

```nocopybutton
Failed to start httpd.service: Unit is masked.
```

To relink the service reference back to the matching service unit configuration file, use the `systemctl unmask` command:

```
sudo systemctl unmask *httpd*
```

For more information, see the `systemctl(1)` manual page.

### Displaying the Status of Services

To check whether a service is running, use the `is-active` subcommand. The output would either be _active_\) or _inactive_, as shown in the following examples:

```
sudo systemctl is-active *httpd*
```

```nocopybutton
active
```

```
systemctl is-active *sshd*
```

```nocopybutton
inactive
```

The `status` subcommand provides a detailed summary of the status of a service, including a tree that displays the tasks in the control group \(`CGroup`\) that the service implements:

```
sudo systemctl status *httpd*
```

```nocopybutton
httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since ...
     Docs: man:httpd.service(8)
 Main PID: 11832 (httpd)
   Status: "Started, listening on: port 80"
    Tasks: 213 (limit: 26213)
   Memory: 32.5M
   CGroup: /system.slice/httpd.service
           ├─11832 /usr/sbin/httpd -DFOREGROUND
           ├─11833 /usr/sbin/httpd -DFOREGROUND
           ├─11834 /usr/sbin/httpd -DFOREGROUND
           ├─11835 /usr/sbin/httpd -DFOREGROUND
           └─11836 /usr/sbin/httpd -DFOREGROUND

Jul 17 00:14:32 Unknown systemd[1]: Starting The Apache HTTP Server...
Jul 17 00:14:32 Unknown httpd[11832]: Server configured, listening on: port 80
Jul 17 00:14:32 Unknown systemd[1]: Started The Apache HTTP Server.
```

A `cgroup` is a collection of processes that are bound together so that you can control their access to system resources. In the example, the `cgroup` for the `httpd` service is `httpd.service`, which is in the `system` slice.

Slices divide the `cgroups` on a system into different categories. To display the slice and `cgroup` hierarchy, use the `systemd-cgls` command:

```
sudo systemd-cgls
```

```
Control group /:
-.slice
├─user.slice
│ └─user-1000.slice
│   ├─user@1000.service
│   │ └─init.scope
│   │   ├─6488 /usr/lib/systemd/systemd --user
│   │   └─6492 (sd-pam)
│   └─session-7.scope
│     ├─6484 sshd: root [priv]
│     ├─6498 sshd: root@pts/0
│     ├─6499 -bash
│     ├─6524 sudo systemd-cgls
│     ├─6526 systemd-cgls
│     └─6527 less
├─init.scope
│ └─1 /usr/lib/systemd/systemd --switched-root --system --deserialize 16
└─system.slice
  ├─rngd.service
  │ └─1266 /sbin/rngd -f --fill-watermark=0
  ├─irqbalance.service
  │ └─1247 /usr/sbin/irqbalance --foreground
  ├─libstoragemgmt.service
  │ └─1201 /usr/bin/lsmd -d
  ├─systemd-udevd.service
  │ └─1060 /usr/lib/systemd/systemd-udevd
  ├─polkit.service
  │ └─1241 /usr/lib/polkit-1/polkitd --no-debug
  ├─chronyd.service
  │ └─1249 /usr/sbin/chronyd
  ├─auditd.service
  │ ├─1152 /sbin/auditd
  │ └─1154 /usr/sbin/sedispatch
  ├─tuned.service
  │ └─1382 /usr/libexec/platform-python -Es /usr/sbin/tuned -l -P
  ├─systemd-journald.service
  │ └─1027 /usr/lib/systemd/systemd-journald
  ├─atd.service
  │ └─1812 /usr/sbin/atd -f
  ├─sshd.service
  │ └─1781 /usr/sbin/sshd
```

The `system.slice` contains services and other system processes. `user.slice` contains user processes, which run within transient cgroups called _scopes_. In the example, the processes for the user with ID 1000 are running in the scope `session-7.scope` under the slice `/user.slice/user-1000.slice`.

You can use the `systemctl` command to limit the CPU, I/O, memory, and other resources that are available to the processes in service and scope cgroups. See [Controlling Access to System Resources](osmanage-WorkingWithSystemServices.md#).

For more information, see the `systemctl(1)` and `systemd-cgls(1)` manual pages.

### Controlling Access to System Resources

Use the `systemctl` command to control a cgroup's access to system resources, for example:

```
sudo systemctl [--runtime] set-property *httpd* CPUShares=512 MemoryLimit=1G
```

`CPUShare` controls access to CPU resources. As the default value is 1024, a value of 512 halves the access to CPU time that the processes in the `cgroup` have. Similarly, `MemoryLimit` controls the maximum amount of memory that the `cgroup` can use.

**Note:**

You don't need to specify the `.service` extension to the name of a service.

If you specify the `--runtime` option, the setting doesn't persist across system reboots.

Alternatively, you can change the resource settings for a service under the `[Service]` heading in the service's configuration file in `/usr/lib/systemd/system`. After editing the file, make `systemd` reload its configuration files and then restart the service:

```
sudo systemctl daemon-reload
sudo systemctl restart *service*
```

You can run general commands within scopes and use `systemctl` to control the access that these transient cgroups have to system resources. To run a command within in a scope, use the `systemd-run` command:

```
sudo systemd-run --scope --unit=*group\_name* [--slice=*slice\_name*]
```

If you don't want to create the group under the default `system` slice, you can specify another slice or the name of a new slice. The following example runs a command named `mymonitor` in `mymon.scope` under `myslice.slice`:

```
sudo systemd-run --scope --unit=*mymon* --slice=*myslice* mymonitor
```

```nocopybutton
Running as unit mymon.scope.
```

**Note:**

If you don't specify the `--scope` option, the control group is a created as a service rather than as a scope.

You can then use `systemctl` to control the access that a scope has to system resources in the same way as for a service. However, unlike a service, you must specify the `.scope` extension, for example:

```
sudo systemctl --runtime set-property *mymon.scope* CPUShares=256
```

For more information see the `systemctl(1)`, `systemd-cgls(1)`, and `systemd.resource-control(5)` manual pages.

### Running systemctl on a Remote System

If the `sshd` service is running on a remote Enterprise Linux system, specify the `-H` option with the `systemctl` command to control the system remotely, for example:

```
sudo systemctl -H root@10.0.0.2 status sshd
```

```nocopybutton
root@10.0.0.2's password: *password*
sshd.service - OpenSSH server daemon
   Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled)
   Active: active (running) since ...
  Process: 1498 ExecStartPre=/usr/sbin/sshd-keygen (code=exited, status=0/SUCCESS)
 Main PID: 1524 (sshd)
   CGroup: /system.slice/sshd.service
```

For more information see the `systemctl(1)` manual page.

## Modifying systemd Service Unit Files

To change the configuration of `systemd` services, copy the files with `.service`, `.target`, `.mount` and `.socket` extensions from `/usr/lib/systemd/system` to `/etc/systemd/system`.

After you have copied the files, you can edit the versions in`/etc/systemd/system`. The files in `/etc/systemd/system` take precedence over the versions in `/usr/lib/systemd/system`. Files in `/etc/systemd/system` aren't overwritten when you update a package that touches files in `/usr/lib/systemd/system`.

To revert to the default `systemd` configuration for a particular service, you can either rename or delete the copies in `/etc/systemd/system`.

The following sections describe the different parts of a service unit file that you cand edit and customize for a system.

### About Service Unit Files

Services run based on their corresponding service unit files. A service unit file typically contains the following sections, with each section having its respective defined options that determine how a specific service runs:

- **`[Unit]`**

  Contains information about the service.

- **`[_UnitType_]`:**

  Contains options that are specific to the unit type of the file. For example, in a service unit file this section is titled `[Service]` and contains options that are specific to units of the service type, such as `ExecStart` or `StandardOutput`.

  Only those unit types that offer options specific to their type have such a section.

- **`[Install]`**

  Contains installation information for the specific unit. The information in this section is used by the `systemctl enable` and `systemctl disable` commands.

A service unit file might contain the following configurations for a service.

```nocopybutton
[Unit]
Description=A test service used to develop a service unit file template

[Service]
Type=simple
StandardOutput=journal
ExecStart=/usr/lib/systemd/helloworld.sh

[Install]
WantedBy=default.target
```

[Configurable Options in Service Unit Files](osmanage-WorkingWithSystemServices.md#) describes some commonly used configured options available under each section. A complete list is also available in the `systemd.service(5)` and `systemd.unit(5)` manual pages.

### Configurable Options in Service Unit Files

Each of the following lists deals with a separate section of the service unit file.

#### Description of Options Under \[Unit\] Section

The following list provides a general overview of the commonly used configurable options available in the `[Unit]` section of service unit file:

- **`Description`**

  Provides information about the service. The information is displayed when you run the `systemctl status` command on the unit.

- **`Documentation`**

  Contains a space-separated list of URIs referencing documentation for this unit or its configuration.

- **`After`**

  Configures the unit to only run after the units listed in the option finish starting up.

  In the following example, if the file _var3_.`service` has the following entry, then it's only started after units `*var1*.service` and `*var2*.service` have started:

  ```
   After=*var1*.service *var2*.service
  ```

- **`Requires`**

  Configures a unit to have requirement dependencies on other units. If a unit is activated, those listed in its `Requires` option are also activated.

- **`Wants`**

  A less stringent version of the `Requires` option. For example, a specific unit can be activated even if one of those listed in its `Wants` option fails to start.

#### Description of Options Under \[Service\] Section

This following list gives a general overview of the commonly used configurable options available in the `[Service]` section of a service unit file.

- **`Type`**

  Configures the process start-up type for the service unit.

  By default, this parameter's value is `simple`, which indicates that the service's main process is that which is started by the `ExecStart` parameter.

  Typically, if a service's type is `simple`, then the definition can be omitted from the file.

- **`StandardOutput`**

  Configures the how the service's events are logged. For example, consider a service unit file has the following entry:

  ```
  StandardOutput=journal
  ```

  In the example, the value `journal` indicates that the events are recorded in the journal, which can be viewed by using the `journalctl` command.

- **`ExecStart`**

  Specifies the full path and command that starts the service, for example, `/usr/bin/npm start`.

- **`ExecStop`**

  Specifies the commands to run to stop the service started through `ExecStart`.

- **`ExecReload`**

  Specifies the commands to run to trigger a configuration reload in the service.

- **`Restart`**

  Configures whether the service is to be restarted when the service process exits, is stopped, or when a timeout is reached.

  **Note:** This option doesn't apply when the process is stopped cleanly by a `systemd` operation, for example a `systemctl stop` or `systemctl restart`. In these cases, the service isn't restarted by this configuration option.

- **`RemainAfterExit`**

  A Boolean value that configures whether the service is to be considered active even when all of its processes have exited. The default value is `no`.

#### Description of Options Under \[Install\] Section

This following list gives a general overview of the commonly used configurable options available in the `[Install]` section of service unit file.

- **`Alias`**

  A space-separated list of names for a unit.

  At installation time, `systemctl enable` creates symlinks from these names to the unit filename.

  Aliases are only effective when the unit is enabled.

- **`RequiredBy`**

  Configures the service to be required by other units.

  For example, consider a unit file `*var1*.service` that has the following configuration added to it:

  ```
  RequiredBy=*var2*.service *var3*.service
  ```

  When `*var1*.service` is enabled, both `*var2*.service` and `*var3*.service` are granted a `Requires` dependency upon `*var1*.service`. This dependency is defined by a symbolic link that's created in the `.requires` folder of each dependent service \(`*var2*.service` and `var3.service`\) that points to the `*var1*.service` system unit file.

- **`WantedBy`**

  Specifies a list of units that are to be granted a `wants` dependency upon the service whose file you're editing.

  For example, consider a unit file `*var1*.service` that has the following configuration added to it:

  ```
  WantedBy=*var2*.service *var3*.service
  ```

  When `*var1*.service` is enabled, both `*var2*.service`and `*var3*.service` are granted a `Wants` dependency upon `*var1*.service`. This dependency is defined by a symbolic link that's created in the “`.wants`” folder of each dependent service \(`*var2*.service` and `var3.service`\) that points to the system unit file for `*var1*.service` .

- **`Also`**

  Lists additional units to install or remove when the unit is installed or removed.

- **`DefaultInstance`**

  The `DefaultInstance` option applies to template unit files only.

  Template unit files enable the creation of multiple units from a single configuration file. The `DefaultInstance` option specifies the instance for which the unit is enabled if the template is enabled without any explicitly set instance.

## Creating a User-Based systemd Service

In addition to the system-wide `systemd` files, `systemd` enables you to create user-based services that you can run from a user level without requiring root access and privileges. These user-based services are under user control and are configurable independent of system services.

The following are some distinguishing features of user-based `systemd` services:

- User-based `systemd` services are linked with a specific user account.
- They're created under the associated user’s home directory in `$HOME/.config/systemd/user/`.
- After these services are enabled, they start when the associated user logs in. This behavior differs from that of enabled `systemd` services which start when the system boots.

To create a user based service:

1. Create the service's unit file in the `~/.config/systemd/user` directory, for example:

   ```
   touch ~/.config/systemd/user/*myservice*.service
   ```

2. Open the unit file and specify the values to the options you want to use, such as `Description`, `ExecStart`, `WantedBy`, and so on.

   For reference, see [Configurable Options in Service Unit Files](osmanage-WorkingWithSystemServices.md#) and the `systemd.service(5)` and `systemd.unit(5)` manual pages.

3. Enable the service to start automatically when you log in.

   ```
   sudo systemctl --user enable *myservice*.service
   ```

   **Note:**

   When you log out, the service is stopped unless the root user has enabled processes to continue to run for the user.

4. Start the service.

   ```
   sudo systemctl --user start *myservice*.service
   ```

5. Verify that the service is running.

   ```
   sudo systemctl --user status *myservice*.service
   ```

## Using Timer Units to Control Service Unit Runtime

Timer units can be configured to control when service units run. You can use timer units instead of configuring the `cron` daemon for time-based events. Timer units can be more complicated to configure than creating a crontab entry. However, timer units are more configurable and the services that they control can be configured for better logging and deeper integration with `systemd` architecture.

Timer units are started, enabled, and stopped similarly to service units. For example, to enable and start a timer unit immediately, type:

```
sudo systemctl enable --now *myscript*.timer
```

To list all existing timers on the system, to see when they last ran, and when they're next configured to run, type:

```
systemctl list-timers
```

For more information about system timers, see the `systemd.timer(5)` and `systemd.time(7)` manual pages.

### Configuring a Realtime Timer Unit

**Realtime timers** activate on a calendar event, similar to events in a crontab. The option `OnCalendar` specifies when the timer runs a service.

- If needed, create a `.service` file that defines the service to be triggered by the timer unit. In the following procedure, the sample service is `/etc/systemd/system/update.service` which is a service unit that runs an update script.

  For more information about creating service units, see [Creating a User-Based systemd Service](osmanage-WorkingWithSystemServices.md#).

- Decide the time and frequency for running the service. In this procedure, the timer is configured to run the service every 2 hours from Monday to Friday.

This task shows you how to create a system timer to trigger a service to run based on a calendar event. The definition of the calendar event is similar to entries that you put in a cron job.

1. Create the `/etc/systemd/system/update.timer` with the following content:

   ```
   [Unit]
   Description="Run the update.service every two hours from Mon to Fri."

   [Timer]
   OnCalendar=Mon..Fri 00/2 
   Unit=update.service

   [Install]
   WantedBy=multi-user.target
   ```

   Defining `OnCalendar` can vary from a simple wetting such as `OnCalendar=weekly` definitions that are more detailed. However, the format for defining settings is constant, as follows:

   ```
   DayofWeek Year-Month-Day Hour:Minute:Second
   ```

   The following definition means "the first 4 days of each month at 12:00 o'clock noon, but only if that day is either a Monday or a Tuesday":

   ```
   OnCalendar=Mon,Tue *-*-01..04 12:00:00
   ```

   For other ways to define `OnCalendar` and for more timer options that you can configure in the system timer file, see the `systemd.timer(5)` and `systemd.time(7)` manual pages.

2. Check that all the files related to this timer are configured correctly.

   ```
   systemd-analyze verify /etc/systemd/system/update.*
   ```

   Any detected errors are reported on the screen.

3. Start the timer.

   ```
   sudo systemctl start update.timer
   ```

   This command starts the timer for the current session only.

4. Ensure that the timer starts when the system is booted.

   ```
   sudo systemctl enable update.timer
   ```

### Configuring a Monotonic Timer Unit

**Monotonic timers** that activate after a time span relative to a varying starting point, such as a boot event, or when a particular `systemd` unit becomes active. These timer units stop if the computer is temporarily suspended or shut down. Monotonic timers are configured by using the `On*Type*Sec` option, where _Type_ is the name of the event to which the timer is related. Common monotonic timers include `OnBootSec` and `OnUnitActiveSec`.

- If needed, create a `.service` file that defines the service to be triggered by the timer unit. In the following procedure, the sample service is `/etc/systemd/system/update.service` which is a service unit that runs an update script.

  For more information about creating service units, see [Creating a User-Based systemd Service](osmanage-WorkingWithSystemServices.md#).

- Decide the time and frequency for running the service. In this procedure, the timer is configured to run the service 10 minutes after a system boot, and every 2 hours from when the service is last activated.

This task shows you how to create a system timer to trigger a service to run at specific events, which are when the system boots or after 2 hours have lapsed from the timer's activation.

1. Create the `/etc/systemd/system/update.timer` with the following content:

   ```
   [Unit]
   Description="Run the update.service every two hours from Mon to Fri."

   [Timer]
   OnBootSec=10min
   OnUnitActiveSec=2h
   Unit=update.service

   [Install]
   WantedBy=multi-user.target
   ```

   For more timer options that you can configure in the system timer, see the `systemd.timer(5)` and `systemd.time(7)` manual pages.

2. Check that all the files related to this timer are configured correctly.

   ```
   systemd-analyze verify /etc/systemd/system/update.*
   ```

   Any detected errors are reported on the screen.

3. Start the timer.

   ```
   sudo systemctl start update.timer
   ```

   This command starts the timer for the current session only.

4. Ensure that the timer starts when the system is booted.

   ```
   sudo systemctl enable update.timer
   ```

### Running a Transient Timer Unit

Transient timers are temporary timers that are valid only for the current session. These timers can be created to run a program or script directly without requiring service or timer units to be configured within `systemd`. These units are generated by using the `systemd-run` command. See the `systemd-run(1)` manual page for more information.

The parameter options that you would add to the `*unit-file*.timer` file also serve as arguments when you use `systemd-run` command to run a transient timer unit.

The following examples show how to use `systemd-run` to activate transient timers.

- Run `update.service` after 2 hours have elapsed.

  ```
  sudo systemd-run --on-active="2h" --unit update.service
  ```

- Create `~/tmp/myfile` after 1 hour.

  ```
  sudo systemd-run --on-active="1h" /bin/touch ~/tmp/myfile
  ```

- Run `~/myscripts/update.sh` 5 minutes after the service manager is started. Use this syntax to run a service after the service manager has started at user login.

  ```
  sudo systemd-run --on-startup="5m" ~/myscripts/update.sh
  ```

- Run `myjob.service` 10 minutes after system boot.

  ```
  sudo systemd-run --on-boot="10m" --unit myjob.service
  ```

- Run `report.service` at the end of the day.

  ```
  sudo systemd-run --on-calendar="17:00:00"
  ```
