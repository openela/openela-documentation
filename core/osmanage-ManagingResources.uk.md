<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Managing Resources

This chapter describes how to manage the use of resources in an Enterprise Linux system.

## About Control Groups

Control groups \(`cgroups`\) are a collection of resources, such as CPU, memory, network, and so on, which have collective settings on how these resources are used by applications and processes. `Cgroups` constitute a functionality in Enterprise Linux that enables you to further manage the use of resources by setting limits, prioritizing, isolating, or allocating these resources. Thus, you can control resource use on a granular level to obtain a more efficient system performance. Control groups are important in system configurations that host multiple virtual machines, Kubernetes clusters, and so on, whose applications compete over resource use.

Enterprise Linux supports two types of control groups:

- **Control groups version 1 \(`cgroups v1`\)**

  These groups provide a per-resource controller hierarchy. Each resource, such as CPU, memory, I/O, and so on, has its own control group hierarchy. A disadvantage of this group is the difficulty of establishing proper coordination of resource use among groups that might belong to different process hierarchies.

- **Control groups version 2 \(`cgroups v2)`**

  These groups provide a single control group hierarchy against which all resource controllers are mounted. In this hierarchy, you can obtain better proper coordination of resource uses across different resource controllers. This version is an improvement over `cgroups v1` whose over flexibility prevented proper coordination of resource use among the system consumers.

Both versions are present in Enterprise Linux. However, by default, the `cgroups v2` functionality is enabled and mounted on Enterprise Linux 9 systems.

For more information about control groups of both versions, see the `cgroups(7)` and `sysfs(5)` manual pages.

## About Kernel Resource Controllers

Control groups manage resource use through _kernel resource controllers_. A kernel resource controller represents a single resource, such as CPU time, memory, network bandwidth, or disk I/O.

To identify mounted resource controllers in the system, check the contents of the `/procs/cgroups` file, for example:

```
sudo less /proc/cgroups
```

```nocopybutton
#subsys_name    hierarchy       num_cgroups     enabled
cpuset  0       103     1
cpu     0       103     1
cpuacct 0       103     1
blkio   0       103     1
memory  0       103     1
devices 0       103     1
freezer 0       103     1
net_cls 0       103     1
perf_event      0       103     1
net_prio        0       103     1
hugetlb 0       103     1
pids    0       103     1
rdma    0       103     1
misc    0       103     1
```

For a detailed explanation of the kernel resource controllers of both `cgroups v1` and `cgroups v2`, see the `cgrouops(7)` manual page.

## About the Control Group File System

In Enterprise Linux, the `cgroup` functionality is mounted as a file system in `/sys/fs/cgroup`. This directory is also called the root control group. The contents of the directory differ depending on which `cgroup` version is mounted on the system. For `cgroups v2`, the directory contents are as follows:

```
ls /sys/fs/cgroup
```

```nocopybutton
cgroup.controllers      cpuset.mems.effective  memory.stat
cgroup.max.depth        cpu.stat               misc.capacity
cgroup.max.descendants  dev-hugepages.mount    sys-fs-fuse-connections.mount
cgroup.procs            dev-mqueue.mount       sys-kernel-config.mount
cgroup.stat             init.scope             sys-kernel-debug.mount
cgroup.subtree_control  io.pressure            sys-kernel-tracing.mount
cgroup.threads          io.stat                system.slice
cpu.pressure            memory.numa_stat       user.slice
cpuset.cpus.effective   memory.pressure
```

To create a new control group, create a child group or subdirectory in the root control group.

```
sudo mkdir /sys/fs/cgroup/MyGroup
```

The child group is automatically populated with resource controllers that you have enabled for the new group. For sample procedures that create child groups where you can implement resource management for an application, see [Setting CPU Weight to Regulate Distribution of CPU Time](osmanage-ManagingResources.md#) and [Setting CPU Bandwidth to Regulate Distribution of CPU Time](osmanage-ManagingResources.md#).

To destroy a child group, ensure that the child group itself doesn't contain other child groups, then remove the directory from the root control group.

```
sudo rm -rf /sys/fs/cgroup/MyGroup
```

## About Control Groups and systemd

Control groups can be used by the `systemd` system and service manager for resource management. `Systemd` uses these groups to organize units and services that consume resources. For more information about `systemd`, see [About the systemd Service Manager](osmanage-WorkingWithSystemServices.md#).

`Systemd` supports different unit types, three of which are for resource control purposes:

- **Service**: A process or a group of processes whose settings are based on a unit configuration file. Services encompass specified processes in a "collection" so that `systemd` can start or stop the processes as one set. Service names follow the format `*name*.service`.

- **Scope**: A group of externally created processes, such as user sessions, containers, virtual machines, and so on. Similar to services, scopes encapsulate these created processes and are started or stopped by the arbitrary processes and then registered by `systemd` at runtime. Scope names follow the format `*name*.scope`.

- **Slice**: A group of hierarchically organized units in which services and scopes are located. Thus, slices themselves don't contain processes. Rather, the scopes and services in a slice define the processes. Every name of a slice unit corresponds to the path to a location in the hierarchy. Root slices, typically `user.slice` for all user-based processes and `system.slice` for system-based processes, are automatically created in the hierarchy. Parent slices exist immediately below the root slice and follow the format `*parent-name*.slice`. These root slices can then have subslices on multiple levels.

The service, the scope, and the slice units directly map to objects in the control group hierarchy. When these units are activated, they map directly to control group paths that are built from the unit names. To display the mapping between the `systemd` resource unit types and control groups, type:

```
sudo systemd-cgls
```

```nocopybutton
Working directory /sys/fs/cgroup:
├─user.slice (#1243)
│ → trusted.invocation_id: 50ce3909b2644f919ee420adc39edb4b
│ ├─user-1001.slice (#4167)
│ │ → trusted.invocation_id: 02e80a960d4549a7a9c69ce0fb546c26
│ │ ├─session-2.scope (#4405)
│ │ │ ├─2417 sshd: alice [priv]
│ │ │ ├─2430 sshd: alice@pts/0
│ │ │ ├─2431 -bash
│ │ │ ├─2689 sudo systemd-cgls
│ │ │ ├─2691 systemd-cgls
│ │ │ └─2692 less
...
│   └─user@984.service … (#3827)
│     → trusted.delegate: 1
│     → trusted.invocation_id: 09b47ce9f3124239b75814114353f3f2
│     └─init.scope (#3861)
│       ├─2058 /usr/lib/systemd/systemd --user
│       └─2099 (sd-pam)
├─init.scope (#19)
│ └─1 /usr/lib/systemd/systemd --switched-root --system --deserialize 17
└─system.slice (#53)
...
  ├─chronyd.service (#2467)
  │ → trusted.invocation_id: c0f77aaa9c7844e6bef6a6898ae4dd56
  │ └─1358 /usr/sbin/chronyd -F 2
  ├─auditd.service (#2331)
  │ → trusted.invocation_id: 756808add6a348609316c9e8c1801846
  │ └─1310 /sbin/auditd
  ├─tuned.service (#3079)
  │ → trusted.invocation_id: 2c358135fc46464d862b05550338d4f4
  │ └─1415 /usr/bin/python3 -Es /usr/sbin/tuned -l -P
  ├─systemd-journald.service (#1651)
  │ → trusted.invocation_id: 7cb7ccb14e044a899aadf47bbb583ada
  │ └─977 /usr/lib/systemd/systemd-journald
  ├─atd.service (#3623)
  │ → trusted.invocation_id: 597a7a4e5646468db407801b8562d869
  │ └─1915 /usr/sbin/atd -f
  ├─sshd.service (#3419)
  │ → trusted.invocation_id: 490504a683fc4311ab0fbeb0864a1a34
  │ └─1871 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
...
```

For an example of how to use `systemd` commands such as `systemctl` to manage resources, see [Controlling Access to System Resources](osmanage-WorkingWithSystemServices.md#). For further technical details, see the `systemctl(1)`, `systemd-cgls(1)`, and `systemd.resource-control(5)` manual pages.

## About Resource Distribution Models

The following distribution models provide you ways of implementing control or regulation in distributing resources for use by `cgroups v2`:

- **Weights**

  In this model, the weights of all the control groups are totaled. Each group receives a fraction of the resource based on the ratio of the group's weight against the total weight.

  Consider 10 control groups, each with a weight of 100 for a combined total of 1000. In this case, each group can use a tenth of a specified resource.

  Weight is typically used to distribute stateless resources. To apply this resource, the `CPUWeight` option is used.

- **Limits**

  To implement this distribution model, the `MemoryMax` option is used.

- **Protections**

  In this model, a group is assigned a _protected boundary_. If the group's resource usage remains within the protected amount, the kernel can't deprive the group of the use of the resource in favor of other groups that are competing for the same resource. In this model, an overcommitment of resources is allowed.

  To implement this model, the `MemoryLow` option is used.

- **Allocations**

  In this model, a specific absolute amount is allocated for the use of finite type of resources, such as real-time budget.

## Using cgroups v2 to Manage Resources for Applications

This section describes how to use `cgroups v2` to manage the resource use of applications and processes that are running in the system. The procedures focus on regulating CPU use by these applications so that CPU consumption becomes more efficient.

To prevent applications from overuse of CPU time, you can use `cgroups v2` to set limits so that the applications' use of the resource is better regulated. In the sections that follow, CPU resource control is implemented by using two methods:

- Setting CPU bandwidth

- Setting CPU weight

### Enabling cgroups v2

At boot time, Enterprise Linux 9 mounts `cgroups v2` by default.

1. Verify that `cgroups v2` is enabled and mounted on the system.

   ```
   sudo mount -l | grep cgroup
   ```

   ```nocopybutton
   cgroup2 on /sys/fs/cgroup type cgroup2 (rw,nosuid,nodev,noexec,relatime,seclabel,nsdelegate,memory_recursiveprot)
   ```

2. Optionally, check the contents of `/sys/fs/cgroup` directory, which is also called the root control group.

   ```
   ll /sys/fs/cgroup/
   ```

   For `cgroups v2`, the files in the directory should have prefixes to their file names, for example, `cgroup`.\*, `cpu`.\*, `memory`.\*, and so on. See [About the Control Group File System](osmanage-ManagingResources.md#).

### Preparing the Control Group for Distribution of CPU Time

1. Verify that in the root control group, the `cpu` and `cpuset` controllers are available in the `/sys/fs/cgroup/cgroup.controllers` file.

   ```
   sudo cat /sys/fs/cgroup/cgroup.controllers
   ```

   ```nocopybutton
   **cpuset****cpu** io memory hugetlb pids rdma misc
   ```

2. Add the CPU controllers to the `cgroup.subtree_control` file.

   By default, only the `memory` and `pids` controllers are in the file. To add the CPU controllers, type:

   ```
   echo "+cpu" | sudo tee /sys/fs/cgroup/cgroup.subtree_control
   echo "+cpuset" | sudo tee /sys/fs/cgroup/cgroup.subtree_control
   ```

3. Optionally, verify that the CPU controllers have been properly added.

   ```
   sudo cat /sys/fs/cgroup/cgroup.subtree_control
   ```

   ```nocopybutton
   cpuset cpu memory pids
   ```

4. Create a child group under the root control group to become the new control group for managing CPU resources on applications.

   ```
   sudo mkdir /sys/fs/cgroup/MyGroup
   ```

5. Optionally, list the contents of the new subdirectory or child group.

   ```
   ll /sys/fs/cgroup/MyGroup
   ```

   ```nocopybutton
   -r—​r—​r--. 1 root root 0 Jun  1 10:33 cgroup.controllers
   -r—​r—​r--. 1 root root 0 Jun  1 10:33 cgroup.events
   -rw-r—​r--. 1 root root 0 Jun  1 10:33 cgroup.freeze
   -rw-r—​r--. 1 root root 0 Jun  1 10:33 cgroup.max.depth
   -rw-r—​r--. 1 root root 0 Jun  1 10:33 cgroup.max.descendants
   -rw-r—​r--. 1 root root 0 Jun  1 10:33 cgroup.procs
   -r—​r—​r--. 1 root root 0 Jun  1 10:33 cgroup.stat
   -rw-r—​r--. 1 root root 0 Jun  1 10:33 cgroup.subtree_control
   …​
   -rw-r—​r--. 1 root root 0 Jun  1 10:33 cpuset.cpus
   -r—​r—​r--. 1 root root 0 Jun  1 10:33 cpuset.cpus.effective
   -rw-r—​r--. 1 root root 0 Jun  1 10:33 cpuset.cpus.partition
   -rw-r—​r--. 1 root root 0 Jun  1 10:33 cpuset.mems
   -r—​r—​r--. 1 root root 0 Jun  1 10:33 cpuset.mems.effective
   -r—​r—​r--. 1 root root 0 Jun  1 10:33 cpu.stat
   -rw-r—​r--. 1 root root 0 Jun  1 10:33 cpu.weight
   -rw-r—​r--. 1 root root 0 Jun  1 10:33 cpu.weight.nice
   …​
   -r—​r—​r--. 1 root root 0 Jun  1 10:33 memory.events.local
   -rw-r—​r--. 1 root root 0 Jun  1 10:33 memory.high
   -rw-r—​r--. 1 root root 0 Jun  1 10:33 memory.low
   …​
   -r—​r—​r--. 1 root root 0 Jun  1 10:33 pids.current
   -r—​r—​r--. 1 root root 0 Jun  1 10:33 pids.events
   -rw-r—​r--. 1 root root 0 Jun  1 10:33 pids.max
   ```

   Based on the CPU controllers that you added to `/sys/fs/cgroup/cgroup.subtree_control`, the contents of the `MyGroup` that are inherited from the root control group are now more limited. Thus, only `cpuset`.\*, `cpu`.\*, `memory`.\*, and `pids`.\* files are in the `MyGroup` directory.

6. Enable the CPU-related controllers in `MyGroup`'s `cgroup.subtre_control` files.

   ```
   echo "+cpu" | sudo tee /sys/fs/cgroup/MyGroup/cgroup.subtree_control
   echo "+cpuset" | sudo tee /sys/fs/cgroup/MyGroup/cgroup.subtree_control
   ```

7. Optionally, verify that the CPU controllers are enabled for child groups under `MyGroup`.

   ```
   sudo cat /sys/fs/cgroup/MyGroup/cgroup.subtree_control
   ```

   ```nocopybutton
   cpuset cpu
   ```

### Setting CPU Bandwidth to Regulate Distribution of CPU Time

This procedure is based on the following assumptions:

- The application that's consuming CPU resources excessively is `sha1sum`, as shown in the following sample output of the `top` command:

  ```
  sudo top
  ```

  ```nocopybutton
  ...
      PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
    **34500** root      20   0   18720   1756   1468 R  **99.0**   0.0   0:31.09 sha1sum
    **34501** root      20   0   18720   1772   1480 R  **99.0**   0.0   0:30.54 sha1sum
        1 root      20   0  109724  17196  11032 S   0.0   0.1   0:03.28 systemd                     
        2 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kthreadd                    
        3 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_gp                      
        4 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_par_gp                  
  ...
  ```

  In the sample output, the `sha1sum` processes have PIDs 34500 and 34501.

- The current system has multiple CPUs.

  To display the number of CPUs in the system, you can use the following command:

  ```
  sudo cat /sys/fs/cgroup/cpuset.cpus.effective
  ```

  ```nocopybutton
  0-1
  ```

  The sample output indicates a dual-core system.

**Important:**

As a prerequisite to the following procedure, you must complete the preparations of `cgroup-v2` as described in [Preparing the Control Group for Distribution of CPU Time](osmanage-ManagingResources.md#). If you skipped those preparations, you can't complete this procedure.

1. Create a `tasks` directory in the `MyGroup` subdirectory.

   ```
   sudo mkdir /sys/fs/cgroup/MyGroup/tasks
   ```

   This directory defines a child group with files that relate only to `cpu` and `cpuset` controllers.

2. Optionally, list the contents of the new subdirectory.

   ```
   ll /sys/fs/cgroup/MyGroup/tasks
   ```

   ```nocopybutton
   ll /sys/fs/cgroup/Example/tasks
   -r—​r—​r--. 1 root root 0 Jun  1 11:45 cgroup.controllers
   -r—​r—​r--. 1 root root 0 Jun  1 11:45 cgroup.events
   -rw-r—​r--. 1 root root 0 Jun  1 11:45 cgroup.freeze
   -rw-r—​r--. 1 root root 0 Jun  1 11:45 cgroup.max.depth
   -rw-r—​r--. 1 root root 0 Jun  1 11:45 cgroup.max.descendants
   -rw-r—​r--. 1 root root 0 Jun  1 11:45 cgroup.procs
   -r—​r—​r--. 1 root root 0 Jun  1 11:45 cgroup.stat
   -rw-r—​r--. 1 root root 0 Jun  1 11:45 cgroup.subtree_control
   -rw-r—​r--. 1 root root 0 Jun  1 11:45 cgroup.threads
   -rw-r—​r--. 1 root root 0 Jun  1 11:45 cgroup.type
   -rw-r—​r--. 1 root root 0 Jun  1 11:45 cpu.max
   -rw-r—​r--. 1 root root 0 Jun  1 11:45 cpu.pressure
   -rw-r—​r--. 1 root root 0 Jun  1 11:45 cpuset.cpus
   -r—​r—​r--. 1 root root 0 Jun  1 11:45 cpuset.cpus.effective
   -rw-r—​r--. 1 root root 0 Jun  1 11:45 cpuset.cpus.partition
   -rw-r—​r--. 1 root root 0 Jun  1 11:45 cpuset.mems
   -r—​r—​r--. 1 root root 0 Jun  1 11:45 cpuset.mems.effective
   -r—​r—​r--. 1 root root 0 Jun  1 11:45 cpu.stat
   -rw-r—​r--. 1 root root 0 Jun  1 11:45 cpu.weight
   -rw-r—​r--. 1 root root 0 Jun  1 11:45 cpu.weight.nice
   -rw-r—​r--. 1 root root 0 Jun  1 11:45 io.pressure
   -rw-r—​r--. 1 root root 0 Jun  1 11:45 memory.pressure
   ```

3. In the `tasks` directory, set the processes that you want to regulate for CPU time to use the same CPU.

   ```
   echo "1" | sudo tee /sys/fs/cgroup/MyGroup/tasks/cpuset.cpus
   ```

4. Optionally, verify that the processes run on the same CPU.

   ```
   cat /sys/fs/cgroup/MyGroup/tasks/cpuset.cpus
   ```

   ```nocopybutton
   1
   ```

5. Configure the CPU bandwidth to set restrictions within the `MyGroup/tasks` child control group.

   ```
   echo "200000 1000000"n | sudo tee /sys/fs/cgroup/MyGroup/tasks/cpu.max
   ```

   In the command, the value 20000 represents the quota of time in microseconds that's allowed for all processes collectively in a child group to run during a specified period. That period, in turn, is defined by the value 1000000. Specifically, the processes in the `/sys/fs/cgroup/MyGroup/tasks` group can run on the CPU for only 0.2 seconds, or one fifth, of every second.

   If the quota is exhausted by the control group within the defined period, then the processes are suspended until the next period.

6. Optionally, verify the time quotas.

   ```
   sudo cat /sys/fs/cgroup/MyGroup/tasks/cpu.max
   ```

   ```nocopybutton
   200000 1000000
   ```

7. Add the PIDs of the applications to the child group.

   ```
   echo "34500" | sudo tee /sys/fs/cgroup/MyGroup/tasks/cgroup.procs
   echo "34501" | sudo tee /sys/fs/cgroup/MyGroup/tasks/cgroup.procs
   ```

8. Verify that the applications are running in the specified control group.

   ```
   sudo cat /proc/34500/cgroup /proc/34501/cgroup
   ```

   ```nocopybutton
   0::/MyGroup/tasks
   0::/MyGroup/tasks
   ```

9. Check the current CPU consumption after you have set the CPU bandwidth.

   ```
   top
   ```

   ```nocopybutton
   ...
       PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
     34500 root      20   0   18720   1756   1468 R  10.0   0.0  37:36.13 sha1sum
     34501 root      20   0   18720   1772   1480 R  10.0   0.0  37:41.22 sha1sum
         1 root      20   0  186192  13940   9500 S   0.0   0.4   0:01.60 systemd
         2 root      20   0       0      0      0 S   0.0   0.0   0:00.01 kthreadd
         3 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_gp
         4 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_par_gp
   ...
   ```

   Because the `MyGroup/tasks` group is limited to a total of 20% of CPU use, then each `sha1sum` process is now limited to 10% of CPU time.

### Setting CPU Weight to Regulate Distribution of CPU Time

This procedure is based on the following assumptions:

- The application that's consuming CPU resources excessively is `sha1sum`, as shown in the following sample output of the `top` command:

  ```
  sudo top
  ```

  ```nocopybutton
  ...
      PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
    **33301** root      20   0   18720   1756   1468 R  **99.0**   0.0   0:31.09 sha1sum
    **33302** root      20   0   18720   1772   1480 R  **99.0**   0.0   0:30.54 sha1sum
    **33303** root      20   0   18720   1772   1480 R  **99.0**   0.0   0:30.54 sha1sum
        1 root      20   0  109724  17196  11032 S   0.0   0.1   0:03.28 systemd                     
        2 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kthreadd                    
        3 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_gp                      
        4 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_par_gp                  
  ...
  ```

  In the output, the `sha1sum` processes have PIDs 33301, 33302, and 33303.

- The current system has multiple CPUs.

  To display the number of CPUs in the system, type:

  ```
  sudo cat /sys/fs/cgroup/cpuset.cpus.effective
  ```

  ```nocopybutton
  0-1
  ```

  The sample output indicates a dual-core system.

**Important:**

As a prerequisite to the following procedure, you must complete the preparations of `cgroup-v2` as described in [Preparing the Control Group for Distribution of CPU Time](osmanage-ManagingResources.md#). If you skipped those preparations, you can't complete this procedure.

1. Create 3 child groups in the `MyGroup` subdirectory.

   ```
   sudo mkdir /sys/fs/cgroup/MyGroup/g1
   sudo mkdir /sys/fs/cgroup/MyGroup/g2
   sudo mkdir /sys/fs/cgroup/MyGroup/g3
   ```

2. Configure the CPU weight for each child group.

   ```
   echo "150" | sudo tee /sys/fs/cgroup/MyGroup/g1/cpu.weight
   echo "100" | sudo tee /sys/fs/cgroup/MyGroup/g2/cpu.weight
   echo "50" | sudo tee /sys/fs/cgroup/MyGroup/g3/cpu.weight
   ```

3. Apply the application PIDs to their corresponding child groups.

   ```
   echo "33301" | sudo tee /sys/fs/cgroup/Example/g1/cgroup.procs
   echo "33302" | sudo tee /sys/fs/cgroup/Example/g2/cgroup.procs
   echo "33303" | sudo /sys/fs/cgroup/Example/g3/cgroup.procs
   ```

   These commands set the selected applications to become members of the `MyGroup/g*/` control groups. The CPU time for each `sha1sum` process depends on the CPU time distribution as configured for each group.

   The weights of the `g1`, `g2`, and `g3` groups that have running processes are summed up at the level of `MyGroup`, which is the parent control group.

   With this configuration, when all processes run at the same time, the kernel allocates to each of the `sha1sum` processes the proportionate CPU time based on their respective `cgroup`'s `cpu.weight` file, as follows:

   | Child group | `cpu.weight` setting | Percent of CPU time allocation                        |
   | ----------- | -------------------- | ----------------------------------------------------- |
   | g1          | 150                  | ~50% \(150/300\) |
   | g2          | 100                  | ~33% \(100/300\) |
   | g3          | 50                   | ~16% \(50/300\)  |

   If one child group has no running processes, then the CPU time allocation for running processes is recalculated based on the total weight of the remaining child groups with running processes. For example, if the `g2` child group doesn't have any running processes, then the total weight becomes 200, which is the weight of `g1+g3`. In this case, the CPU time for `g1` becomes 150/200 \(~75%\) and for `g3`, 50/200 \(~25%\)

4. Check that the applications are running in the specified control groups.

   ```
   sudo cat /proc/33301/cgroup /proc/33302/cgroup /proc/33303/cgroup
   ```

   ```nocopybutton
   0::/MyGroup/g1
   0::/MyGroup/g2
   0::/MyGroup/g3
   ```

5. Check the current CPU consumption after you have set the CPU weights.

   ```
   top
   ```

   ```nocopybutton
   ...
       PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
     **33301** root      20   0   18720   1748   1460 R  **49.5**   0.0 415:05.87 sha1sum
     **33302** root      20   0   18720   1756   1464 R  **32.9**   0.0 412:58.33 sha1sum
     **33303** root      20   0   18720   1860   1568 R  **16.3**   0.0 411:03.12 sha1sum
       760 root      20   0  416620  28540  15296 S   0.3   0.7   0:10.23 tuned
         1 root      20   0  186328  14108   9484 S   0.0   0.4   0:02.00 systemd
         2 root      20   0       0      0      0 S   0.0   0.0   0:00.01 kthread
   ...
   ```

## Using cgroups v2 to Manage Resources for Users

The previous sample procedures describe how to manage applications' use of system resources. You can also manage resource use by directly implementing resource filters to users who log in to the system.
