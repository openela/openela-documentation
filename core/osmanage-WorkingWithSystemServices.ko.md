<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# systemd를 사용하여 시스템 서비스 관리

`systemd` 데몬은 Enterprise Linux의 시스템 초기화 및 서비스 관리자입니다. 이 장에서는 `systemd`를 사용하여 시스템 프로세스, 서비스 및 `systemd` 대상을 관리하는 방법을 설명합니다.

## systemd 서비스 관리자

`systemd` 데몬은 시스템 부팅 후 시작되는 첫 번째 프로세스이며 시스템이 종료될 때 실행되는 마지막 프로세스입니다. `systemd`는 부팅의 마지막 단계를 제어하고 시스템 사용을 준비합니다. 또한 서비스를 동시에 로드하여 부팅 속도를 높입니다.

`systemd`는 `/etc/systemd` 디렉터리의 파일에서 구성을 읽습니다. 예를 들어 `/etc/systemd/system.conf` 파일은 `systemd`가 시스템 초기화를 처리하는 방법을 제어합니다.

`systemd` 데몬은 `/etc/systemd/system/default.target` 심볼릭 링크를 읽어 부팅 프로세스 중에 서비스를 시작합니다. 다음 예에서는 그래픽 사용자 인터페이스 없이  multi-user 모드로 부팅하도록 구성된 시스템(`multi-user.target`이라는 대상)의 `/etc/systemd/system/default.target` 값을 보여줍니다.:

```
sudo ls -l /etc/systemd/system/default.target
```

```nocopybutton
 /etc/systemd/system/default.target -> /usr/lib/systemd/system/multi-user.target 
```

**Note:**

커널 부팅 매개변수를 사용하여 기본 시스템 target을 재정의할 수 있습니다. 참고: [커널 부팅 매개변수](ko-osmanage-WorkingWiththeGRUB2BootloaderandConfiguringBootServices.md#커널-부팅-매개변수).

### systemd Units

`systemd`는 관리하는 다양한 유형의 리소스를 단위(Unit)로 구성합니다. 대부분의 Unit은 시스템 요구 사항에 따라 이러한 Unit을 구성할 수 있는 Unit 구성 파일에 구성됩니다. 파일 외에도 `systemd` 런타임 명령을 사용하여 단위를 구성할 수도 있습니다.

다음 목록에서는 `systemd`를 사용하여 Enterprise Linux 시스템에서 관리할 수 있는 일부 시스템 장치를 설명합니다.:

- **Services**

  서비스 단위 구성 파일의 파일 이름 형식은 _service\_name_.`service`입니다(예: `sshd.service`, `crond.service` 및 `httpd.service`).

  서비스 단위는 데몬과 데몬을 구성하는 프로세스를 시작하고 제어합니다.

  다음 예에서는 Apache HTTP 서버 `httpd.service`에 대한 `systemd` 서비스 단위를 시작하는 방법을 보여줍니다.:

  ```
  sudo systemctl start httpd.service
  ```

- **Targets**

  target 장치 구성 파일의 파일 이름 형식은 _target\_name_.`target`입니다(예: `graphical.target`).

  target은 런레벨과 유사합니다. 리소스가 구성됨에 따라 부팅 프로세스 중에 시스템은 다른 target에 도달합니다. 예를 들어 시스템은 `network-online.target` target에 도달하기 전에 `network-pre.target`에 도달합니다.

  많은 target 단위에는 종속성이 있습니다. 예를 들어, `multi-user.target` \(multi-user 시스템의 경우\)도 활성화되어 있지 않으면 `graphical.target` \(그래픽 세션의 경우\) 활성화가 실패합니다.

- **File System Mount Points**

  마운트 장치 구성 파일의 파일 이름 형식은 _mount\_point\_name_.`mount`입니다.

  마운트 유닛을 사용하면 부팅 시 파일 시스템을 마운트할 수 있습니다. 예를 들어 다음 명령을 실행하여 부팅 시 `/tmp`에 임시 파일 시스템 \(`tmpfs`\)을 마운트할 수 있습니다.:

  ```
  sudo systemctl enable tmp.mount
  ```

- **Devices**

  장치 장치 구성 파일의 파일 이름 형식은 _device\_unit\_name_.`device`입니다.

  장치 장치의 이름은 해당 장치가 제어하는 ​​`/sys` 및 `/dev` 경로를 따라 지정됩니다. 예를 들어 `/dev/sda5` 장치는 systemd에서 `dev-sda5.device`로 노출됩니다.

  장치 단위를 사용하면 장치 기반 활성화를 구현할 수 있습니다.

- **Sockets**

  소켓 장치 구성 파일의 파일 이름 형식은 _socket\_unit\_name_.`socket`입니다.

  소켓의 수신 트래픽에서 서비스가 시작되도록 구성하려면 각 "\*.`socket`" 파일에 해당하는 "\*.`service`" 파일이 필요합니다.

  소켓 유닛을 사용하면 소켓 기반 활성화를 구현할 수 있습니다.

- **Timers**

  타이머 장치 구성 파일의 파일 이름 형식은 _timer\_unit\_name_.`timer`입니다.

  구성된 타이머 이벤트에서 서비스가 시작되도록 구성하려면 각 "\*.`timer`" 파일에 해당하는 "\*.`service`" 파일이 필요합니다. 필요한 경우 `Unit` 구성 항목을 사용하여 타이머 단위와 다르게 이름이 지정된 서비스를 지정할 수 있습니다.

  타이머 장치는 서비스 장치가 실행되는 시기를 제어할 수 있으며 cron 데몬 사용의 대안으로 작동할 수 있습니다. 타이머 단위는 달력 시간 이벤트, 단조로운 시간 이벤트에 대해 구성할 수 있으며 비동기적으로 실행될 수 있습니다.

`systemd` 장치 구성 파일의 경로는 목적과 `systemd`가 `User` 모드에서 실행되는지 `System` 모드에서 실행되는지 여부에 따라 다릅니다. 예를 들어, 패키지에서 설치된 장치에 대한 구성은 `/usr/lib/systemd/system` 또는 `/usr/local/lib/systemd/system`에서 사용할 수 있는 반면, 사용자 모드 구성 단위는 다음과 같습니다. 자세한 내용은 `systemd.unit(5)` 매뉴얼 페이지를 참조하세요.

참조: [System-State Targets](ko-osmanage-WorkingWithSystemServices.md#system-state-targets).

## About System-State Targets

system-state target을 사용하면 특정 목적에 필요한 서비스만 시작하도록 `systemd`를 제어할 수 있습니다. 예를 들어, 시스템 부팅 시 그래픽 사용자 인터페이스가 사용되지 않도록 프로덕션 서버에서 기본 target을 `multi-user.target`으로 설정합니다. 문제를 해결하거나 진단을 수행해야 하는 경우 target을 `rescue.target`으로 설정하는 것을 고려할 수 있습니다.

각 실행 수준은 `systemd`가 중지하거나 시작하는 서비스를 정의합니다. 예를 들어 `systemd`는 `multi-user.target`에 대한 네트워크 서비스와 `graphical.target`에 대한 X Window System을 시작하고 `rescue.target`에 대한 두 서비스를 모두 중지합니다.

[Table 1](ko-osmanage-WorkingWithSystemServices.md#ol-systemctltgt) 일반적으로 사용되는 system-state target과 이에 상응하는 실행 수준 target을 보여줍니다.

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

네트워킹 및 디스플레이 관리자를 사용하여 multi-user 시스템을 설정합니다.

</td></tr><tr><td>

`multi-user.target`

</td><td>

`runlevel2.target`

`runlevel3.target`

`runlevel4.target`

</td><td>

네트워킹을 사용하여 비그래픽 multi-user 시스템을 설정합니다.

</td></tr><tr><td>

`poweroff.target`

</td><td>

`runlevel0.target`

</td><td>

시스템을 종료하고 전원을 끕니다.

</td></tr><tr><td>

`reboot.target`

</td><td>

`runlevel6.target`

</td><td>

시스템을 종료하고 재부팅합니다.

</td></tr><tr><td>

`rescue.target`

</td><td>

`runlevel1.target`

</td><td>

Set up a rescue shell.

</td></tr><tbody></table>
Note that `runlevel*` targets are implemented as symbolic links.

자세한 내용은 `systemd.target(5)` 매뉴얼 페이지를 참조하세요.

### 기본 및 활성 System-State Targets 표시

기본 시스템 상태 target을 표시하려면 `systemctl get-default` 명령을 사용하세요.:

```
sudo systemctl get-default
```

```nocopybutton
graphical.target
```

시스템의 활성 target을 표시하려면 `systemctl list-units --type target` 명령을 사용하세요.:

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

`graphical` target이 활성화된 시스템의 출력은 이 target이 네트워킹과 사운드를 지원하기 위해 `network` 및 `sound`를 포함한 다른 활성 target에 의존한다는 것을 보여줍니다.

목록에 비활성 target을 포함하려면 `--all` 옵션을 사용하세요.

자세한 내용은 `systemctl(1)` 및 `systemd.target(5)` 매뉴얼 페이지를 참조하세요.

**Note:**

target은 `systemd` 유형의 장치 중 하나일 뿐입니다. 모든 유형의 단위를 표시하려면 다음 명령을 사용하십시오.:

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

### 기본 및 활성 System-State Targets 변경

기본 시스템 상태 target을 변경하려면 `systemctl set-default` 명령을 사용하세요.:

```
sudo systemctl set-default multi-user.target
```

```nocopybutton
Removed /etc/systemd/system/default.target. 
Created symlink /etc/systemd/system/default.target → /usr/lib/systemd/system/multi-user.target
```

**Note:**

이 명령은 기본 target이 연결된 대상을 변경하지만 시스템 상태는 변경하지 않습니다.

현재 활성 시스템 target을 변경하려면 `systemctl isolate` 명령을 사용하십시오.

```
sudo systemctl isolate multi-user.target
```

자세한 내용은 `systemctl(1)` 매뉴얼 페이지를 참조하세요.

## 시스템 종료, 일시 중지 및 재부팅

<table><thead><tr><th>

systemctl Command

</th><th>

Description

</th></tr></thead><tbody><tr><td>

`systemctl halt`

</td><td>

시스템을 정지합니다.

</td></tr><tr><td>

`systemctl hibernate`

</td><td>

시스템을 최대 절전 모드로 전환합니다.

</td></tr><tr><td>

`systemctl hybrid-sleep`

</td><td>

시스템을 최대 절전 모드로 전환하고 작동을 일시 중지합니다.

</td></tr><tr><td>

`systemctl poweroff`

</td><td>

시스템을 정지하고 전원을 끕니다.

</td></tr><tr><td>

`systemctl reboot`

</td><td>

시스템을 재부팅합니다.

</td></tr><tr><td>

`systemctl suspend`

</td><td>

시스템을 일시 중단(절전상태전환)합니다.

</td></tr><tbody></table>
For more information, see the `systemctl(1)` manual page.

## 서비스 관리

Enterprise Linux 시스템의 서비스는 `systemctl *subcommand*` 명령으로 관리됩니다.

하위 명령의 예로는 `enable`, `disable`, `stop`, `start`, `restart`, reload 및 `status`가 있습니다.

자세한 내용은 `systemctl(1)` 매뉴얼 페이지를 참조하세요.

### 서비스 시작 및 중지

서비스를 시작하려면 `systemctl start` 명령을 사용하세요.:

```
sudo systemctl start *sshd*
```

서비스를 중지하려면 `systemctl stop` 명령을 사용하세요.:

```
sudo systemctl stop *sshd*
```

서비스 상태 변경은 시스템이 동일한 상태로 유지되는 동안에만 지속됩니다. 서비스를 중지한 다음 system-state target을 서비스가 실행되도록 구성된 상태로 변경하면\(예: 시스템 재부팅\) 서비스가 다시 시작됩니다. 마찬가지로 서비스를 시작해도 재부팅 후 서비스가 시작되지는 않습니다. 참조: [서비스 활성화 및 비활성화](ko-osmanage-WorkingWithSystemServices.md#서비스-활성화-및-비활성화)

### 서비스 활성화 및 비활성화

`systemctl` 명령을 사용하여 시스템 부팅 시 서비스 시작을 활성화하거나 비활성화할 수 있습니다.

```
sudo systemctl enable *httpd*
```

```nocopybutton
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
```

`enable` 명령은 서비스가 시작되어야 하는 가장 낮은 수준의  system-state target에 대한 심볼릭 링크를 생성하여 서비스를 활성화합니다. 이전 예에서 이 명령은 `multi-user` target에 대한 심볼릭 링크 `httpd.service`를 생성합니다.

서비스를 비활성화하면 심볼릭 링크가 제거됩니다.:

```
sudo systemctl disable *httpd*
```

```nocopybutton
Removed /etc/systemd/system/multi-user.target.wants/httpd.service.
```

서비스가 활성화되었는지 확인하려면 다음 예와 같이 `is-enabled` 하위 명령을 사용하세요.:

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

`systemctl 비활성화` 명령을 실행한 후에도 사용자 계정, 스크립트 및 기타 프로세스에 의해 서비스가 시작되거나 중지될 수 있습니다. 그러나 서비스 충돌 등으로 인해 서비스가 실수로 시작될 수 있는지 확인해야 하는 경우 다음과 같이 `systemctl Mask` 명령을 사용하세요.:

```
sudo systemctl mask *httpd*
```

```nocopybutton
Created symlink from '/etc/systemd/system/multi-user.target.wants/httpd.service' to '/dev/null'
```

`mask` 명령은 서비스 참조를 `/dev/null`로 설정합니다. 마스킹된 서비스를 시작하려고 하면 다음 예시와 같은 오류가 발생합니다.:

```
sudo systemctl start *httpd*
```

```nocopybutton
Failed to start httpd.service: Unit is masked.
```

서비스 참조를 일치하는 서비스 단위 구성 파일에 다시 연결하려면 `systemctl unmask` 명령을 사용하세요.:

```
sudo systemctl unmask *httpd*
```

자세한 내용은 `systemctl(1)` 매뉴얼 페이지를 참조하세요.

### 서비스 상태 표시

서비스가 실행 중인지 확인하려면 `is-active` 하위 명령을 사용하세요. 다음 예에 표시된 대로 출력은 _활성_\) 또는 _비활성_이 됩니다.:

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

`status` 하위 명령은 서비스가 구현하는 제어 그룹\(`CGroup`\)의 작업을 표시하는 트리를 포함하여 서비스 상태에 대한 자세한 요약을 제공합니다.:

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

`cgroup`은 시스템 리소스에 대한 액세스를 제어할 수 있도록 함께 바인딩된 프로세스 모음입니다. 이 예에서 `httpd` 서비스의 `cgroup`은 `system` 슬라이스에 있는 `httpd.service`입니다.

슬라이스는 시스템의 `cgroups`을 여러 카테고리로 나눕니다. 슬라이스 및 `cgroup` 계층 구조를 표시하려면 `systemd-cgls` 명령을 사용하십시오.:

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

`system.slice`에는 서비스 및 기타 시스템 프로세스가 포함되어 있습니다. `user.slice`에는 _scopes_라는 임시 cgroup 내에서 실행되는 사용자 프로세스가 포함되어 있습니다. 이 예에서는 ID가 1000인 사용자에 대한 프로세스가 `/user.slice/user-1000.slice` 슬라이스 아래의 `session-7.scope` 범위에서 실행 중입니다.

`systemctl` 명령을 사용하여 서비스 및 범위 cgroup의 프로세스에 사용할 수 있는 CPU, I/O, 메모리 및 기타 리소스를 제한할 수 있습니다. [시스템 리소스에 대한 액세스 제어](ko-osmanage-WorkingWithSystemServices.md#시스템-리소스에-대한-액세스-제어)를 참조하세요.

systemd-분석 확인 /etc/systemd/system/update.\*

### 시스템 리소스에 대한 액세스 제어

예를 들어 `systemctl` 명령을 사용하여 시스템 리소스에 대한 cgroup의 액세스를 제어합니다.:

```
sudo systemctl [--runtime] set-property *httpd* CPUShares=512 MemoryLimit=1G
```

`CPUShare`는 CPU 리소스에 대한 액세스를 제어합니다. 기본값은 1024이므로 값 512는 `cgroup`의 프로세스가 갖는 CPU 시간에 대한 액세스를 절반으로 줄입니다. 마찬가지로 `MemoryLimit`은 `cgroup`이 사용할 수 있는 최대 메모리 양을 제어합니다.

**Note:**

서비스 이름에 `.service` 확장자를 지정할 필요가 없습니다.

`--runtime` 옵션을 지정하면 시스템 재부팅 후에도 설정이 유지되지 않습니다.

또는 `/usr/lib/systemd/system`에 있는 서비스 구성 파일의 `[서비스]` 제목 아래에서 서비스에 대한 리소스 설정을 변경할 수 있습니다. 파일을 편집한 후 `systemd`가 구성 파일을 다시 로드하도록 한 다음 서비스를 다시 시작하세요.:

```
sudo systemctl daemon-reload
sudo systemctl restart *service*
```

범위 내에서 일반 명령을 실행하고 `systemctl`을 사용하여 이러한 임시 cgroup이 시스템 리소스에 대해 갖는 액세스를 제어할 수 있습니다. 범위 내에서 명령을 실행하려면 `systemd-run` 명령을 사용하세요.:

```
sudo systemd-run --scope --unit=*group\_name* [--slice=*slice\_name*]
```

기본 `system` 슬라이스 아래에 그룹을 생성하지 않으려면 다른 슬라이스를 지정하거나 새 슬라이스의 이름을 지정할 수 있습니다. 다음 예에서는 `myslice.slice` 아래의 `mymon.scope`에서 `mymonitor`라는 명령을 실행합니다.:

```
sudo systemd-run --scope --unit=*mymon* --slice=*myslice* mymonitor
```

```nocopybutton
Running as unit mymon.scope.
```

**Note:**

`--scope` 옵션을 지정하지 않으면 제어 그룹은 범위가 아닌 서비스로 생성됩니다.

그런 다음 `systemctl`을 사용하여 서비스와 동일한 방식으로 시스템 리소스에 대한 범위의 액세스를 제어할 수 있습니다. 그러나 서비스와 달리 `.scope` 확장자를 지정해야 합니다.

```
sudo systemctl --runtime set-property *mymon.scope* CPUShares=256
```

자세한 내용은 `systemctl(1)`, `systemd-cgls(1)` 및 `systemd.resource-control(5)` 매뉴얼 페이지를 참조하세요.

### 원격 시스템에서 systemctl 실행

`sshd` 서비스가 원격 Enterprise Linux 시스템에서 실행 중인 경우 `systemctl` 명령과 함께 `-H` 옵션을 지정하여 시스템을 원격으로 제어합니다.

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

자세한 내용은 `systemctl(1)` 매뉴얼 페이지를 참조하세요.

## systemd 서비스 단위 파일 수정

`systemd` 서비스의 구성을 변경하려면 `.service`, `.target`, `.mount` 및 `.socket` 확장자를 가진 파일을 `/usr/lib/systemd/system`에서 `/etc/ 시스템/시스템`.

파일을 복사한 후 `/etc/systemd/system`에서 버전을 편집할 수 있습니다. `/etc/systemd/system`에 있는 파일은 `/usr/lib/systemd/system`에 있는 버전보다 우선합니다. `/usr/lib/systemd/system`의 파일과 관련된 패키지를 업데이트할 때 `/etc/systemd/system`의 파일을 덮어쓰지 않습니다.

특정 서비스에 대한 기본 `systemd` 구성으로 되돌리려면 `/etc/systemd/system`에서 복사본의 이름을 바꾸거나 복사본을 삭제하면 됩니다.

다음 섹션에서는 시스템에 대해 편집하고 사용자 정의할 수 있는 서비스 단위 파일의 다양한 부분을 설명합니다.

### 서비스 단위 파일

서비스는 해당 서비스 단위 파일을 기반으로 실행됩니다. 서비스 단위 파일에는 일반적으로 다음 섹션이 포함되어 있으며, 각 섹션에는 특정 서비스 실행 방법을 결정하는 각각의 정의된 옵션이 있습니다.:

- **`[Unit]`**

  서비스에 대한 정보가 포함되어 있습니다.

- **`[_UnitType_]`:**

  파일의 단위 유형과 관련된 옵션이 포함되어 있습니다. 예를 들어 서비스 단위 파일에서 이 섹션의 제목은 `[Service]`이며 `ExecStart` 또는 `StandardOutput`과 같은 서비스 유형 단위와 관련된 옵션을 포함합니다.

  해당 유형에 특정한 옵션을 제공하는 장치 유형에만 해당 섹션이 있습니다.

- **`[Install]`**

  특정 장치에 대한 설치 정보가 포함되어 있습니다. 이 섹션의 정보는 `systemctl 활성화` 및 `systemctl 비활성화` 명령에 사용됩니다.

서비스 단위 파일에는 서비스에 대한 다음 구성이 포함될 수 있습니다.

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

[서비스 단위 파일의 구성 가능한 옵션](ko-osmanage-WorkingWithSystemServices.md#서비스-단위-파일의-구성-가능한-옵션) 각 섹션에서 일반적으로 사용되는 구성 옵션 중 일부를 설명합니다. 전체 목록은 `systemd.service(5)` 및 `systemd.unit(5)` 매뉴얼 페이지에서도 확인할 수 있습니다.

### 서비스 단위 파일의 구성 가능한 옵션

다음 목록은 각각 서비스 단위 파일의 별도 섹션을 다룹니다.

#### \[Unit\] 섹션 아래 옵션에 대한 설명

다음 목록은 서비스 단위 파일의 `[Unit]` 섹션에서 일반적으로 사용되는 구성 가능 옵션에 대한 일반적인 개요를 제공합니다.:

- **`Description`**

  서비스에 대한 정보를 제공합니다. 해당 정보는 장치에서 `systemctl status` 명령을 실행하면 표시됩니다.

- **`Documentation`**

  이 장치 또는 해당 구성에 대한 문서를 참조하는 공백으로 구분된 URI 목록을 포함합니다.

- **`After`**

  옵션에 나열된 장치가 시작을 마친 후에만 장치가 실행되도록 구성합니다.

  다음 예에서 _var3_.`service` 파일에 다음 항목이 있으면 `*var1*.service` 및 `*var2*.service` 단위가 시작된 후에만 시작됩니다.:

  ```
   After=*var1*.service *var2*.service
  ```

- **`Requires`**

  다른 장치에 대한 요구 사항 종속성을 갖도록 장치를 구성합니다. 유닛이 활성화되면 `Requires` 옵션에 나열된 유닛도 활성화됩니다.

- **`Wants`**

  `Requires` 옵션의 덜 엄격한 버전입니다. 예를 들어 `Wants` 옵션에 나열된 장치 중 하나가 시작되지 않더라도 특정 장치를 활성화할 수 있습니다.

#### \[Service\] 섹션 아래 옵션에 대한 설명

다음 목록은 서비스 단위 파일의 `[Service]` 섹션에서 일반적으로 사용되는 구성 가능한 옵션에 대한 일반적인 개요를 제공합니다.

- **`Type`**

  서비스 단위에 대한 프로세스 시작 유형을 구성합니다.

  이는 서비스의 기본 프로세스가 `ExecStart` 매개변수에 의해 시작되는 프로세스임을 나타냅니다.

  일반적으로 서비스 유형이 `simple`인 경우 파일에서 정의를 생략할 수 있습니다.

- **`StandardOutput`**

  서비스의 이벤트가 기록되는 방식을 구성합니다. 예를 들어 서비스 단위 파일에 다음 항목이 있다고 가정합니다.:

  ```
  StandardOutput=journal
  ```

  예시에서 `journal` 값은 이벤트가 저널에 기록되었음을 나타내며 `journalctl` 명령을 사용하여 볼 수 있습니다.

- **`ExecStart`**

  서비스를 시작하는 전체 경로와 명령을 지정합니다(예: `/usr/bin/npm start`).

- **`ExecStop`**

  `ExecStart`를 통해 시작된 서비스를 중지하기 위해 실행할 명령을 지정합니다.

- **`ExecReload`**

  서비스에서 구성 다시 로드를 트리거하기 위해 실행할 명령을 지정합니다.

- **`Restart`**

  서비스 프로세스가 종료되거나 중지되거나 시간 초과에 도달할 때 서비스를 다시 시작할지 여부를 구성합니다.

  **Note:** 이 옵션은 `systemctl stop` 또는 `systemctl restart`와 같은 `systemd` 작업에 의해 프로세스가 완전히 중지된 경우에는 적용되지 않습니다. 이러한 경우 이 구성 옵션으로 서비스가 다시 시작되지 않습니다.

- **`RemainAfterExit`**

  모든 프로세스가 종료된 경우에도 서비스가 활성 상태로 간주되는지 여부를 구성하는 부울 값입니다. 기본값은 `no`입니다.

#### \[Install\] 섹션 아래 옵션에 대한 설명

다음 목록은 서비스 단위 파일의 `[Intall]` 섹션에서 일반적으로 사용되는 구성 가능한 옵션에 대한 일반적인 개요를 제공합니다.

- **`Alias`**

  공백으로 구분된 장치 이름 목록입니다.

  설치 시 `systemctl 활성화`는 이러한 이름에서 장치 파일 이름으로의 심볼릭 링크를 생성합니다.

  alias는 장치가 활성화된 경우에만 유효합니다.

- **`RequiredBy`**

  다른 장치에 필요한 서비스를 구성합니다.

  예를 들어, 다음 구성이 추가된 유닛 파일 `*var1*.service`를 생각해 보세요.:

  ```
  RequiredBy=*var2*.service *var3*.service
  ```

  `*var1*.service`가 활성화되면 `*var2*.service`와 `*var3*.service` 모두 `*var1*.service`에 대한 `Requires` 종속성이 부여됩니다. 이 종속성은 `*var1*.service` 시스템을 가리키는 각 종속 서비스\(`*var2*.service` 및 `var3.service`\)의 `.requires` 폴더에 생성된 기호 링크로 정의됩니다.

- **`WantedBy`**

  파일을 편집 중인 서비스에 대해 'Wants' 종속성을 부여할 유닛 목록을 지정합니다.

  예를 들어, 다음 구성이 추가된 유닛 파일 `*var1*.service`를 생각해 보세요.:

  ```
  WantedBy=*var2*.service *var3*.service
  ```

  `*var1*.service`가 활성화되면 `*var2*.service`와 `*var3*.service` 모두 `*var1*.service`에 대한 `Wants` 종속성이 부여됩니다. 이 종속성은 `*에 대한 시스템 장치 파일을 가리키는 각 종속 서비스 \(`_var2_.service`및`var3.service`\)의 ".wants`" 폴더에 생성된 기호 링크로 정의됩니다.

- **`Also`**

  장치를 설치하거나 제거할 때 설치하거나 제거할 추가 장치를 나열합니다.

- **`DefaultInstance`**

  `DefaultInstance` 옵션은 템플릿 단위 파일에만 적용됩니다.

  템플릿 단위 파일을 사용하면 단일 구성 파일에서 여러 단위를 생성할 수 있습니다. `DefaultInstance` 옵션은 명시적으로 설정된 인스턴스 없이 템플릿이 활성화된 경우 장치가 활성화되는 인스턴스를 지정합니다.

## 사용자 기반 시스템 서비스 생성

시스템 전체 `systemd` 파일 외에도 `systemd`를 사용하면 루트 액세스 및 권한 없이 사용자 수준에서 실행할 수 있는 사용자 기반 서비스를 만들 수 있습니다. 이러한 사용자 기반 서비스는 사용자 제어 하에 있으며 시스템 서비스와 독립적으로 구성 가능합니다.

다음은 사용자 기반 `systemd` 서비스의 몇 가지 구별되는 특징입니다.

- 사용자 기반 `systemd` 서비스는 특정 사용자 계정과 연결됩니다.
- 연결된 사용자의 홈 디렉터리 `$HOME/.config/systemd/user/` 아래에 생성됩니다.
- 이러한 서비스가 활성화되면 관련 사용자가 로그인할 때 시작됩니다. 이 동작은 시스템 부팅 시 시작되는 활성화된 `systemd` 서비스의 동작과 다릅니다.

사용자 기반 서비스를 생성하려면:

1. 예를 들어 `~/.config/systemd/user` 디렉터리에 서비스의 단위 파일을 만듭니다.:

   ```
   touch ~/.config/systemd/user/*myservice*.service
   ```

2. 유닛 파일을 열고 `Description`, `ExecStart`, `WantedBy` 등과 같이 사용하려는 옵션에 대한 값을 지정합니다.

   자세한 내용은 [서비스 단위 파일의 구성 가능한 옵션](ko-osmanage-WorkingWithSystemServices.md#서비스-단위-파일의-구성-가능한-옵션)을 참조하세요.

3. 로그인할 때 서비스가 자동으로 시작되도록 활성화합니다.

   ```
   sudo systemctl --user enable *myservice*.service
   ```

   **Note:**

   로그아웃하면 루트 사용자가 해당 사용자에 대해 프로세스가 계속 실행되도록 활성화하지 않은 한 서비스가 중지됩니다.

4. 서비스를 시작합니다.

   ```
   sudo systemctl --user start *myservice*.service
   ```

5. 서비스가 실행 중인지 확인합니다.

   ```
   sudo systemctl --user status *myservice*.service
   ```

## 타이머 장치를 사용하여 서비스 장치 런타임 제어

서비스 장치가 실행되는 시기를 제어하도록 타이머 장치를 구성할 수 있습니다. 시간 기반 이벤트에 대해 `cron` 데몬을 구성하는 대신 타이머 장치를 사용할 수 있습니다. 타이머 단위는 crontab 항목을 생성하는 것보다 구성하기가 더 복잡할 수 있습니다. 그러나 타이머 장치는 더 쉽게 구성할 수 있으며 더 나은 로깅과 `systemd` 아키텍처와의 심층 통합을 위해 타이머 장치가 제어하는 ​​서비스를 구성할 수 있습니다.

타이머 장치는 서비스 장치와 유사하게 시작, 활성화 및 중지됩니다. 예를 들어 타이머 장치를 즉시 활성화하고 시작하려면 다음을 입력합니다.:

```
sudo systemctl enable --now *myscript*.timer
```

시스템의 모든 기존 타이머를 나열하고, 마지막으로 실행된 시간과 다음에 실행되도록 구성되는 시간을 확인하려면 다음을 입력하십시오.:

```
systemctl list-timers
```

시스템 타이머에 대한 자세한 내용은 `systemd.timer(5)` 및 `systemd.time(7)` 매뉴얼 페이지를 참조하세요.

### 실시간 타이머 장치 구성

**실시간 타이머**는 crontab의 이벤트와 유사하게 캘린더 이벤트에서 활성화됩니다. `OnCalendar` 옵션은 타이머가 서비스를 실행하는 시기를 지정합니다.

- 필요한 경우 타이머 장치에 의해 트리거될 서비스를 정의하는 `.service` 파일을 만듭니다. 다음 절차에서 샘플 서비스는 업데이트 스크립트를 실행하는 서비스 단위인 `/etc/systemd/system/update.service`입니다.

  서비스 단위 생성에 대한 자세한 내용은 다음을 참조하세요.[사용자 기반 시스템 서비스 생성](ko-osmanage-WorkingWithSystemServices.md#사용자-기반-시스템-서비스-생성).

- 서비스 실행 시간과 빈도를 결정합니다. 이 절차에서는 월요일부터 금요일까지 2시간마다 서비스를 실행하도록 타이머를 구성합니다.

이 작업에서는 달력 이벤트에 따라 서비스가 실행되도록 트리거하는 시스템 타이머를 만드는 방법을 보여줍니다. 캘린더 이벤트의 정의는 크론 작업에 입력하는 항목과 유사합니다.

1. 다음 내용으로 `/etc/systemd/system/update.timer`를 생성합니다.:

   ```
   [Unit]
   Description="Run the update.service every two hours from Mon to Fri."

   [Timer]
   OnCalendar=Mon..Fri 00/2 
   Unit=update.service

   [Install]
   WantedBy=multi-user.target
   ```

   `OnCalendar` 정의는 더 자세한 `OnCalendar=weekly` 정의와 같은 단순한 젖음부터 다양할 수 있습니다. 그러나 설정을 정의하는 형식은 다음과 같이 일정합니다.:

   ```
   DayofWeek Year-Month-Day Hour:Minute:Second
   ```

   다음 정의는 "매월 첫 4일 정오 12시(단, 해당 날짜가 월요일 또는 화요일인 경우에만 해당)"를 의미합니다.:

   ```
   OnCalendar=Mon,Tue *-*-01..04 12:00:00
   ```

   `OnCalendar`를 정의하는 다른 방법과 시스템 타이머 파일에서 구성할 수 있는 추가 타이머 옵션에 대해서는 `systemd.timer(5)` 및 `systemd.time(7)` 매뉴얼 페이지를 참조하세요.

2. 이 타이머와 관련된 모든 파일이 올바르게 구성되었는지 확인하십시오.

   ```
   systemd-analyze verify /etc/systemd/system/update.*
   ```

   감지된 오류는 화면에 보고됩니다.

3. 타이머를 시작하세요.

   ```
   sudo systemctl start update.timer
   ```

   이 명령은 현재 세션에 대해서만 타이머를 시작합니다.

4. 시스템이 부팅될 때 타이머가 시작되는지 확인합니다.

   ````
   sudo systemctl 활성화 update.timer
   ```

   ````

### Monotonic 타이머 장치 구성

**Monotonic 타이머**는 부팅 이벤트와 같은 다양한 시작 지점을 기준으로 일정 시간이 지난 후 또는 특정 `systemd` 장치가 활성화될 때 활성화됩니다. 컴퓨터가 일시적으로 중단되거나 종료되면 이러한 타이머 장치가 중지됩니다. 여기서 _Type_은 타이머와 관련된 이벤트의 이름입니다. 일반적인 단조 타이머에는 `OnBootSec` 및 `OnUnitActiveSec`이 포함됩니다.

- 필요한 경우 타이머 장치에 의해 트리거될 서비스를 정의하는 `.service` 파일을 만듭니다. 다음 절차에서 샘플 서비스는 업데이트 스크립트를 실행하는 서비스 단위인 `/etc/systemd/system/update.service`입니다.

  서비스 단위 생성에 대한 자세한 내용은 다음을 참조하세요.[사용자 기반 시스템 서비스 생성](ko-osmanage-WorkingWithSystemServices.md#사용자-기반-시스템-서비스-생성).

- 서비스 실행 시간과 빈도를 결정합니다. 이 절차에서는 시스템 부팅 후 10분, 서비스가 마지막으로 활성화된 후 2시간마다 서비스를 실행하도록 타이머가 구성됩니다.

이 작업에서는 시스템이 부팅될 때 또는 타이머 활성화 후 2시간이 경과한 후의 특정 이벤트에서 서비스가 실행되도록 트리거하는 시스템 타이머를 생성하는 방법을 보여줍니다.

1. 다음 내용으로 `/etc/systemd/system/update.timer`를 생성합니다.:

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

2. 이 타이머와 관련된 모든 파일이 올바르게 구성되었는지 확인하십시오.

   ```
   systemd-analyze verify /etc/systemd/system/update.*
   ```

   감지된 오류는 화면에 보고됩니다.

3. 타이머를 시작하세요.

   ```
   sudo systemctl start update.timer
   ```

   이 명령은 현재 세션에 대해서만 타이머를 시작합니다.

4. 시스템이 부팅될 때 타이머가 시작되는지 확인합니다.

   ````
   sudo systemctl 활성화 update.timer
   ```

   ````

### 임시 타이머 장치 실행

임시 타이머는 현재 세션에만 유효한 임시 타이머입니다. 이러한 타이머는 `systemd` 내에서 서비스나 타이머 장치를 구성할 필요 없이 프로그램이나 스크립트를 직접 실행하도록 생성될 수 있습니다. 이러한 단위는 `systemd-run` 명령을 사용하여 생성됩니다. 자세한 내용은 `systemd-run(1)` 매뉴얼 페이지를 참조하세요.

`*unit-file*.timer` 파일에 추가하는 매개변수 옵션은 임시 타이머 단위를 실행하기 위해 `systemd-run` 명령을 사용할 때 인수 역할도 합니다.

다음 예에서는 `systemd-run`을 사용하여 임시 타이머를 활성화하는 방법을 보여줍니다.

- 2시간이 지난 후 `update.service`를 실행하세요.

  ```
  sudo systemd-run --on-active="2h" --unit update.service
  ```

- 1시간 후에 `~/tmp/myfile`을 생성합니다.

  ```
  sudo systemd-run --on-active="1h" /bin/touch ~/tmp/myfile
  ```

- 서비스 관리자가 시작된 후 5분 후에 `~/myscripts/update.sh`를 실행합니다. 사용자 로그인 시 서비스 관리자가 시작된 후 서비스를 실행하려면 이 구문을 사용합니다.

  ```
  sudo systemd-run --on-startup="5m" ~/myscripts/update.sh
  ```

- 시스템 부팅 후 10분 후에 `myjob.service`를 실행합니다.

  ```
  sudo systemd-run --on-boot="10m" --unit myjob.service
  ```

- 하루가 끝나면 `report.service`를 실행하세요.

  ```
  sudo systemd-run --on-calendar="17:00:00"
  ```
