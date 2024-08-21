<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# 시스템 설정 구성

이 장에서는 시스템의 구성 설정을 변경하는 데 사용할 수 있는 파일 및 가상 파일 시스템에 대해 설명합니다.

## /etc/sysconfig 디렉터리

`/etc/sysconfig` 디렉터리에는 시스템 구성을 제어하는 파일이 포함되어 있습니다. 이 디렉터리의 내용은 시스템에 설치한 패키지에 따라 다릅니다.

`/etc/sysconfig` 디렉터리에서 찾을 수 있는 특정 파일은 다음과 같습니다.

- **`atd`**

  `atd` 데몬에 대한 커맨드라인 파라메터를 지정합니다.

- **`crond`**

  부팅 시 `crond` 데몬에 파라메터를 전달합니다.

- **`chronyd`**

  부팅 시 NTP 서비스에 사용되는 `chronyd` 데몬에 파라메터를 전달합니다.

- **`firewalld`**

  부팅 시 방화벽 데몬(`firewalld`)에 파라메터를 전달합니다.

- **`named`**

  부팅 시 이름(name) 서비스 데몬에 파라메터를 전달합니다. `named` 데몬몬 BIND(Berkeley Internet Name Domain) 배포의 일부인 DNS(Domain Name System) 서버입니다. 이 서버는 호스트 이름을 네트워크의 IP 주소와 연결하는 테이블을 유지 관리합니다.

- **`samba`**

  Windows 클라이언트에 대한 파일 공유 연결, NetBIOS-over-IP 이름(name) 지정 서비스 및 도메인 컨트롤러에 대한 연결 관리를 지원하기 위해 부팅 시 `smbd`, `nmbd` 및 `winbindd` 데몬에 파라메터를 전달합니다.

- **`selinux`**

  시스템의 SELinux 상태를 제어합니다. 이 파일은 `/etc/selinux/config`에 대한 심볼릭 링크입니다.

- **`snapper`**

  `snapper` 유틸리티의 목록을 정의합니다.

- **`sysstat`**

  **`sysstat`**
  `sar`와 같은 시스템 활동 데이터 수집기 유틸리티들에 대한 로깅 파라메터를 설정합니다.

더 많은 정보는 `/usr/share/doc/initscripts*/sysconfig.txt` 파일을 참조 하십시오.

## /proc 가상 파일시스템

`/proc` 디렉터리 계층 구조의 파일에는 시스템 하드웨어 및 시스템에서 실행 중인 프로세스에 대한 정보가 포함되어 있습니다. 쓰기 권한이 있는 특정 파일에 기록하여 커널 구성을 변경할 수 있습니다.

`/proc` 디렉터리 아래에 있는 파일은 기본 데이터 구조 및 시스템 정보에 대한 탐색 가능한 보기를 제공하기 위해 커널이 요청 시 생성하는 가상 파일입니다. 따라서 `/proc`은 가상 파일 시스템의 예입니다. 대부분의 가상 파일은 크기가 0바이트로 표시되지만, 볼 때 많은 양의 정보가 포함되어 있습니다.

`/proc/interrupts`, `/proc/meminfo`, `/proc/mounts` 및 `/proc/partitions`와 같은 가상 파일은 시스템 하드웨어에 대한 보기를 제공합니다. `/proc/filesystems` 및 `/proc/sys` 아래의 파일과 같은 다른 파일은 시스템 구성에 대한 정보를 제공하고 필요에 따라 구성을 변경할 수 있습니다.

관련 항목에 대한 정보가 포함된 파일은 가상 디렉터리로 그룹화됩니다. 시스템에서 실행 중인 각 프로세스에 대한 별도의 디렉터리가 `/proc` 디렉터리에 존재합니다. 디렉터리 이름은 숫자로 된 프로세스 ID에 해당합니다. 예를 들어 `/proc/1`은 PID가 1인 `systemd` 프로세스에 해당합니다.

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

### /proc 하위의 가상 파일 및 디렉터리

다음 표에서는 `/proc` 디렉터리 계층 구조 아래에서 가장 유용한 가상 파일과 디렉터리를 설명합니다.

<table><thead><tr><th>

가상 파일과 디렉터리

</th><th>

설명

</th></tr></thead><tbody><tr><td>

`*PID*` \(Directory\)

</td><td>

프로세스 ID \(_PID_\)를 사용하여 프로세스에 대한 정보를 제공합니다. 디렉터리의 소유자 및 그룹은 프로세스의 소유자 및 그룹과 동일합니다. 디렉터리 아래의 유용한 파일은 다음과 같습니다.

- **`cmdline`**

커맨드 경로

- **`cwd`**

프로세스의 현재 작업 디렉터리에 대한 심볼릭 링크입니다.

- **`environ`**

환경 변수

- **`exe`**

명령 실행 파일에 대한 심볼릭 링크입니다.

- **`fd/_N_`**

File descriptors.

- **`maps`**

메모리는 실행 파일 및 라이브러리 파일에 매핑됩니다.

- **`root`**

프로세스의 유효 루트 디렉터리에 대한 기호 링크입니다.

- **`stack`**

커널 스택의 내용입니다.

- **`status`**

실행 상태 및 메모리 사용량.

</td></tr><tr><td>

`buddyinfo`

</td><td>

메모리 단편화 진단을 위한 정보를 제공합니다.

</td></tr><tr><td>

`bus` \(directory\)

</td><td>

시스템에서 사용할 수 있는 다양한 버스\(예: `pci` 및 `usb`\)에 대한 정보가 포함되어 있습니다. `lspci`, `lspcmcia`, `lsusb`와 같은 명령을 사용하여 해당 장치에 대한 정보를 표시할 수 있습니다.

</td></tr><tr><td>

`cgroups`

</td><td>

시스템에서 사용 중인 리소스 제어 그룹에 대한 정보를 제공합니다.

</td></tr><tr><td>

`cmdline`

</td><td>

부팅 시 커널에 전달된 파라메터를 나열합니다.

</td></tr><tr><td>

`cpuinfo`

</td><td>

시스템의 CPU에 대한 정보를 제공합니다.

</td></tr><tr><td>

`crypto`

</td><td>

설치된 모든 암호화 기법에 대한 정보를 제공합니다.

</td></tr><tr><td>

`devices`

</td><td>

현재 구성된 모든 문자와 블록 장치의 이름과 주요 장치 번호를 나열합니다.

</td></tr><tr><td>

`dma`

</td><td>

현재 사용 중인 직접 메모리 액세스\(DMA\) 채널을 나열합니다.

</td></tr><tr><td>

`driver` \(directory\)

</td><td>

비휘발성 RAM(`nvram`), 실시간 시계(`rtc`) 및 사운드용 메모리 할당(`snd-page-alloc`)과 같이 커널에서 사용하는 드라이버에 대한 정보가 포함되어 있습니다.

</td></tr><tr><td>

`execdomains`

</td><td>

Enterprise Linux 커널이 지원하는 바이너리의 실행 도메인을 나열합니다.

</td></tr><tr><td>

`filesystems`

</td><td>

커널이 지원하는 파일 시스템 유형을 나열합니다. `nodev`로 표시된 항목은 사용되지 않습니다.

</td></tr><tr><td>

`fs` \(directory\)

</td><td>

파일 시스템 유형별로 구성된 마운트된 파일 시스템에 대한 정보를 포함합니다.

</td></tr><tr><td>

`interrupts`

</td><td>

시스템 시작 후 각 CPU에 대한 인터럽트 요청 큐\(IRQ\)당 인터럽트 수를 기록합니다.

</td></tr><tr><td>

`iomem`

</td><td>

각 물리적 장치에 대한 시스템 메모리 맵을 나열합니다.

</td></tr><tr><td>

`ioports`

</td><td>

커널이 장치와 함께 사용하는 I/O 포트 주소 범위를 나열합니다.

</td></tr><tr><td>

`irq` \(directory\)

</td><td>

각 IRQ에 대한 정보를 포함합니다. 각 IRQ와 시스템의 CPU 간의 선호도(affinity)를 구성할 수 있습니다.

</td></tr><tr><td>

`kcore`

</td><td>

`crash` 또는 `gdb`와 같은 디버거를 사용하여 검사할 수 있는 `core` 파일 형식으로 시스템의 물리적 메모리를 제공합니다. 이 파일은 사람이 읽을 수 없습니다.

</td></tr><tr><td>

`kmsg`

</td><td>

커널이 생성한 메시시를 기록하며, `dmesg`와 같은 프로그램에서 `kmsg`를 채용합니다.

</td></tr><tr><td>

`loadavg`

</td><td>

지난 1분, 5분, 15분 동안의 시스템 부하 평균\(대기 중인 프로세스 수\), 실행 중인 프로세스 수, 총 프로세스 수, 실행 중인 프로세스의 PID를 표시합니다.

</td></tr><tr><td>

`locks`

</td><td>

프로세스를 대신하여 커널이 현재 보유하고 있는 파일 잠금에 대한 정보를 표시합니다. 제공되는 정보는 다음과 같습니다:

- lock class \(`FLOCK` or `POSIX`\)

- lock type \(`ADVISORY` or `MANDATORY`\)

- access type \(`READ` or `WRITE`\)

- process ID

- major device, minor device, and inode numbers

- bounds of the locked region

</td></tr><tr><td>

`mdstat`

</td><td>

다중 디스크 RAID 장치에 대한 정보를 나열합니다.

</td></tr><tr><td>

`meminfo`

</td><td>

`free` 또는 `top` 명령을 사용하여 사용할 수 있는 것보다 더 자세한 시스템의 메모리 사용량을 보고합니다.

</td></tr><tr><td>

`modules`

</td><td>

현재 커널에 로드된 모듈에 대한 정보를 표시합니다. `lsmod` 명령은 모듈의 커널 메모리 오프셋을 제외하고 동일한 정보를 형식화하고 표시합니다.

</td></tr><tr><td>

`mounts`

</td><td>

마운트된 모든 파일 시스템에 대한 정보를 나열합니다.

</td></tr><tr><td>

`net` \(directory\)

</td><td>

네트워킹 프로토콜, 파라메터 및 통계에 대한 정보를 제공합니다. 각 디렉터리와 가상 파일은 시스템 네트워크 구성의 측면을 설명합니다.

</td></tr><tr><td>

`partitions`

</td><td>

major 및 minor 장치의 번호, 블록 수, 시스템에 의해 마운트된 파티션 이름을 나열합니다.

</td></tr><tr><td>

`scsi/device_info`

</td><td>

SCSI 장치들의 정보를 제공합니다.

</td></tr><tr><td>

`scsi/scsi` and

`scsi/sg/*`

</td><td>

공급업체, 모델, 채널, ID 및 LUN 데이터를 포함하여 구성된 SCSI 장치에 대한 정보를 제공합니다.

</td></tr><tr><td>

`self`

</td><td>

`/proc`을 검사하는 프로세스에 대한 심볼릭 링크입니다.

</td></tr><tr><td>

`slabinfo`

</td><td>

slab 메모리 사용량에 대한 자세한 정보를 제공합니다.

</td></tr><tr><td>

`softirqs`

</td><td>

소프트웨어 인터럽트\(`softirqs`\)에 대한 정보를 표시합니다. `softirq`는 하드웨어 인터럽트\(`hardirq`\)와 유사하며 하드웨어 인터럽트 중에 너무 오래 걸리는 비동기 처리를 수행하도록 커널을 구성합니다.

</td></tr><tr><td>

`stat`

</td><td>

다음을 포함하여 시스템 시작 시부터 시스템에 대한 정보를 기록합니다.

- **`cpu`**

사용자 모드, 우선 순위가 낮은 사용자 모드, 시스템 모드, 유휴 I/O 대기, hardirq 이벤트 처리 및 Softirq 이벤트 처리에 소요된 총 CPU 시간(`jiffies`로 측정).

- **`cpu_N_`**

Times for CPU _N_.

</td></tr><tr><td>

`swaps`

</td><td>

스왑 장치에 대한 정보를 제공합니다. 크기 및 사용량 단위는 킬로바이트입니다.

</td></tr><tr><td>

`sys` \(directory\)

</td><td>

시스템에 대한 정보를 제공하고 커널 기능을 활성화, 비활성화 또는 수정할 수도 있습니다. 쓰기 권한이 있는 모든 파일에 새 설정을 쓸 수 있습니다. 참조: [커널 파라메터 수정](ko-osmanage-ConfiguringSystemSettings.md#커널-파라메터-수정).

/proc/sys의 다음 하위 디렉터리 계층에는 가상 파일이 포함되어 있으며 그 중 일부 값은 변경할 수 있습니다.:

- **`dev`**

Device parameters.

- **`fs`**

File system parameters.

- **`kernel`**

Kernel configuration parameters.

- **`net`**

Networking parameters.

</td></tr><tr><td>

`sysvipc` \(directory\)

</td><td>

메시지(`msg`), 세마포어(`sem`) 및 공유 메모리(`shm`)에 대한 System V IPC(Interprocess Communication) 리소스 사용에 대한 정보를 제공합니다.

</td></tr><tr><td>

`tty` \(directory\)

</td><td>

시스템에서 사용 가능하고 현재 사용되는 터미널 장치에 대한 정보를 제공합니다. `driver` 가상 파일에는 현재 구성된 장치가 나열됩니다.

</td></tr><tr><td>

`vmstat`

</td><td>

가상 메모리 사용량에 대한 정보를 제공합니다.

</td></tr><tbody></table>
For more information, see the `proc(5)` manual page.

### 커널 파라메터 수정

`/proc` 아래, 특히 `/proc/sys` 아래의 일부 가상 파일은 쓰기 가능합니다. 이 파일을 통해 커널의 설정을 조정할 수 있습니다. 예를 들어 호스트 이름을 변경하려면 `/proc/sys/kernel/hostname` 파일을 다음과 같이 수정합니다.:

```
echo www.mydomain.com > /proc/sys/kernel/hostname
```

예를 들어 IP 포워딩 세팅의 경우 `/proc/sys/net/ipv4/ip_forward`에 정의합니다.:

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

`sysctl` 명령을 사용하여 `/proc/sys` 디렉터리 아래의 값을 보거나 수정할 수 있습니다.

**Note:**

`root` 유저 마저도 `/proc` 아래 가상 파일 항목의 파일 액세스 권한을 우회할 수 없습니다. 예를 들어 `/proc/partitions`와 같은 읽기 전용 항목의 값을 변경할 경우 `write()` 시스템 호출을 서비스하는 커널 코드가 존재하지 않기 때문에 변경할 수 없습니다.

현재 커널 설정을 표시하려면 다음 명령을 사용하십시오.:

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

설정 이름의 구분 문자는 `net.ipv4.ip_forward`와 같이 `/proc/sys`에 대한 상대 경로에서 슬래시 \(`/`\)가 아닌 마침표 \(`.`\)입니다. 이 설정은 `net/ipv4/ip_forward`를 나타냅니다. 또 다른 예로 `kernel.msgmax`는 `kernel/msgmax`를 나타냅니다.

개별 설정을 표시하려면 `sysctl`에 대한 인수로 해당 이름을 지정하십시오.

```
sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 0
```

설정 값을 변경하려면 다음 명령 형식을 사용하십시오.:

```
sysctl -w net.ipv4.ip_forward=1
net.ipv4.ip_forward = 1
```

이러한 방식으로 변경한 사항은 시스템이 재부팅될 때까지만 적용됩니다. 시스템을 재부팅한 후에도 구성 변경 사항을 유지하려면 구성 파일로 `/etc/sysctl.d` 디렉터리에 추가해야 합니다. 이 디렉터리의 파일에 대한 모든 변경 사항은 시스템을 재부팅하거나 `sysctl --system` 명령을 실행하면 적용됩니다.

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

자세한 내용은 `sysctl(8)` 및 `sysctl.d(5)` 매뉴얼 페이지를 참조하세요.

### 시스템 성능을 제어하는 ​​파라메터

다음 파라메터는 시스템 성능의 다양한 측면을 제어합니다.

- **`fs.file-max`**

  모든 프로세스에 대해 열린 파일의 최대 수를 지정합니다. 파일 핸들 부족에 대한 메시지가 표시되면 이 파라메터의 값을 늘리십시오.

- **`net.core.netdev_max_backlog`**

  인터페이스가 커널이 처리할 수 있는 것보다 더 빠른 속도로 패킷을 수신하는 경우 사용되는 수신자 백로그 대기열의 크기를 지정합니다. 이 대기열이 너무 작으면 네트워크가 아닌 수신자에서 패킷이 손실됩니다.

- **`net.core.rmem_max`**

  최대 읽기 소켓 버퍼 크기를 지정합니다. 네트워크 패킷 손실을 최소화하려면 이 버퍼가 들어오는 네트워크 패킷을 처리할 수 있을 만큼 커야 합니다.

- **`net.core.wmem_max`**

  최대 쓰기 소켓 버퍼 크기를 지정합니다. 네트워크 패킷 손실을 최소화하려면 이 버퍼가 나가는 네트워크 패킷을 처리할 수 있을 만큼 커야 합니다.

- **`net.ipv4.tcp_available_congestion_control`**

  사용할 수 있는 TCP 혼잡 회피 알고리즘을 표시합니다. `htcp` 알고리즘을 구현하기 위해 `tcp_htcp`와 같은 추가 모듈을 로드해야 하는 경우 `modprobe` 명령을 사용하세요.

- **`net.ipv4.tcp_congestion_control`**

  어떤 TCP 혼잡 회피 알고리즘이 사용되는지 지정합니다.

- **`net.ipv4.tcp_max_syn_backlog`**

  Specifies the number of outstanding `SYN` requests that are allowed. 합법적인 연결 시도로 인해 서버가 과부하되어 `synflood` 경고 로그가 표시되는 경우 이 파라메터의 값을 늘리십시오.

- **`net.ipv4.tcp_rmem`**

  TCP 소켓에 사용되는 최소, 기본 및 최대 수신 버퍼 크기를 지정합니다. 최대값은 `net.core.rmem_max`보다 클 수 없습니다.

- **`net.ipv4.tcp_wmem`**

  TCP 소켓에 사용되는 최소, 기본 및 최대 전송 버퍼 크기를 지정합니다. 최대값은 `net.core.wmem_max`보다 클 수 없습니다.

- **`vm.swappiness`**

  커널이 시스템 페이지 캐시에서 페이지를 삭제하는 대신 스왑을 위해 로드된 페이지를 사용할 가능성을 지정합니다. 0으로 설정하면 메모리 부족 상태를 방지하기 위해서만 스와핑이 발생합니다. 100으로 설정하면 커널이 적극적으로 스왑됩니다. 데스크탑 시스템의 경우 낮은 값을 설정하면 대기 시간이 줄어들어 시스템 응답성이 향상될 수 있습니다. 기본값은 60입니다.

  CAUTION:

  This parameter is intended for use with laptop computers to reduce power consumption by the hard disk. Do not adjust this value on server systems.

### 커널 패닉을 제어하는 ​​파라메터

다음 파라메터는 커널 패닉이 발생할 수 있는 상황을 제어합니다.:

- **`kernel.hung_task_panic`**

  1로 설정된 경우, 커널 또는 사용자 스레드가 `kernel.hung_task_timeout_secs` 초 이상 `TASK_UNINTERRUPTIBLE` 상태\(_D 상태_\)에서 휴면하면 커널 패닉이 발생합니다. I/O가 완료되기를 기다리는 동안 프로세스는 D 상태로 유지됩니다. 이 상태에서는 프로세스를 중지하거나 중단할 수 없습니다.

  기본값은 0이며 패닉을 비활성화합니다.

  **Tip:**

  중단된 스레드를 진단하려면 커널 및 사용자 스레드 모두에 대한 커널 스택을 표시하는 `/proc/*PID*/stack`을 검사할 수 있습니다.

- **`kernel.hung_task_timeout_secs`**

  `kernel.hung_task_panic` 값이 1인 경우 경고 메시지가 생성되거나 커널 패닉이 발생하기 전에 사용자 또는 커널 스레드가 D 상태에 머무를 수 있는 기간을 지정합니다. 기본값은 120초입니다. 값이 0이면 시간 초과가 비활성화됩니다.

- **`kernel.nmi_watchdog`**

  1\(기본값\)으로 설정하면 커널에서 마스크 불가능한 인터럽트\(NMI\) 감시 스레드를 활성화합니다. NMI 스위치나 OProfile 시스템 프로파일러를 사용하여 정의되지 않은 NMI를 생성하려면 `kernel.nmi_watchdog` 값을 0으로 설정하십시오.

- **`kernel.panic`**

  패닉이 발생한 후 시스템이 자동으로 재부팅되기 전까지의 시간(초)을 지정합니다.

  값이 기본값인 0이면 시스템이 일시 중지되며 문제 해결을 위해 패닉에 대한 자세한 정보를 수집할 수 있습니다.

  자동 재부팅을 활성화하려면 0이 아닌 값을 설정하십시오. 메모리 이미지 \(`vmcore`\)가 필요한 경우 Kdump가 이 이미지를 생성할 수 있도록 충분한 시간을 두십시오. 제안된 값은 30초이지만 대형 시스템에는 더 긴 시간이 필요합니다.

- **`kernel.panic_on_io_nmi`**

  0\(기본값\)으로 설정된 경우 커널이 일반적으로 수정할 수 없는 하드웨어 오류를 나타내는 I/O channel check\(IOCHK\) NMI를 감지하면 시스템은 패닉 발생을 유발하지 않고 작업을 계속하려고 시도합니다. 1로 설정하면 시스템 패닉이 발생합니다.

- **`kernel.panic_on_oops`**

  0으로 설정하면 커널이 `oops` 또는 BUG 조건을 감지하면 시스템이 작업을 계속하려고 시도합니다. 1\(기본값\)로 설정하면 시스템은 패닉이 발생하기 전에 커널 로그 데몬 `klogd`에 `oops` 출력을 기록할 시간을 제공하기 위해 몇 초를 지연합니다.

  OCFS2 클러스터에서. 커널 오류가 발생할 경우 시스템이 패닉이 발생하도록 지정하려면 값을 1로 설정합니다. 클러스터 작업에 필요한 커널 스레드가 실패하면 시스템이 자체적으로 재설정되어야 합니다. 그렇지 않으면 다른 노드가 노드의 응답 속도가 느리거나 응답할 수 없는지 여부를 감지하지 못해 클러스터 작업이 중단될 수 있습니다.

- **`kernel.panic_on_unrecovered_nmi`**

  0\(기본값\)으로 설정된 경우 커널이 일반적으로 수정할 수 없는 패리티 또는 ECC 메모리 오류를 나타내는 NMI를 감지하면 시스템은 작업을 계속하려고 시도합니다. 1로 설정하면 시스템 패닉이 발생합니다.

- **`kernel.softlockup_panic`**

  0\(기본값\)으로 설정된 경우 커널이 NMI 감시 스레드가 `kernel.watchdog_thresh` 값의 두 배 이상 타임스탬프를 업데이트하지 못하도록 하는 _soft-lockup_ 오류를 감지하면 시스템은 작업을 계속하려고 시도합니다. 1로 설정하면 시스템 패닉이 발생합니다.

- **`kernel.unknown_nmi_panic`**

  `1`로 설정하면 커널이 정의되지 않은 NMI를 감지하면 시스템 패닉이 발생합니다. 일반적으로 NMI 스위치를 수동으로 눌러 정의되지 않은 NMI를 생성합니다. NMI watchdog 스레드도 정의되지 않은 NMI를 사용하므로 `kernel.nmi_watchdog`을 1로 설정한 경우 `kernel.unknown_nmi_panic` 값을 0으로 설정합니다.

- **`kernel.watchdog_thresh`**

  커널이 _hard-lockup_ 및 _soft-lockup_ 오류를 확인하는 데 사용하는 NMI 성능 모니터링 인터럽트 생성 사이의 간격을 지정합니다. CPU가 `kernel.watchdog_thresh` 초 이상 인터럽트에 응답하지 않으면 하드 잠금 오류가 발생한다고 가정합니다. 기본값은 10초입니다. 값이 0이면 잠금 오류 감지가 비활성화됩니다.

- **`vm.panic_on_oom`**

  0\(기본값\)으로 설정하면 커널의 OOM-killer가 전체 태스크 목록을 검색하고 메모리를 많이 차지하는 프로세스를 중지하여 패닉을 방지합니다. 1로 설정하면 커널이 패닉 상태가 되지만 특정 조건에서는 살아남을 수 있습니다. 프로세스가 메모리 정책이나 CPUset을 사용하여 특정 노드에 대한 할당을 제한하고 해당 노드가 메모리 소진 상태에 도달하면 OOM-killer는 하나의 프로세스를 중지할 수 있습니다. 이 경우 다른 노드의 메모리가 비어 있고 시스템 전체에 아직 메모리가 부족하지 않을 수 있으므로 패닉이 발생하지 않습니다. 2로 설정하면 OOM 조건이 발생할 때 커널이 항상 패닉 상태가 됩니다. 1과 2 설정은 정의된 장애 조치 정책에 따라 클러스터에 사용하기 위한 것입니다.

## /sys 가상 파일 시스템

커널은 `/proc` 파일 시스템 외에도 `/sys` 가상 파일 시스템\(`sysfs`\)으로 정보를 내보냅니다. 동적 장치 관리자\(`udev`\)와 같은 프로그램은 `/sys`를 사용하여 장치 및 장치 드라이버 정보에 액세스합니다.

**Note:**

`/sys`는 커널 데이터 구조와 제어 지점을 나타냅니다. 따라서 `/sys`에 사용된 `find` 명령은 절대 멈추지 않을 수 있습니다.

### /sys 디렉터리 아래의 가상 디렉터리

다음 표에서는 `/sys` 디렉터리 계층 구조 아래에 있는 몇 가지 유용한 가상 디렉터리에 대해 설명합니다.

<table><thead><tr><th>

가상 디렉터리

</th><th>

설명

</th></tr></thead><tbody><tr><td>

`block`

</td><td>

블록 장치에 대한 하위 디렉터리를 포함합니다. 예를 들어: `/sys/block/sda`.

</td></tr><tr><td>

`bus`

</td><td>

`pci`, `pcmcia`, `scsi` 또는 `usb`와 같은 각 물리적 버스 유형에 대한 하위 디렉터리를 포함합니다. 각 버스 유형 아래의 `devices` 디렉터리에는 검색된 장치가 나열되고, `drivers` 디렉터리에는 각 장치 드라이버에 대한 디렉터리가 포함됩니다.

</td></tr><tr><td>

`class`

</td><td>

커널에 등록된 모든 장치 클래스에 대한 하위 디렉터리를 포함합니다.

</td></tr><tr><td>

`dev`

</td><td>

`char/` 및 `block/` 디렉터리를 포함합니다. 이 두 디렉터리 안에는 _major_:_minor_라는 심볼릭 링크가 있습니다. 이러한 심볼릭 링크는 특정 장치의 `sysfs` 디렉터리를 가리킵니다. `/sys/dev` 디렉터리는 `stat(2)` 작업의 결과에서 장치의 `sysfs` 인터페이스를 찾는 빠른 방법을 제공합니다.

</td></tr><tr><td>

`devices`

</td><td>

시스템에 있는 모든 장치의 전역 장치 계층 구조를 포함합니다. 플랫폼 디렉터리에는 특정 플랫폼에 특정한 장치 컨트롤러와 같은 주변 장치가 포함되어 있습니다. `system` 디렉터리에는 CPU 및 APIC와 같은 비주변 장치가 포함되어 있습니다. `virtual` 디렉터리에는 가상 장치와 pseudo 장치가 포함되어 있습니다. [시스템 장치 관리](ko-osmanage-ManagingSystemDevices.md#시스템-장치-관리)를 참조하세요.

</td></tr><tr><td>

`firmware`

</td><td>

펌웨어 개체에 대한 하위 디렉터리가 포함되어 있습니다.

</td></tr><tr><td>

`fs`

</td><td>

파일시스템 개체에 대한 하위 디렉터리를 포함합니다.

</td></tr><tr><td>

`kernel`

</td><td>

커널 개체에 대한 하위 디렉터리를 포함합니다.

</td></tr><tr><td>

`module`

</td><td>

커널에 로드된 각 모듈의 하위 디렉터리를 포함합니다. 로드된 모듈의 일부 파라메터 값을 변경할 수 있습니다. 참조: [모듈 매개변수](ko-osmanage-ManagingKernelModules.md#모듈-매개변수)

</td></tr><tr><td>

`power`

</td><td>

시스템의 전원 상태를 제어하는 ​​속성을 포함합니다.

</td></tr><tbody></table>
For more information, see [https://www.kernel.org/doc/Documentation/filesystems/sysfs.txt](https://www.kernel.org/doc/Documentation/filesystems/sysfs.txt).

## 시스템 날짜 및 시간 설정 구성

시스템 시간은 1970년 1월 1일 목요일 00:00:00 UTC로부터 시간이 경과한 초수로 측정되는 POSIX 시간 표준을 기반으로 합니다. 하루는 86400초로 정의되고 윤초는 자동으로 차감됩니다.

시스템의 날짜 및 시간 표현은 특정 시간대와 일치하도록 설정할 수 있습니다. 사용 가능한 시간대를 나열하려면 다음을 실행하세요.

```
timedatectl list-timezones
```

사용 가능한 시간대에서 반환된 값과 일치하도록 시스템 시간대를 설정하려면 다음을 실행할 수 있습니다.

```
timedatectl set-timezone *America/Los\_Angeles*
```

_America/Los\_Angeles_를 유효한 시간대 항목으로 대체하세요.

이 명령은 `/etc/localtime`의 심볼릭 링크를 설정하여 `/usr/share/zoneinfo/`에 있는 적절한 영역 정보 파일을 가리킵니다. 설정은 즉시 적용됩니다. 현재 시스템 시간대를 감지하기 위해 `/etc/localtime`을 사용하는 일부 장기 실행 프로세스는 프로세스가 다시 시작될 때까지 시스템 시간대의 변경 사항을 감지하지 못할 수 있습니다.

시간대는 표시 목적이나 사용자 입력 처리를 위해 주로 사용됩니다. 시간대를 변경해도 시스템 시계의 시간은 변경되지 않습니다. `TZ` 환경 변수를 설정하여 모든 콘솔에서 시스템 시간에 대한 표시를 변경할 수 있습니다. 예를 들어 도쿄의 현재 시간을 보려면 다음을 실행하면 됩니다.

```
TZ="*Asia/Tokyo*" date
```

`timedatectl` 명령을 실행하여 시스템의 현재 날짜 및 시간 구성을 확인할 수 있습니다.

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

시스템 시간을 수동으로 설정하려면 `timedatectl set-time` 명령을 사용하세요.

```
timedatectl set-time "*2021-07-17 01:59:59*"
```

이 명령은 현재 설정된 시스템 시간대를 가정하여 지정된 시간을 기준으로 현재 시스템 시간을 설정합니다. 이 명령은 시스템 실시간 시계\(RTC\)도 업데이트합니다.

보다 정확한 시간 유지를 위해 네트워크 시간 동기화를 사용하도록 시스템을 구성하는 것을 고려하십시오. 특히 고가용성을 설정하거나 네트워크 기반 파일 시스템을 사용할 때 네트워크 시간 동기화를 사용하는 것이 중요합니다.

NTP 서비스를 구성하는 경우 다음 명령을 실행하여 NTP를 활성화합니다.

```
timedatectl set-ntp true****
```

이 명령은 가능한 경우 `chronyd` 서비스를 활성화하고 시작합니다.

## Watchdog 서비스 구성

Watchdog은 백그라운드에서 실행되어 호스트 가용성과 프로세스를 모니터링하고 커널에 다시 보고하는 Enterprise Linux 서비스입니다. Watchdog 서비스가 시스템이 정상임을 커널에 알리지 못하는 경우 일반적으로 커널은 시스템을 자동으로 재부팅합니다.

Watchdog 패키지를 설치하려면 다음을 실행하세요.

```
sudo dnf install watchdog
```

Watchdog 서비스를 구성하려면 `/etc/watchdog.conf` 파일을 편집하세요. `watchdog.conf` 파일에는 모든 Watchdog 구성 속성이 포함되어 있습니다. 이 파일을 편집하는 방법에 대한 자세한 내용은 `watchdog.conf(5)` 매뉴얼 페이지를 참조하세요.

Watchdog 서비스를 활성화하고 시작하려면 다음을 실행합니다.

```
sudo systemctl enable --now watchdog
```

Watchdog 서비스는 즉시 시작되어 백그라운드에서 실행됩니다.

**Note:** Watchdog 서비스는 재부팅 후 즉시 시작되고 실행됩니다.
