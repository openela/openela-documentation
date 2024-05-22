<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->
# 시스템 설정 구성

이 장에서는 시스템의 구성 설정을 변경하는 데 사용할 수 있는 파일 및 가상 파일 시스템에 대해 설명합니다.

## /etc/sysconfig 디렉토리의 정보

`/etc/sysconfig` 디렉토리에는 시스템 구성을 제어하는 파일이 포함되어 있습니다. 이 디렉터리의 내용은 시스템에 설치한 패키지에 따라 다릅니다.

`/etc/sysconfig` 디렉토리에서 찾을 수 있는 특정 파일은 다음과 같습니다.

-   **`atd`**

    `atd` 데몬에 대한 커맨드라인 파라메터를 지정합니다.

-   **`crond`**

    부팅 시 `crond` 데몬에 파라메터를 전달합니다.

-   **`chronyd`**

    부팅 시 NTP 서비스에 사용되는 `chronyd` 데몬에 파라메터를 전달합니다.

-   **`firewalld`**

    부팅 시 방화벽 데몬(`firewalld`)에 파라메터를 전달합니다.

-   **`named`**

    부팅 시 이름(name) 서비스 데몬에 파라메터를 전달합니다. `named` 데몬몬 BIND(Berkeley Internet Name Domain) 배포의 일부인 DNS(Domain Name System) 서버입니다. 이 서버는 호스트 이름을 네트워크의 IP 주소와 연결하는 테이블을 유지 관리합니다.

-   **`samba`**

    Windows 클라이언트에 대한 파일 공유 연결, NetBIOS-over-IP 이름(name) 지정 서비스 및 도메인 컨트롤러에 대한 연결 관리를 지원하기 위해 부팅 시 `smbd`, `nmbd` 및 `winbindd` 데몬에 파라메터를 전달합니다.

-   **`selinux`**

    시스템의 SELinux 상태를 제어합니다. 이 파일은 `/etc/selinux/config`에 대한 심볼릭 링크입니다.

-   **`snapper`**

    `snapper` 유틸리티의 목록을 정의합니다.

-   **`sysstat`**
    `sar`와 같은 시스템 활동 데이터 수집기 유틸리티들에 대한 로깅 파라메터를 설정합니다.


더 많은 정보는 `/usr/share/doc/initscripts*/sysconfig.txt` 파일을 참조 하십시오.

## /proc 가상 파일시스템

`/proc` 디렉토리 계층 구조의 파일에는 시스템 하드웨어 및 시스템에서 실행 중인 프로세스에 대한 정보가 포함되어 있습니다. 쓰기 권한이 있는 특정 파일에 기록하여 커널 구성을 변경할 수 있습니다.

`/proc` 디렉토리 아래에 있는 파일은 기본 데이터 구조 및 시스템 정보에 대한 탐색 가능한 보기를 제공하기 위해 커널이 요청 시 생성하는 가상 파일입니다. 따라서 `/proc`은 가상 파일 시스템의 예입니다. 대부분의 가상 파일은 크기가 0바이트로 표시되지만, 보면 많은 양의 정보가 포함되어 있습니다.

`/proc/interrupts`, `/proc/meminfo`, `/proc/mounts` 및 `/proc/partitions`와 같은 가상 파일은 시스템 하드웨어에 대한 보기를 제공합니다. `/proc/filesystems` 및 `/proc/sys` 아래의 파일과 같은 다른 파일은 시스템 구성에 대한 정보를 제공하고 필요에 따라 구성을 변경할 수 있습니다.

관련 항목에 대한 정보가 포함된 파일은 가상 디렉터리로 그룹화됩니다. 시스템에서 실행 중인 각 프로세스에 대한 별도의 디렉터리가 '/proc' 디렉터리에 존재합니다. 디렉터리 이름은 숫자로 된 프로세스 ID에 해당합니다. 예를 들어 `/proc/1`은 PID가 1인 `systemd` 프로세스에 해당합니다.

가상 파일을 검사하려면 다음 예와 같이 `cat`, `less`, `view`와 같은 명령을 사용할 수 있습니다.

```
cat /proc/cpuinfo
```

```nocopybutton
processor         : 0
vendor_id         : GenuineIntel
cpu family        : 6
model             : 42
model name        : Intel(R) Core(TM) i5-2520M CPU @ 2.50GHz
stepping          : 7
cpu MHz           : 2393.714
cache size        : 6144 KB
physical id       : 0
siblings          : 2
core id           : 0
cpu cores         : 2
apicid            : 0
initial apicid    : 0
fpu               : yes
fpu_exception     : yes
cpuid level       : 5
wp                : yes
...
```

사람이 읽을 수 없는 콘텐츠가 포함된 파일의 경우 `lspci`, `free`, `top` 및 `sysctl`과 같은 유틸리티를 사용하여 정보에 액세스할 수 있습니다. 예를 들어, `lspci` 명령은 시스템의 PCI 장치를 나열합니다.

```
sudo lspci
```

```nocopybutton
00:00.0 Host bridge: Intel Corporation 440FX - 82441FX PMC [Natoma] (rev 02)
00:01.0 ISA bridge: Intel Corporation 82371SB PIIX3 ISA [Natoma/Triton II]
00:01.1 IDE interface: Intel Corporation 82371AB/EB/MB PIIX4 IDE (rev 01)
00:02.0 VGA compatible controller: InnoTek Systemberatung GmbH VirtualBox Graphics Adapter
00:03.0 Ethernet controller: Intel Corporation 82540EM Gigabit Ethernet Controller (rev 02)
00:04.0 System peripheral: InnoTek Systemberatung GmbH VirtualBox Guest Service
00:05.0 Multimedia audio controller: Intel Corporation 82801AA AC'97 Audio Controller (rev 01)
00:06.0 USB controller: Apple Inc. KeyLargo/Intrepid USB
00:07.0 Bridge: Intel Corporation 82371AB/EB/MB PIIX4 ACPI (rev 08)
00:0b.0 USB controller: Intel Corporation 82801FB/FBM/FR/FW/FRW (ICH6 Family) USB2 EHCI Controller
00:0d.0 SATA controller: Intel Corporation 82801HM/HEM (ICH8M/ICH8M-E) SATA Controller [AHCI mode]
        (rev 02)
...
```

### Virtual Files and Directories Under /proc
### /proc 하위의 가상 파일 및 디렉토리

다음 표에서는 `/proc` 디렉토리 계층 구조 아래에서 가장 유용한 가상 파일과 디렉토리를 설명합니다.

<table><thead><tr><th>

가상 파일과 디렉토리
</th><th>
설명
</th></tr></thead><tbody><tr><td>

`*PID*` \(Directory\)

</td><td>

프로세스 ID \(*PID*\)를 사용하여 프로세스에 대한 정보를 제공합니다. 디렉토리의 소유자 및 그룹은 프로세스의 소유자 및 그룹과 동일합니다. 디렉토리 아래의 유용한 파일은 다음과 같습니다.

 -   **`cmdline`**

커맨드 경로

-   **`cwd`**

프로세스의 현재 작업 디렉터리에 대한 심볼릭 링크입니다.

-   **`environ`**

환경 변수

-   **`exe`**

명령 실행 파일에 대한 심볼릭 링크입니다.

-   **`fd/_N_`**

File descriptors.

-   **`maps`**

메모리는 실행 파일 및 라이브러리 파일에 매핑됩니다.

-   **`root`**

프로세스의 유효 루트 디렉터리에 대한 기호 링크입니다.

-   **`stack`**

커널 스택의 내용입니다.

-   **`status`**

실행 상태 및 메모리 사용량.

</td></tr><tr><td>

`buddyinfo`

</td><td>

메모리 단편화 진단을 위한 정보를 제공합니다.

</td></tr><tr><td>

`bus` \(directory\)

</td><td>

Contains information about the various buses \(such as `pci` and `usb`\) that are available on the system. You can use commands such as `lspci`, `lspcmcia`, and `lsusb` to display information for such devices.

</td></tr><tr><td>

`cgroups`

</td><td>

Provides information about the resource control groups that are in use on the system.

</td></tr><tr><td>

`cmdline`

</td><td>

Lists parameters passed to the kernel at boot time.

</td></tr><tr><td>

`cpuinfo`

</td><td>

Provides information about the system's CPUs.

</td></tr><tr><td>

`crypto`

</td><td>

Provides information about all installed cryptographic cyphers.

</td></tr><tr><td>

`devices`

</td><td>

Lists the names and major device numbers of all currently configured characters and block devices.

</td></tr><tr><td>

`dma`

</td><td>

Lists the direct memory access \(DMA\) channels that are currently in use.

</td></tr><tr><td>

`driver` \(directory\)

</td><td>

Contains information about drivers used by the kernel, such as those for nonvolatile RAM \(`nvram`\), the real-time clock \(`rtc`\), and memory allocation for sound \(`snd-page-alloc`\).
</td></tr><tr><td>

`execdomains`
</td><td>

Lists the execution domains for binaries that the Enterprise Linux kernel supports.

</td></tr><tr><td>

`filesystems`

</td><td>

Lists the file system types that the kernel supports. Entries marked with `nodev` aren't in use.

</td></tr><tr><td>

`fs` \(directory\)

</td><td>

Contains information about mounted file systems, organized by file system type.

</td></tr><tr><td>

`interrupts`

</td><td>

Records the number of interrupts per interrupt request queue \(IRQ\) for each CPU after system startup.

</td></tr><tr><td>

`iomem`

</td><td>

Lists the system memory map for each physical device.

</td></tr><tr><td>

`ioports`

</td><td>

Lists the range of I/O port addresses that the kernel uses with devices.
</td></tr><tr><td>

`irq` \(directory\)

</td><td>

Contains information about each IRQ. You can configure the affinity between each IRQ and the system CPUs.

</td></tr><tr><td>

`kcore`

</td><td>

Presents the system's physical memory in `core` file format that you can examine using a debugger such as `crash` or `gdb`. This file isn't human-readable.

</td></tr><tr><td>

`kmsg`

</td><td>

Records kernel-generated messages, which are picked up by programs such as `dmesg`.

</td></tr><tr><td>

`loadavg`

</td><td>

Displays the system load averages \(number of queued processes\) for the past 1, 5, and 15 minutes, the number of running processes, the total number of processes, and the PID of the process that's running.

</td></tr><tr><td>

`locks`

</td><td>

Displays information about the file locks that the kernel is currently holding on behalf of processes. The information provided includes:

 -   lock class \(`FLOCK` or `POSIX`\)

-   lock type \(`ADVISORY` or `MANDATORY`\)

-   access type \(`READ` or `WRITE`\)

-   process ID

-   major device, minor device, and inode numbers

-   bounds of the locked region


</td></tr><tr><td>

`mdstat`

</td><td>

Lists information about multiple-disk RAID devices.

</td></tr><tr><td>

`meminfo`

</td><td>

Reports the system's usage of memory in more detail than is available using the `free` or `top` commands.

</td></tr><tr><td>

`modules`

</td><td>

Displays information about the modules that are currently loaded into the kernel. The `lsmod` command formats and displays the same information, excluding the kernel memory offset of a module.

</td></tr><tr><td>

`mounts`

</td><td>

Lists information about all mounted file systems.

</td></tr><tr><td>

`net` \(directory\)

</td><td>

Provides information about networking protocol, parameters, and statistics. Each directory and virtual file describes aspects of the configuration of the system's network.

</td></tr><tr><td>

`partitions`

</td><td>

Lists the major and minor device numbers, number of blocks, and name of partitions mounted by the system.

</td></tr><tr><td>

`scsi/device_info`

</td><td>

Provides information about SCSI devices.

</td></tr><tr><td>

`scsi/scsi` and

 `scsi/sg/*`

</td><td>

Provide information about configured SCSI devices, including vendor, model, channel, ID, and LUN data .

</td></tr><tr><td>

`self`

</td><td>

Symbolic link to the process that's examining `/proc`.

</td></tr><tr><td>

`slabinfo`

</td><td>

Provides detailed information about slab memory usage.

</td></tr><tr><td>

`softirqs`

</td><td>

Displays information about software interrupts \(`softirqs`\). A `softirq` is similar to a hardware interrupt \(`hardirq`\) and configures the kernel to perform asynchronous processing that would take too long during a hardware interrupt.

</td></tr><tr><td>

`stat`

</td><td>

Records information about the system from when it was started, including:

 -   **`cpu`**

Total CPU time \(measured in `jiffies`\) spent in user mode, low-priority user mode, system mode, idle, waiting for I/O, handling hardirq events, and handling softirq events.

-   **`cpu_N_`**

Times for CPU *N*.


</td></tr><tr><td>

`swaps`

</td><td>

Provides information about swap devices. The units of size and usage are in kilobytes.

</td></tr><tr><td>

`sys` \(directory\)

</td><td>

Provides information about the system and also enables you to enable, disable, or modify kernel features. You can write new settings to any file that has write permission. See [Modifying Kernel Parameters](osmanage-ConfiguringSystemSettings.md#).

 The following subdirectory hierarchies of `/proc/sys` contain virtual files, some of whose values you can alter:

 -   **`dev`**

Device parameters.

-   **`fs`**

File system parameters.

-   **`kernel`**

Kernel configuration parameters.

-   **`net`**

Networking parameters.


</td></tr><tr><td>

`sysvipc` \(directory\)

</td><td>

Provides information about the usage of System V Interprocess Communication \(IPC\) resources for messages \(`msg`\), semaphores \(`sem`\), and shared memory \(`shm`\).

</td></tr><tr><td>

`tty` \(directory\)

</td><td>

Provides information about the available and currently used terminal devices on the system. The `drivers` virtual file lists the devices that are currently configured.

</td></tr><tr><td>

`vmstat`

</td><td>

Provides information about virtual memory usage.

</td></tr><tbody></table>
For more information, see the `proc(5)` manual page.

### Modifying Kernel Parameters

Some virtual files under `/proc`, and especially under `/proc/sys`, are writable. You can adjust settings in the kernel through these files. For example, to change the hostname, you would revise the `/proc/sys/kernel/hostname` file as follows:

```
echo www.mydomain.com > /proc/sys/kernel/hostname
```

Other files take binary or Boolean values, such as the setting of IP forwarding, which is defined in `/proc/sys/net/ipv4/ip_forward`:

```
cat /proc/sys/net/ipv4/ip_forward
```

```nocopybutton
0
```

```
echo 1 > /proc/sys/net/ipv4/ip_forward
cat /proc/sys/net/ipv4/ip_forward
```

```nocopybutton
1
```

You can use the `sysctl` command to view or modify values under the `/proc/sys` directory.

**Note:**

Even `root` can't bypass the file access permissions of virtual file entries under `/proc`. If you change the value of a read-only entry such as `/proc/partitions`, no kernel code exists to service the `write()` system call.

To display the current kernel settings, use the following command:

```
sysctl -a
```

```nocopybutton
kernel.sched_child_runs_first = 0
kernel.sched_min_granularity_ns = 2000000
kernel.sched_latency_ns = 10000000
kernel.sched_wakeup_granularity_ns = 2000000
kernel.sched_shares_ratelimit = 500000
...
```

**Note:**

The delimiter character in the name of a setting is a period \(`.`\) rather than a slash \(`/`\) in a path relative to `/proc/sys`, such as `net.ipv4.ip_forward`. This setting represents `net/ipv4/ip_forward`. As another example, `kernel.msgmax` represents `kernel/msgmax`.

To display an individual setting, specify its name as the argument to `sysctl`:

```
**sysctl net.ipv4.ip\_forward**
net.ipv4.ip_forward = 0
```

To change the value of a setting, use the following command format:

```
**sysctl -w net.ipv4.ip\_forward=1**
net.ipv4.ip_forward = 1
```

Changes that you make in this way remain in force only until the system is rebooted. To make configuration changes persist after the system is rebooted, you must add them to the `/etc/sysctl.d` directory as a configuration file. Any changes that you make to the files in this directory take effect when the system reboots or if you run the `sysctl --system` command, for example:

```
echo 'net.ipv4.ip_forward=1' > /etc/sysctl.d/ip_forward.conf
grep -r ip_forward /etc/sysctl.d
```

```nocopybutton
/etc/sysctl.d/ip_forward.conf:net.ipv4.ip_forward=1
```

```
sysctl net.ipv4.ip_forward
```

```nocopybutton
net.ipv4.ip_forward = 0
```

```
sysctl --system
```

```nocopybutton
* Applying /usr/lib/sysctl.d/00-system.conf ...
net.bridge.bridge-nf-call-ip6tables = 0
net.bridge.bridge-nf-call-iptables = 0
net.bridge.bridge-nf-call-arptables = 0
* Applying /usr/lib/sysctl.d/50-default.conf ...
kernel.sysrq = 16
kernel.core_uses_pid = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.promote_secondaries = 1
net.ipv4.conf.all.promote_secondaries = 1
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
* Applying /etc/sysctl.d/99-sysctl.conf ...
* Applying /etc/sysctl.d/ip_forward.conf ...
net.ipv4.ip_forward = 1
* Applying /etc/sysctl.conf ...

```

```
sysctl net.ipv4.ip_forward
```

```nocopybutton
net.ipv4.ip_forward = 1
```

For more information, see the `sysctl(8)` and `sysctl.d(5)` manual pages.

### Parameters That Control System Performance

The following parameters control various aspects of system performance:

-   **`fs.file-max`**

    Specifies the maximum number of open files for all processes. Increase the value of this parameter if you see messages about running out of file handles.

-   **`net.core.netdev_max_backlog`**

    Specifies the size of the receiver backlog queue, which is used if an interface receives packets faster than the kernel can process them. If this queue is too small, packets are lost at the receiver, rather than on the network.

-   **`net.core.rmem_max`**

    Specifies the maximum read socket buffer size. To minimize network packet loss, this buffer must be large enough to handle incoming network packets.

-   **`net.core.wmem_max`**

    Specifies the maximum write socket buffer size. To minimize network packet loss, this buffer must be large enough to handle outgoing network packets.

-   **`net.ipv4.tcp_available_congestion_control`**

    Displays the TCP congestion avoidance algorithms that are available for use. Use the `modprobe` command if you need to load additional modules such as `tcp_htcp` to implement the `htcp` algorithm.

-   **`net.ipv4.tcp_congestion_control`**

    Specifies which TCP congestion avoidance algorithm is used.

-   **`net.ipv4.tcp_max_syn_backlog`**

    Specifies the number of outstanding `SYN` requests that are allowed. Increase the value of this parameter if you see `synflood` warnings in the logs that are cuased by the server being overloaded by legitimate connection attempts.

-   **`net.ipv4.tcp_rmem`**

    Specifies minimum, default, and maximum receive buffer sizes that are used for a TCP socket. The maximum value can't be larger than `net.core.rmem_max`.

-   **`net.ipv4.tcp_wmem`**

    Specifies minimum, default, and maximum send buffer sizes that are used for a TCP socket. The maximum value can't be larger than `net.core.wmem_max`.

-   **`vm.swappiness`**

    Specifies how likely the kernel is to write loaded pages to swap rather than drop pages from the system page cache. When set to 0, swapping only occurs to avoid an out of memory condition. When set to 100, the kernel swaps aggressively. For a desktop system, setting a lower value can improve system responsiveness by decreasing latency. The default value is 60.

    CAUTION:

    This parameter is intended for use with laptop computers to reduce power consumption by the hard disk. Do not adjust this value on server systems.


### Parameters That Control Kernel Panics

The following parameters control the circumstances under which a kernel panic can occur:

-   **`kernel.hung_task_panic`**

    If set to 1, the kernel panics if any kernel or user thread sleeps in the `TASK_UNINTERRUPTIBLE` state \(*D state*\) for more than `kernel.hung_task_timeout_secs` seconds. A process remains in D state while waiting for I/O to complete. You can't stop or interrupt a process in this state.

    The default value is 0, which disables the panic.

    **Tip:**

    To diagnose a hung thread, you can examine `/proc/*PID*/stack`, which displays the kernel stack for both kernel and user threads.

-   **`kernel.hung_task_timeout_secs`**

    Specifies how long a user or kernel thread can remain in D state before a warning message is generated or the kernel panics, if the value of `kernel.hung_task_panic` is 1. The default value is 120 seconds. A value of 0 disables the timeout.

-   **`kernel.nmi_watchdog`**

    If set to 1 \(default\), enables the nonmaskable interrupt \(NMI\) watchdog thread in the kernel. To use the NMI switch or the OProfile system profiler to generate an undefined NMI, set the value of `kernel.nmi_watchdog` to 0.

-   **`kernel.panic`**

    Specifies the number of seconds after a panic before a system automatically resets itself.

    If the value is 0, which is the default value, the system bcomes suspended, and you can collect detailed information about the panic for troubleshooting.

    To enable automatic reset, set a nonzero value. If you require a memory image \(`vmcore`\), leave enough time for Kdump to create this image. The suggested value is 30 seconds, although large systems require a longer time.

-   **`kernel.panic_on_io_nmi`**

    If set to 0 \(default\), the system tries to continue operations if the kernel detects an I/O channel check \(IOCHK\) NMI that typically indicates a uncorrectable hardware error. If set to 1, the system panics.

-   **`kernel.panic_on_oops`**

    If set to 0, the system tries to continue operations if the kernel detects an `oops` or BUG condition. If set to 1 \(default\), the system delays a few seconds to give the kernel log daemon, `klogd`, time to record the oops output before the panic occurs.

    In an OCFS2 cluster. set the value to 1 to specify that a system must panic if a kernel oops occurs. If a kernel thread required for cluster operation fails, the system must reset itself. Otherwise, another node might not detect whether a node is slow to respond or unable to respond, causing cluster operations to halt.

-   **`kernel.panic_on_unrecovered_nmi`**

    If set to 0 \(default\), the system tries to continue operations if the kernel detects an NMI that usually indicates an uncorrectable parity or ECC memory error. If set to 1, the system panics.

-   **`kernel.softlockup_panic`**

    If set to 0 \(default\), the system tries to continue operations if the kernel detects a *soft-lockup* error that causes the NMI watchdog thread to fail to update its timestamp for more than twice the value of `kernel.watchdog_thresh` seconds. If set to 1, the system panics.

-   **`kernel.unknown_nmi_panic`**

    If set to `1`, the system panics if the kernel detects an undefined NMI. You would usually generate an undefined NMI by manually pressing an NMI switch. As the NMI watchdog thread also uses the undefined NMI, set the value of `kernel.unknown_nmi_panic` to 0 if you set `kernel.nmi_watchdog` to 1.

-   **`kernel.watchdog_thresh`**

    Specifies the interval between generating an NMI performance monitoring interrupt that the kernel uses to check for *hard-lockup* and *soft-lockup* errors. A hard-lockup error is assumed if a CPU is unresponsive to the interrupt for more than `kernel.watchdog_thresh` seconds. The default value is 10 seconds. A value of 0 disables the detection of lockup errors.

-   **`vm.panic_on_oom`**

    If set to 0 \(default\), the kernel’s OOM-killer scans through the entire task list and stops a memory-hogging process to avoid a panic. If set to 1, the kernel panics but can survive under certain conditions. If a process limits allocations to certain nodes by using memory policies or cpusets, and those nodes reach memory exhaustion status, the OOM-killer can stop one process. No panic occurs in this case because other nodes’ memory might be free and the system as a whole might not yet be out of memory. If set to 2, the kernel always panics when an OOM condition occurs. Settings of 1 and 2 are for intended for use with clusters, depending on the defined failover policy.


## About the /sys Virtual File System

In addition to the `/proc` file system, the kernel exports information to the `/sys` virtual file system \(`sysfs`\). Programs such as the dynamic device manager \(`udev`\), use `/sys` to access device and device driver information.

**Note:**

`/sys` exposes kernel data structures and control points, which implies that the directory contains circular references, where a directory links to an ancestor directory. Thus, a `find` command used on `/sys` might never stop.

### Virtual Directories Under the /sys Directory

The following table describes some useful virtual directories under the `/sys` directory hierarchy.

<table><thead><tr><th>

Virtual Directory
</th><th>

Description
</th></tr></thead><tbody><tr><td>

`block`

</td><td>

Contains subdirectories for block devices. For example: `/sys/block/sda`.

</td></tr><tr><td>

`bus`

</td><td>

Contains subdirectories for each physical bus type, such as `pci`, `pcmcia`, `scsi`, or `usb`. Under each bus type, the `devices` directory lists discovered devices, and the `drivers` directory contains directories for each device driver.

</td></tr><tr><td>

`class`

</td><td>

Contains subdirectories for every class of device that's registered with the kernel.

</td></tr><tr><td>

`dev`

</td><td>

Contains the `char/` and `block/` directories. Inside these two directories are symlinks named *major*:*minor*. These symlinks point to the `sysfs` directory for the particular device. The `/sys/dev` directory provides a quick way to look up the `sysfs` interface for a device from the result of the `stat(2)` operation.

</td></tr><tr><td>

`devices`

</td><td>

Contains the global device hierarchy of all devices on the system. The platform directory contains peripheral devices such as device controllers that are specific to a particular platform. The `system` directory contains non peripheral devices such as CPUs and APICs. The `virtual` directory contains virtual and pseudo devices. See [Managing System Devices](osmanage-ManagingSystemDevices.md#).

</td></tr><tr><td>

`firmware`

</td><td>

Contains subdirectories for firmware objects.

</td></tr><tr><td>

`fs`

</td><td>

Contains subdirectories for file system objects.

</td></tr><tr><td>

`kernel`

</td><td>

Contains subdirectories for other kernel objects

</td></tr><tr><td>

`module`

</td><td>

Contains subdirectories for each module loaded into the kernel. You can alter some parameter values for loaded modules. See [About Module Parameters](osmanage-ManagingKernelModules.md#).

</td></tr><tr><td>

`power`
</td><td>

Contains attributes that control the system's power state.

</td></tr><tbody></table>
For more information, see [https://www.kernel.org/doc/Documentation/filesystems/sysfs.txt](https://www.kernel.org/doc/Documentation/filesystems/sysfs.txt).

## Configuring System Date and Time Settings

System time is based on the POSIX time standard, where time is measured as the number of seconds that have elapsed from 00:00:00 Coordinated Universal Time \(UTC\), Thursday, January 1, 1970. A day is defined as 86400 seconds and leap seconds are subtracted automatically.

Date and time representation on a system can be set to match a specific timezone. To list the available timezones, run:

```
timedatectl list-timezones
```

To set the system timezone to match a value returned from the available timezones, you can run:

```
timedatectl set-timezone *America/Los\_Angeles*
```

Substitute *America/Los\_Angeles* with a valid timezone entry.

This command sets a symbolic link from `/etc/localtime` to point to the appropriate zone information file in `/usr/share/zoneinfo/`. The setting takes effect immediately. Some long running processes that use `/etc/localtime` to detect the current system timezone might not detect a change in system timezone until the process is restarted.

Note that timezones are largely used for display purposes or to handle user input. Changing timezone doesn't change the time for the system clock. You can change the presentation for system time in any console by setting the `TZ` environment variable. For example, to see the current time in Tokyo, you can run:

```
TZ="*Asia/Tokyo*" date
```

You can check the system's current date and time configuration by running the `timedatectl` command on its own:

```
timedatectl
```

```nocopybutton
               Local time: Wed 2021-07-17 00:50:58 EDT                                                                                                                                                                                                     
           Universal time: Wed 2021-07-17 04:50:58 UTC                                                                                                                                                                                                     
                 RTC time: Wed 2021-07-17 04:50:55                                                                                                                                                                                                         
                Time zone: America/New_York (EDT, -0400)                                                                                                                                                                                                   
System clock synchronized: yes                                                                                                                                                                                                                             
              NTP service: active                                                                                                                                                                                                                          
          RTC in local TZ: no                              
```

To set system time manually, use the `timedatectl set-time` command:

```
timedatectl set-time "*2021-07-17 01:59:59*"
```

This command sets the current system time based on the time specified assuming the currently set system timezone. The command also updates the system Real Time Clock \(RTC\).

Consider configuring the system to use network time synchronization for more accurate time-keeping. Using network time synchronization is important especially when setting up high-availability or when using network-based file systems.

If you configure an NTP service, enable NTP by running the following command:

```
timedatectl set-ntp true****
```

This command enables and starts the `chronyd` service, if available.

## Configuring the Watchdog Service

Watchdog is an Enterprise Linux service that runs in the background to monitor host availability and processes and reports back to the kernel. If the Watchdog service fails to notify the kernel that the system is healthy, the kernel typically automatically reboots the system.

To install the Watchdog package, run:

```
sudo dnf install watchdog
```

To configure the Watchdog service, edit the `/etc/watchdog.conf` file. The `watchdog.conf` file includes all Watchdog configuration properties. For information on how to edit this file, see the `watchdog.conf(5)` manual page.

To enable and start the Watchdog service, run:

```
sudo systemctl enable --now watchdog
```

The Watchdog service immediately starts and runs in the background.

**Note:** The Watchdog service starts and runs immediately after a power reset.

