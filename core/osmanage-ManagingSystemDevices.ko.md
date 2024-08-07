<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# 시스템 장치 관리

이 장에서는 시스템이 장치 파일을 사용하는 방법과 Udev 장치 관리자가 장치 노드 파일을 동적으로 생성하거나 제거하는 방법을 설명합니다.

## 장치 파일

`/dev` 디렉토리에는 하드 디스크와 같은 주변 장치, 디스크 파티션과 같은 주변 장치의 리소스, 난수 생성기와 같은 의사 장치에 대한 액세스를 제공하는 _장치 파일_ 또는 _장치 노드_가 포함되어 있습니다.

`/dev` 디렉토리에는 여러 하위 디렉토리 계층이 있으며 각 하위 디렉토리에는 특정 유형의 장치와 관련된 장치 파일이 들어 있습니다. 그러나 이러한 하위 디렉터리의 내용은 `/dev`의 해당 파일에 대한 심볼릭 링크로 구현됩니다. 따라서 파일은 `/dev`에 연결된 파일이나 하위 디렉터리의 해당 파일을 통해 액세스할 수 있습니다.

`ls -l /dev` 명령을 사용하면 파일 목록이 표시되며, 그 중 일부는 `b` 유형(_블록_\의 경우) 또는 `c` 유형(_문자_\의 경우)으로 표시됩니다. 이러한 장치에는 시스템에서 장치를 식별하는 연관된 번호 쌍이 있습니다.

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

블록 장치는 데이터에 대한 임의 액세스를 지원하여 데이터용 미디어를 찾고 일반적으로 데이터를 쓰거나 읽는 동안 데이터를 버퍼링합니다. 블록 장치의 예로는 하드 디스크, CD-ROM 드라이브, 플래시 메모리 및 기타 주소 지정이 가능한 메모리 장치가 있습니다.

문자 장치는 장치와의 데이터 스트리밍을 지원합니다. 데이터는 일반적으로 버퍼링되지 않으며 장치의 데이터에 대한 임의 액세스가 허용되지 않습니다. 커널은 한 번에 1바이트씩 문자 장치에 데이터를 쓰거나 읽습니다. 문자 장치의 예로는 키보드, 마우스, 터미널, 의사 터미널 및 테이프 드라이브가 있습니다. `tty0`과 `tty1`은 사용자가 직렬 터미널이나 터미널 에뮬레이터에서 로그인할 수 있도록 터미널 장치에 해당하는 문자 장치 파일입니다.

모조 터미널 (Pseudo terminal - Pty) 보조 장치는 실제 터미널 장치를 에뮬레이트하여 소프트웨어와 상호 작용합니다. 예를 들어, 사용자는 `/dev/tty1`과 같은 터미널 장치에 로그인한 다음 의사 터미널 기본 장치인 `/dev/pts/ptmx`를 사용하여 기본 모조 터미널 장치와 상호 작용할 수 있습니다. 모조 터미널 보조 및 기본 장치의 문자 장치 파일은 다음 예와 같이 `/dev/pts` 디렉터리에 있습니다.:

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

표준 입력용 `stdin`과 같은 일부 장치 항목은 `proc` 파일 시스템의 `self` 하위 디렉터리를 통해 기호적으로 연결됩니다. 실제로 가리키는 의사 터미널 장치 파일은 프로세스의 컨텍스트에 따라 다릅니다.

```
ls -l /proc/self/fd/[012]
```

```nocopybutton
lrwx------. 1 root root 64 Oct  7 08:23 /proc/self/fd/0 -> /dev/pts/0
lrwx------. 1 root root 64 Oct  7 08:23 /proc/self/fd/1 -> /dev/pts/0
lrwx------. 1 root root 64 Oct  7 08:23 /proc/self/fd/2 -> /dev/pts/0

```

`null`, `random`, `urandom` 및 `zero`와 같은 문자 장치는 물리적 하드웨어가 아닌 소프트웨어에 구현된 가상 기능에 대한 액세스를 제공하는 모조 장치의 예입니다.

`/dev/null`은 데이터 싱크입니다. `/dev/null`에 쓴 데이터는 사실상 사라지지만 쓰기 작업은 성공합니다. `/dev/null`에서 읽으면 `EOF`\(파일 끝\)이 반환됩니다.

`/dev/zero`는 0 바이트 값을 무제한으로 사용할 수 있는 데이터 소스입니다.

`/dev/random` 및 `/dev/urandom`은 모조 난수 바이트 스트림의 데이터 소스입니다. 높은 엔트로피 출력을 유지하기 위해 엔트로피 풀에 충분한 노이즈 비트가 포함되어 있지 않으면 `/dev/random`이 차단됩니다. `/dev/urandom`은 차단되지 않으므로 출력의 엔트로피가 `/dev/random`의 엔트로피만큼 일관되게 높지 않을 수 있습니다. 그러나 `/dev/random` 또는 `/dev/urandom`은 군사보안급 암호화와 같은 보안 암호화 목적에 충분히 무작위인 것으로 간주되지 않습니다.

`/proc/sys/kernel/random` 아래의 가상 파일에서 엔트로피 풀의 크기와 `/dev/random`에 대한 엔트로피 값을 확인할 수 있습니다.:

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

자세한 내용은 `null(4)`, `pts(4)` 및 `random(4)` 매뉴얼 페이지를 참조하세요.

## Udev 장치 관리자

Udev 장치 관리자는 부팅 시 장치 노드 파일을 동적으로 생성하거나 제거합니다. 장치 노드를 생성할 때 `udev`는 장치의 `/sys` 디렉터리에서 레이블, 일련 번호, 버스 장치 번호와 같은 속성을 읽습니다.

Udev는 검색 순서에 관계없이 재부팅 시 일관된 장치 이름 지정을 보장하기 위해 영구 장치 이름을 사용할 수 있습니다. 영구 장치 이름은 외부 저장 장치를 사용할 때 특히 중요합니다.

`udev`에 대한 구성 파일은 `/etc/udev/udev.conf`이며, 여기서 `udev_log` 로깅 우선순위를 정의할 수 있으며 `err`, `info` 및 `debug`로 설정할 수 있습니다. 기본값은 `err`입니다.

자세한 내용은 `udev(7)` 매뉴얼 페이지를 참조하세요.

## Udev 규칙

`udev` 서비스\(`systemd-udevd`\)는 시스템 시작 시 규칙 파일을 읽고 규칙을 메모리에 저장합니다. 커널이 새 장치를 발견하거나 기존 장치가 오프라인 상태가 되면 커널은 `/sys`의 장치 속성에 대해 메모리 내 규칙과 일치하는 이벤트 작업 \(_uevent_\) 알림을 `udev`에 보냅니다.

여러 규칙 파일이 서로 다른 디렉터리에 존재합니다. 그러나 `/etc/udev/rules.d/*.rules` 파일은 수정할 수 있는 유일한 규칙 파일이므로 이것만 알면 됩니다. [Udev 규칙 수정](ko-osmanage-ManagingSystemDevices.md#udev-규칙-수정).

Udev는 규칙 파일의 디렉터리에 관계없이 어휘 순서로 규칙 파일을 처리합니다. `/etc/udev/rules.d`에 있는 규칙 파일은 다른 위치에 있는 동일한 이름의 규칙 파일을 재정의합니다.

다음 규칙은 `/lib/udev/rules.d/50-udev-default.rules` 파일에서 추출되며 udev 규칙의 구문을 보여줍니다.:

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

규칙은 키에 값을 할당하거나 현재 값을 지정된 값과 비교하여 키와 일치하는 항목을 찾으려고 시도합니다. 다음 표에서는 사용할 수 있는 할당 및 비교 연산자를 보여줍니다.

<table><thead><tr><th>

Operator

</th><th>

Description

</th></tr></thead><tbody><tr><td>

`=`

</td><td>

키에 값을 할당하여 이전 값을 덮어씁니다.

</td></tr><tr><td>

`+=`

</td><td>

키의 현재 값 목록에 값을 추가하여 값을 할당합니다.

</td></tr><tr><td>

`:=`

</td><td>

키에 값을 할당합니다. 이 값은 추가 규칙에 의해 변경될 수 없습니다.

</td></tr><tr><td>

`==`

</td><td>

키의 현재 값을 지정된 값과 일치하는지 비교합니다.

</td></tr><tr><td>

`!=`

</td><td>

키의 현재 값을 지정된 값과 일치하는지 않는지 비교합니다.

</td></tr><tbody></table>
You can use the following shell-style pattern-matching characters in values.

<table><thead><tr><th>

Character

</th><th>

Description

</th></tr></thead><tbody><tr><td>

`?`

</td><td>

단일 문자와 일치합니다.

</td></tr><tr><td>

`*`

</td><td>

0을 포함하여 모든 문자 수와 일치합니다.

</td></tr><tr><td>

`[]`

</td><td>

단일 문자 또는 대괄호 안에 지정된 문자 범위의 문자와 일치합니다. 예를 들어 `tty[sS][0-9]`는 `ttys7` 또는 `ttyS7`과 일치합니다.

</td></tr><tbody></table>
The following table describes commonly used match keys in rules.

<table><thead><tr><th>

Match Key

</th><th>

Description

</th></tr></thead><tbody><tr><td>

`ACTION`

</td><td>

이벤트를 발생시킨 작업의 이름과 일치합니다. 예를 들어 `ACTION="추가"` 또는 `ACTION="제거"`입니다.

</td></tr><tr><td>

`ENV{*key*}`

</td><td>

장치 속성 _key_의 값과 일치합니다. 예를 들어 `ENV{DEVTYPE}=="disk"`입니다.

</td></tr><tr><td>

`KERNEL`

</td><td>

이벤트의 영향을 받는 장치의 이름과 일치합니다. 예를 들어 디스크 미디어의 경우 `KERNEL=="dm-*"`입니다.

</td></tr><tr><td>

`NAME`

</td><td>

장치 파일 또는 네트워크 인터페이스의 이름과 일치합니다. 예를 들어 하나 이상의 문자로 구성된 이름의 경우 `NAME="?*"`입니다.

</td></tr><tr><td>

`SUBSYSTEM`

</td><td>

이벤트의 영향을 받는 장치의 하위 시스템과 일치합니다. 예를 들어 `SUBSYSTEM=="tty"`입니다.

</td></tr><tr><td>

`TEST`

</td><td>

예를 들어 `TEST=="/lib/udev/devices/$name"`, 여기서 `$name`은 현재 일치하는 장치 파일의 이름입니다.

</td></tr><tbody></table>
Other match keys include `ATTR{*filename*}`, `ATTRS{*filename*}`, `DEVPATH`, `DRIVER`, `DRIVERS`, `KERNELS`, `PROGRAM`, `RESULT`, `SUBSYSTEMS`, and `SYMLINK`.

다음 표에서는 규칙에서 일반적으로 사용되는 할당 키를 설명합니다.

<table><thead><tr><th>

Assignment Key

</th><th>

Description

</th></tr></thead><tbody><tr><td>

`ENV{*key*}`

</td><td>

`GROUP="disk"`와 같은 장치 속성 _key_의 값을 지정합니다.

</td></tr><tr><td>

`GROUP`

</td><td>

`GROUP="disk"`와 같은 장치 파일의 그룹을 지정합니다.

</td></tr><tr><td>

`IMPORT{*type*}`

</td><td>

_type_에 따라 장치 속성에 대한 변수 세트를 지정합니다.

- **`cmdline`**

부팅 `커널` 커맨드라인에서 단일 속성을 가져옵니다. 간단한 플래그의 경우 `udev`는 속성 값을 1로 설정합니다. 예를 들어 `IMPORT{cmdline}="nodmraid"`입니다.

- **`db`**

지정된 값을 장치 데이터베이스에 대한 인덱스로 해석하고 이전 이벤트에 의해 이미 설정되어 있어야 하는 단일 속성을 가져옵니다. 예를 들어 `IMPORT{db}="DM_UDEV_LOW_PRIORITY_FLAG"`입니다.

- **`file`**

지정된 값을 텍스트 파일의 이름으로 해석하고 환경 키 형식이어야 하는 해당 내용을 가져옵니다. 예를 들어 `IMPORT{file}="keyfile"`입니다.

- **`parent`**

지정된 값을 키 이름 필터로 해석하고 상위 장치에 대한 데이터베이스 항목에서 저장된 키를 가져옵니다. 예를 들어 `IMPORT{parent}="ID_*"`입니다.

- **`program`**

지정된 값을 외부 프로그램으로 실행하고 해당 결과를 가져옵니다. 예를 들어 `IMPORT{program}="usb_id --export %p"`입니다.

</td></tr><tr><td>

`MODE`

</td><td>

`MODE="0640"`과 같은 장치 파일에 대한 권한을 지정합니다.

</td></tr><tr><td>

`NAME`

</td><td>

`NAME="em1"`과 같은 장치 파일의 이름을 지정합니다.

</td></tr><tr><td>

`OPTIONS`

</td><td>

`OPTIONS ="ignore_remove"`와 같은 규칙 및 장치 옵션을 지정합니다.

</td></tr><tr><td>

`OWNER`

</td><td>

`GROUP="root"`와 같은 장치 파일의 소유자를 지정합니다.

</td></tr><tr><td>

`RUN`

</td><td>

`RUN ="/usr/bin/eject $kernel"`과 같이 장치 파일이 생성된 후에 실행할 명령을 지정합니다.

</td></tr><tr><td>

`SYMLINK`

</td><td>

`SYMLINK ="disk/by-uuid/$env{ID_FS_UUID_ENC}"`와 같이 장치 파일에 대한 심볼릭 링크의 이름을 지정합니다.

</td></tr><tbody></table>
Other assignment keys include `ATTR{*key*}`, `GOTO`, `LABEL`, `RUN`, and `WAIT_FOR`.

다음 표에서는 `GROUP`, `MODE`, `NAME`, `OWNER`, `PROGRAM`, `RUN` 및 `SYMLINK` 키와 함께 일반적으로 사용되는 문자열 대체를 설명합니다.

<table><thead><tr><th>

String Substitution

</th><th>

Description

</th></tr></thead><tbody><tr><td>

`$attr{*file*}` or

`%s{*file*}`

</td><td>

`ENV{MATCHADDR}="$attr{address}"`와 같이 `/sys` 아래 파일의 장치 속성 값을 지정합니다.

</td></tr><tr><td>

`$devpath` or

`%p`

</td><td>

`/sys` 아래의 `sysfs` 파일 시스템에 있는 장치의 장치 경로입니다(예: `RUN ="keyboard-force-release.sh $devpath common-volume-keys"`).

</td></tr><tr><td>

`$env{*key*}` or

`%E{*key*}`

</td><td>

`SYMLINK ="disk/by-id/md-name-$env{MD_NAME}-part%n"`과 같은 장치 속성 값을 지정합니다.

</td></tr><tr><td>

`$kernel` or

`%k`

</td><td>

장치의 커널 이름을 지정합니다.

</td></tr><tr><td>

`$major` or

`%M`

</td><td>

`IMPORT{program}="udisks-dm-export %M %m"`과 같이 장치의 주요 번호를 지정합니다.

</td></tr><tr><td>

`$minor` or

`%m`

</td><td>

`RUN ="$env{LVM_SBIN_PATH}/lvm pvscan --cache --major $major --minor $minor"`와 같이 장치의 부 번호를 지정합니다.

</td></tr><tr><td>

`$name`

</td><td>

`TEST=="/lib/udev/devices/$name"`과 같이 현재 장치의 장치 파일을 지정합니다.

</td></tr><tbody></table>
Udev expands the strings specified for `RUN` immediately before its program is run, which is after udev has finished processing all other rules for the device. For the other keys, `udev` expands the strings while it's processing the rules.

자세한 내용은 `udev(7)` 매뉴얼 페이지를 참조하세요.

## Udev 및 Sysfs 쿼리

`udevadm` 명령을 사용하여 `udev` 데이터베이스와 `sysfs`를 쿼리할 수 있습니다.

장치 파일 `/dev/sda`에 해당하는 `/sys`를 기준으로 `sysfs` 장치 경로를 쿼리하려면:

```
udevadm info --query=path --name=/dev/sda
```

```nocopybutton
/devices/pci0000:00/0000:00:0d.0/host0/target0:0:0/0:0:0:0/block/sda
```

`/dev/sda`를 가리키는 심볼릭 링크를 쿼리하려면 다음 명령을 사용하십시오.:

```
udevadm info --query=symlink --name=/dev/sda
```

```nocopybutton
block/8:0
disk/by-id/ata-VBOX_HARDDISK_VB6ad0115d-356e4c09
disk/by-id/scsi-SATA_VBOX_HARDDISK_VB6ad0115d-356e4c09
disk/by-path/pci-0000:00:0d.0-scsi-0:0:0:0
```

`/dev/sda`의 속성을 쿼리하려면 다음 명령을 사용하십시오.:

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

`/dev/sda`에 대한 전체 정보를 쿼리하려면 다음 명령을 사용하십시오.:

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

`/dev/sda`의 모든 속성과 `udev`가 `/sys`에서 찾은 상위 장치를 표시하려면 다음 명령을 사용하십시오.:

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

명령은 장치 경로에 지정된 장치에서 시작하여 상위 장치 체인 위로 이동합니다. 명령이 찾은 모든 장치에 대해 명령은 `udev` 규칙의 일치 키 형식을 사용하여 장치 및 해당 상위 장치에 대해 가능한 속성을 표시합니다.

자세한 내용은 `udevadm(8)` 매뉴얼 페이지를 참조하세요.

## Udev 규칙 수정

규칙이 평가되는 순서가 중요합니다. Udev는 규칙을 어휘순으로 처리합니다. 사용자 정의 규칙을 추가하려면 기본 규칙보다 먼저 이러한 규칙을 찾고 평가하기 위해 `udev`가 필요합니다.

다음 예에서는 디스크 장치 `/dev/sdb`에 심볼릭 링크를 추가하는 `udev` 규칙 파일을 구현하는 방법을 보여줍니다.

1. udev가 다른 규칙 파일보다 먼저 읽는 `10-local.rules`와 같은 파일 이름을 사용하여 `/etc/udev/rules.d` 아래에 규칙 파일을 만듭니다.

   `10-local.rules`의 다음 규칙은 `/dev/sdb`를 가리키는 심볼릭 링크 `/dev/my_disk`를 생성합니다.:

   ```
   KERNEL=="sdb", ACTION=="add", SYMLINK="my_disk"
   ```

   `/dev`에 장치 파일을 나열하면 `udev`가 아직 규칙을 적용하지 않았음을 알 수 있습니다.:

   ```
   ls /dev/sd* /dev/my_disk
   ```

   ```nocopybutton
   ls: cannot access /dev/my_disk: No such file or directory
   /dev/sda  /dev/sda1  /dev/sda2  /dev/sdb
   ```

2. `udev`가 규칙을 적용하여 장치를 생성하는 방법을 시뮬레이션하려면 `/sys/class/block` 계층 아래에 ​​나열된 `sdb` 장치 경로와 함께 `udevadm test` 명령을 사용할 수 있습니다.

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

3. `systemd-udevd` 서비스를 다시 시작하세요.:

   ```
   sudo systemctl restart systemd-udevd
   ```

   `udev`가 규칙 파일을 처리한 후 `/dev/my_disk` 심볼릭 링크가 추가되었습니다.:

   ```
   ls -F /dev/sd* /dev/my_disk
   ```

   ```nocopybutton
   /dev/my_disk@  /dev/sda  /dev/sda1  /dev/sda2  /dev/sdb
   ```

4. \(선택 사항\) 변경 사항을 취소하려면 `/etc/udev/rules.d/10-local.rules` 및 `/dev/my_disk`를 제거한 다음 `systemctl restart systemd-udevd`를 다시 실행하세요.
