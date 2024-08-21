<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# 리소스 관리

이 장에서는 Enterprise Linux 시스템에서 리소스 사용을 관리하는 방법을 설명합니다.

## 컨트롤 그룹

컨트롤 그룹\(`cgroups`\)은 CPU, 메모리, 네트워크 등과 같은 리소스 모음으로, 애플리케이션과 프로세스에서 이러한 리소스를 사용하는 방법에 대한 집합적 설정을 갖습니다. `Cgroups`은 제한을 설정하고, 우선 순위를 지정하고, 격리하거나 리소스를 할당하여 리소스 사용을 추가로 관리할 수 있는 Enterprise Linux의 기능을 구성합니다. 따라서 보다 효율적인 시스템 성능을 얻기 위해 세부적인 수준에서 리소스 사용을 제어할 수 있습니다. 컨트롤 그룹은 애플리케이션이 리소스 사용을 놓고 경쟁하는 여러 가상 머신, Kubernetes 클러스터 등을 호스팅하는 시스템 구성에서 중요합니다.

Enterprise Linux는 두 가지 유형의 컨트롤 그룹을 지원합니다.:

- **컨트롤 그룹 버전 1 \(`cgroups v1`\)**

  이러한 그룹은 리소스별 컨트롤러 계층 구조를 제공합니다. CPU, 메모리, I/O 등과 같은 각 리소스에는 자체 컨트롤 그룹 계층 구조가 있습니다. 이 그룹의 단점은 서로 다른 프로세스 계층에 속할 수 있는 그룹 간에 리소스 사용을 적절하게 조정하는 것이 어렵다는 것입니다.

- **컨트롤 그룹 버전 2 \(`cgroups v2)`**

  이러한 그룹은 모든 리소스 컨트롤러가 마운트되는 단일 컨트롤 그룹 계층 구조를 제공합니다. 이 계층 구조에서는 다양한 리소스 컨트롤러에서 리소스 사용을 보다 적절하게 조정할 수 있습니다. 이 버전은 과도한 유연성으로 인해 시스템 소비자 간의 리소스 사용이 적절하게 조정되지 않는 `cgroups v1`보다 개선되었습니다.

두 버전 모두 Enterprise Linux에 있습니다. 그러나 Enterprise Linux 9에서는 `cgroups v2`이 Default로 활성화되고 마운트되어 사용됩니다.

두 버전의 컨트롤 그룹에 대한 자세한 내용은 `cgroups(7)` 및 `sysfs(5)` 매뉴얼 페이지를 참조하세요.

## 커널 리소스 컨트롤러

컨트롤 그룹은 _커널 리소스 컨트롤러_를 통해 리소스 사용을 관리합니다. 커널 리소스 컨트롤러는 CPU 시간, 메모리, 네트워크 대역폭 또는 디스크 I/O와 같은 단일 리소스를 나타냅니다.

시스템에 마운트된 리소스 컨트롤러를 식별하려면 `/procs/cgroups` 파일의 내용을 확인하십시오.

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

`cgroups v1`과 `cgroups v2`의 커널 리소스 컨트롤러에 대한 자세한 설명은 `cgrouops(7)` 매뉴얼 페이지를 참조하세요.

## 컨트롤 그룹 파일 시스템

Enterprise Linux에서 `cgroup` 기능은 `/sys/fs/cgroup`에 파일 시스템으로 마운트됩니다. 이 디렉터리는 루트 컨트롤 그룹이라고도 합니다. 디렉토리의 내용은 시스템에 마운트된 `cgroup` 버전에 따라 다릅니다. `cgroups v2`의 경우 디렉터리 내용은 다음과 같습니다.:

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

새 컨트롤 그룹을 생성하려면 루트 컨트롤 그룹에 하위 그룹이나 하위 디렉터리를 생성하십시오.

```
sudo mkdir /sys/fs/cgroup/MyGroup
```

하위 그룹은 새 그룹에 대해 활성화한 리소스 컨트롤러로 자동으로 채워집니다. [CPU 시간 분배를 위한 제어 그룹 준비](ko-osmanage-ManagingResources.md#cpu-시간-분배를-위한-제어-그룹-준비).

하위 그룹을 삭제하려면 하위 그룹 자체에 다른 하위 그룹이 포함되어 있지 않은지 확인한 다음 루트 컨트롤 그룹에서 디렉터리를 제거합니다.

```
sudo rm -rf /sys/fs/cgroup/MyGroup
```

## 컨트롤 그룹 및 systemd

컨트롤 그룹은 리소스 관리를 위해 `systemd` 시스템 및 서비스 관리자에서 사용할 수 있습니다. `Systemd`는 이러한 그룹을 사용하여 리소스를 소비하는 장치와 서비스를 구성합니다. [systemd 서비스 관리자](ko-osmanage-WorkingWithSystemServices.md#systemd-서비스-관리자).

`Systemd`는 다양한 단위 유형을 지원하며 그 중 3개는 리소스 제어 목적으로 사용됩니다.:

- **Service**: 설정이 장치 구성 파일을 기반으로 하는 프로세스 또는 프로세스 그룹입니다. 서비스는 지정된 프로세스를 "컬렉션"으로 포함하므로 `systemd`가 프로세스를 하나의 세트로 시작하거나 중지할 수 있습니다. 서비스 이름은 `*name*.service` 형식을 따릅니다.

- **Scope**: 사용자 세션, 컨테이너, 가상 머신 등과 같이 외부에서 생성된 프로세스 그룹입니다. 서비스와 마찬가지로 범위는 생성된 프로세스를 캡슐화하고 임의 프로세스에 의해 시작 또는 중지된 다음 런타임 시 `systemd`에 의해 등록됩니다. 범위 이름은 `*name*.scope` 형식을 따릅니다.

- **Slice**: 서비스와 범위가 위치한 계층적으로 구성된 단위 그룹입니다. 따라서 슬라이스 자체에는 프로세스가 포함되어 있지 않습니다. 오히려 슬라이스의 범위와 서비스가 프로세스를 정의합니다. 슬라이스 단위의 모든 이름은 계층 구조의 특정 위치에 대한 경로에 해당합니다. 루트 슬라이스(일반적으로 모든 사용자 기반 프로세스의 경우 `user.slice`, 시스템 기반 프로세스의 경우 `system.slice`)는 계층 구조에 자동으로 생성됩니다. 상위 슬라이스는 루트 슬라이스 바로 아래에 존재하며 `*parent-name*.slice` 형식을 따릅니다. 그러면 이러한 루트 슬라이스는 여러 수준의 하위 슬라이스를 가질 수 있습니다.

서비스, ​​범위 및 조각 단위는 컨트롤 그룹 계층 구조의 개체에 직접 매핑됩니다. 이러한 유닛이 활성화되면 유닛 이름으로 구성된 컨트롤 그룹 경로에 직접 매핑됩니다. `systemd` 리소스 단위 유형과 컨트롤 그룹 간의 매핑을 표시하려면 다음을 입력합니다.:

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

`systemctl`과 같은 `systemd` 명령을 사용하여 리소스를 관리하는 방법에 대한 예는 [시스템 리소스에 대한 액세스 제어](ko-osmanage-WorkingWithSystemServices.md#시스템-리소스에-대한-액세스-제어)를 참조하세요. `systemctl(1)`, `systemd-cgls(1)`, and `systemd.resource-control(5)` manual pages.

## 리소스 분배 모델

다음 분배 모델은 `cgroups v2`에서 사용할 리소스 분배에 대한 제어 또는 규제를 구현하는 방법을 제공합니다.:

- **Weights**

  이 모델에서는 모든 컨트롤 그룹의 가중치가 합산됩니다. 각 그룹은 전체 가중치에 대한 그룹의 가중치 비율에 따라 자원의 일부를 받습니다.

  10개의 컨트롤 그룹을 고려하면 각각의 가중치는 100이므로 총합은 1000이 됩니다. 이 경우 각 그룹은 지정된 리소스의 10분의 1을 사용할 수 있습니다.

  가중치는 일반적으로 상태 비저장 리소스를 배포하는 데 사용됩니다. 이 리소스를 적용하려면 `CPUWeight` 옵션이 사용됩니다.

- **Limits**

  이 분배 모델을 구현하기 위해 `MemoryMax` 옵션이 사용됩니다.

- **Protections**

  이 모델에서는 그룹에 _보호된 경계_가 할당됩니다. 그룹의 리소스 사용량이 보호된 양 내에서 유지되는 경우 커널은 동일한 리소스를 놓고 경쟁하는 다른 그룹을 위해 그룹의 리소스 사용을 박탈할 수 없습니다. 이 모델에서는 리소스의 과도한 할당이 허용됩니다.

  이 모델을 구현하기 위해 `MemoryLow` 옵션이 사용됩니다.

- **Allocations**

  이 모델에서는 real-time budget과 같은 유한한 유형의 리소스 사용을 위해 특정의 절대 양이 할당됩니다.

## cgroups v2를 사용하여 애플리케이션 리소스 관리

이 섹션에서는 `cgroups v2`를 사용하여 시스템에서 실행 중인 애플리케이션과 프로세스의 리소스 사용을 관리하는 방법을 설명합니다. 절차에서는 CPU 소비가 보다 효율적이 되도록 이러한 애플리케이션의 CPU 사용을 규제하는 데 중점을 둡니다.

애플리케이션이 CPU 시간을 과도하게 사용하는 것을 방지하려면 `cgroups v2`를 사용하여 제한을 설정하여 애플리케이션의 리소스 사용이 더 잘 규제되도록 할 수 있습니다. 다음 섹션에서는 두 가지 방법을 사용하여 CPU 리소스 제어를 구현합니다.

- CPU 대역폭 설정

- CPU 가중치 설정

### cgroups v2 활성화

부팅 시 Enterprise Linux 9는 기본적으로 `cgroups v2`를 마운트합니다.

1. `cgroups v2`가 활성화되어 시스템에 마운트되어 있는지 확인하세요.

   ```
   sudo mount -l | grep cgroup
   ```

   ```nocopybutton
   /sys/fs/cgroup의 cgroup2 유형 cgroup2(rw,nosuid,nodev,noexec,relatime,seclabel,nsdelegate,memory_recursiveprot)
   ```

2. 추가적으로 루트 제어 그룹이라고도 하는 `/sys/fs/cgroup` 디렉터리의 내용을 확인합니다.

   ```
   ll /sys/fs/cgroup/
   ```

   `cgroups v2`의 경우 디렉터리에 있는 파일에는 파일 이름 앞에 접두사가 있어야 합니다. (예, `cgroup`.\*, `cpu`.\*, `memory`.\*, 등) [컨트롤 그룹 파일 시스템](ko-osmanage-ManagingResources.md#컨트롤-그룹-파일-시스템).

### CPU 시간 분배를 위한 제어 그룹 준비

1. 루트 제어 그룹의 `/sys/fs/cgroup/cgroup.controllers` 파일에서 `cpu` 및 `cpuset` 컨트롤러를 사용할 수 있는지 확인하세요.

   ```
   sudo cat /sys/fs/cgroup/cgroup.controllers
   ```

   ```nocopybutton
   **cpuset****cpu** io memory hugetlb pids rdma misc
   ```

2. `cgroup.subtree_control` 파일에 CPU 컨트롤러를 추가합니다.

   기본적으로 `memory` 및 `pids` 컨트롤러만 파일에 있습니다. CPU 컨트롤러를 추가하려면 다음을 입력하십시오.:

   ```
   echo "+cpu" | sudo tee /sys/fs/cgroup/cgroup.subtree_control
   echo "+cpuset" | sudo tee /sys/fs/cgroup/cgroup.subtree_control
   ```

3. 추가적으로 CPU 컨트롤러가 제대로 추가되었는지 확인합니다.

   ```
   sudo cat /sys/fs/cgroup/cgroup.subtree_control
   ```

   ```nocopybutton
   cpuset cpu memory pids
   ```

4. 루트 제어 그룹 아래에 하위 그룹을 생성하여 애플리케이션의 CPU 리소스를 관리하기 위한 새 제어 그룹이 됩니다.

   ```
   sudo mkdir /sys/fs/cgroup/MyGroup
   ```

5. 새 하위 디렉터리 또는 하위 그룹의 내용을 나열합니다.

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

   `/sys/fs/cgroup/cgroup.subtree_control`에 추가한 CPU 컨트롤러에 따라 루트 제어 그룹에서 상속된 `MyGroup`의 내용이 이제 더 제한됩니다. 따라서 `cpuset`.\*, `cpu`.\*, `memory`.\* 및 `pids`.\* 파일만 `MyGroup` 디렉터리에 있습니다.

6. `MyGroup`의 `cgroup.subtre_control` 파일에서 CPU 관련 컨트롤러를 활성화합니다.

   ```
   echo "+cpu" | sudo tee /sys/fs/cgroup/MyGroup/cgroup.subtree_control
   echo "+cpuset" | sudo tee /sys/fs/cgroup/MyGroup/cgroup.subtree_control
   ```

7. 선택적으로 `MyGroup` 아래의 하위 그룹에 대해 CPU 컨트롤러가 활성화되어 있는지 확인하세요.

   ```
   sudo cat /sys/fs/cgroup/MyGroup/cgroup.subtree_control
   ```

   ```nocopybutton
   cpuset cpu
   ```

### CPU 시간 분배를 조절하기 위한 CPU 대역폭 설정

이 절차는 다음 가정을 기반으로 합니다.:

- `top` 명령의 다음 샘플 출력에 표시된 것처럼 CPU 리소스를 과도하게 소비하는 애플리케이션은 `sha1sum`입니다.:

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

  샘플 출력에서 ​​`sha1sum` 프로세스의 PID는 34500 및 34501입니다.

- 현재 시스템에는 여러 개의 CPU가 있습니다.

  시스템의 CPU 수를 표시하려면 다음 명령을 사용할 수 있습니다.:

  ```
  sudo cat /sys/fs/cgroup/cpuset.cpus.effective
  ```

  ```nocopybutton
  0-1
  ```

  The sample output indicates a dual-core system.

**중요:**

다음 절차의 전제 조건으로, 다음에 설명된 대로 `cgroup-v2` 준비를 완료해야 합니다. 해당 준비를 건너뛴 경우 이 절차를 완료할 수 없습니다.

1. `MyGroup` 하위 디렉터리에 `tasks` 디렉터리를 만듭니다.

   ```
   sudo mkdir /sys/fs/cgroup/MyGroup/tasks
   ```

   이 디렉터리는 `cpu` 및 `cpuset` 컨트롤러에만 관련된 파일이 포함된 하위 그룹을 정의합니다.

2. 선택적으로 새 하위 디렉터리의 내용을 나열합니다.

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

3. `tasks` 디렉터리에서 동일한 CPU를 사용하도록 CPU 시간을 규제하려는 프로세스를 설정합니다.

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

5. `MyGroup/tasks` 하위 제어 그룹 내에서 제한을 설정하려면 CPU 대역폭을 구성하세요.

   ```
   echo "200000 1000000"n | sudo tee /sys/fs/cgroup/MyGroup/tasks/cpu.max
   ```

   명령에서 값 20000은 지정된 기간 동안 하위 그룹의 모든 프로세스가 집합적으로 실행되도록 허용되는 시간 할당량(마이크로초)을 나타냅니다. 해당 기간은 값 1000000으로 정의됩니다. 특히 `/sys/fs/cgroup/MyGroup/tasks` 그룹의 프로세스는 0.2초, 즉 매초의 1/5 동안만 CPU에서 실행될 수 있습니다.

   정의된 기간 내에 컨트롤 그룹에 의해 할당량이 소진되면 프로세스는 다음 기간까지 일시 중지됩니다.

6. 선택적으로 시간 할당량을 확인합니다.

   ```
   sudo cat /sys/fs/cgroup/MyGroup/tasks/cpu.max
   ```

   ```nocopybutton
   200000 1000000
   ```

7. 하위 그룹에 애플리케이션의 PID를 추가합니다.

   ```
   echo "34500" | sudo tee /sys/fs/cgroup/MyGroup/tasks/cgroup.procs
   echo "34501" | sudo tee /sys/fs/cgroup/MyGroup/tasks/cgroup.procs
   ```

8. 지정된 제어 그룹에서 애플리케이션이 실행되고 있는지 확인하십시오.

   ```
   sudo cat /proc/34500/cgroup /proc/34501/cgroup
   ```

   ```nocopybutton
   0::/MyGroup/tasks
   0::/MyGroup/tasks
   ```

9. CPU 대역폭을 설정한 후 현재 CPU 사용량을 확인하십시오.

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

   `MyGroup/tasks` 그룹은 CPU 사용량의 총 20%로 제한되므로 이제 각 `sha1sum` 프로세스는 CPU 시간의 10%로 제한됩니다.

### CPU 시간 분배를 조절하기 위한 CPU 가중치 설정

이 절차는 다음 가정을 기반으로 합니다.:

- `top` 명령의 다음 샘플 출력에 표시된 것처럼 CPU 리소스를 과도하게 소비하는 애플리케이션은 `sha1sum`입니다.:

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

  출력에서 `sha1sum` 프로세스의 PID는 33301, 33302, 33303입니다.

- 현재 시스템에는 여러 개의 CPU가 있습니다.

  시스템의 CPU 수를 표시하려면 다음을 입력하십시오.:

  ```
  sudo cat /sys/fs/cgroup/cpuset.cpus.effective
  ```

  ```nocopybutton
  0-1
  ```

  The sample output indicates a dual-core system.

**중요:**

다음 절차의 전제 조건으로, 다음에 설명된 대로 `cgroup-v2` 준비를 완료해야 합니다. 해당 준비를 건너뛴 경우 이 절차를 완료할 수 없습니다.

1. `MyGroup` 하위 디렉터리에 하위 그룹 3개를 만듭니다.

   ```
   sudo mkdir /sys/fs/cgroup/MyGroup/g1
   sudo mkdir /sys/fs/cgroup/MyGroup/g2
   sudo mkdir /sys/fs/cgroup/MyGroup/g3
   ```

2. 각 하위 그룹의 CPU 가중치를 구성합니다.

   ```
   echo "150" | sudo tee /sys/fs/cgroup/MyGroup/g1/cpu.weight
   echo "100" | sudo tee /sys/fs/cgroup/MyGroup/g2/cpu.weight
   echo "50" | sudo tee /sys/fs/cgroup/MyGroup/g3/cpu.weight
   ```

3. 해당 하위 그룹에 애플리케이션 PID를 적용합니다.

   ```
   echo "33301" | sudo tee /sys/fs/cgroup/Example/g1/cgroup.procs
   echo "33302" | sudo tee /sys/fs/cgroup/Example/g2/cgroup.procs
   echo "33303" | sudo /sys/fs/cgroup/Example/g3/cgroup.procs
   ```

   이 명령은 선택한 애플리케이션을 `MyGroup/g*/` 제어 그룹의 구성원으로 설정합니다. 각 `sha1sum` 프로세스의 CPU 시간은 각 그룹에 구성된 CPU 시간 분포에 따라 달라집니다.

   실행 중인 프로세스가 있는 `g1`, `g2`, `g3` 그룹의 가중치는 상위 제어 그룹인 `MyGroup` 수준에서 합산됩니다.

   이 구성을 사용하면 모든 프로세스가 동시에 실행될 때 커널은 다음과 같이 각 `sha1sum` 처리에 해당 `cgroup`의 `cpu.weight` 파일을 기반으로 비례적인 CPU 시간을 할당합니다.:

   | Child group | `cpu.weight` setting | Percent of CPU time allocation                        |
   | ----------- | -------------------- | ----------------------------------------------------- |
   | g1          | 150                  | ~50% \(150/300\) |
   | g2          | 100                  | ~33% \(100/300\) |
   | g3          | 50                   | ~16% \(50/300\)  |

   하나의 하위 그룹에 실행 중인 프로세스가 없으면 실행 중인 프로세스에 대한 CPU 시간 할당은 실행 중인 프로세스가 있는 나머지 하위 그룹의 총 가중치를 기준으로 다시 계산됩니다. 예를 들어 `g2` 하위 그룹에 실행 중인 프로세스가 없으면 총 가중치는 `g1 g3`의 가중치인 200이 됩니다. 이 경우 `g1`의 CPU 시간은 150/200\(~75%\)가 되고 `g3`의 경우 50/200\(~25%\)가 됩니다.

4. 지정된 제어 그룹에서 애플리케이션이 실행되고 있는지 확인하십시오.

   ```
   sudo cat /proc/33301/cgroup /proc/33302/cgroup /proc/33303/cgroup
   ```

   ```nocopybutton
   0::/MyGroup/g1
   0::/MyGroup/g2
   0::/MyGroup/g3
   ```

5. CPU 가중치를 설정한 후 현재 CPU 사용량을 확인하십시오.

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

## cgroups v2를 사용하여 사용자 리소스 관리

이전 샘플 절차에서는 애플리케이션의 시스템 리소스 사용을 관리하는 방법을 설명합니다. 시스템에 로그인하는 사용자에게 리소스 필터를 직접 구현하여 리소스 사용을 관리할 수도 있습니다.
