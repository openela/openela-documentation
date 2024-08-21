<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# 커널 모듈 관리

이 장에서는 커널 모듈의 동작을 로드, 언로드 및 수정하는 방법을 설명합니다.

## 커널 모듈

부트로더는 커널을 메모리에 로드합니다. 커널 소스 트리에 소스 파일을 포함시키고 커널을 다시 컴파일하여 커널에 새 코드를 추가할 수 있습니다. 커널 모듈은 커널이 새로운 하드웨어에 액세스하고, 다양한 파일 시스템 유형을 지원하고, 다른 방식으로 기능을 확장할 수 있도록 하는 장치 드라이버를 제공합니다. 모듈은 요청 시 동적으로 로드 및 언로드될 수 있습니다. 사용하지 않는 장치 드라이버로 인한 메모리 낭비를 방지하기 위해 Enterprise Linux는 로드 가능한 커널 모듈\(LKM\)을 지원합니다.

부팅 시 악성 코드가 실행되지 않도록 시스템을 보호하기 위해 커널 모듈에 서명할 수 있습니다. UEFI 보안 부팅이 활성화되면 올바른 서명 정보가 포함된 커널 모듈만 로드할 수 있습니다.

## 로드된 모듈에 대한 정보 나열

`lsmod` 명령은 커널에 로드된 모듈을 나열합니다.:

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

출력에는 모듈 이름, 사용하는 메모리 양, 모듈을 사용하는 프로세스 수 및 종속된 다른 모듈의 이름이 표시됩니다. 예를 들어 `dm_log` 모듈은 `dm_region_hash` 및 `dm_mirror` 모듈에 따라 달라집니다. 또한 이 예에서는 두 프로세스가 세 모듈을 모두 사용하고 있음을 보여줍니다.

modinfo 명령을 사용하여 모듈에 대한 자세한 정보를 표시합니다.:

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

출력에는 다음 정보가 포함됩니다.:

- **`filename`**

  커널 개체 파일의 절대 경로입니다.

- **`version`**

  모듈의 버전 번호입니다. 버전 번호는 패치된 모듈에 대해 업데이트되지 않을 수 있으며 최신 커널에서는 누락되거나 제거될 수 있습니다.

- **`license`**

  모듈에 대한 라이센스 정보입니다.

- **`description`**

  모듈에 대한 간략한 설명입니다.

- **`author`**

  Author credit for the module.

- **`srcversion`**

  모듈을 생성하는 데 사용된 소스 코드의 해시입니다.

- **`alias`**

  모듈의 내부 별칭 이름입니다.

- **`depends`**

  이 모듈이 의존하는 모든 모듈의 쉼표로 구분된 목록입니다.

- **`retpoline`**

  Spectre 보안 취약성에 대한 완화를 포함하는 모듈이 구축되었음을 나타내는 플래그입니다.

- **`intree`**

  모듈이 커널 내 트리 소스에서 빌드되었으며 오염되지 않았음을 나타내는 플래그입니다.

- **`vermagic`**

  모듈을 컴파일하는 데 사용된 커널 버전으로, 모듈이 로드될 때 현재 커널과 비교하여 확인됩니다.

- **`sig_id`**

  보안 부팅용 모듈(일반적으로 PKCS)에 서명하는 데 사용되었을 수 있는 서명 키를 저장하는 데 사용되는 방법\#7

- **`signer`**

  보안 부팅용 모듈에 서명하는 데 사용되는 서명 키의 이름입니다.

- **`sig_key`**

  모듈 서명에 사용되는 키의 서명 키 식별자입니다.

- **`sig_hashalgo`**

  서명된 모듈에 대한 서명 해시를 생성하는 데 사용되는 알고리즘입니다.

- **`signature`**

  서명된 모듈의 서명 데이터입니다.

- **`parm`**

  모듈 매개변수 및 설명.

모듈은 커널 개체 파일 \(`/lib/modules/*kernel\_version*/kernel/*ko*`\)에서 커널로 로드됩니다. 커널 객체 파일의 절대 경로를 표시하려면 `-n` 옵션을 지정하십시오.

```
modinfo -n parport
```

```nocopybutton
/lib/modules/5.4.17-2136.306.1.3.el8
```

자세한 내용은 `lsmod(5)` 및 `modinfo(8)` 매뉴얼 페이지를 참조하세요.

## 모듈 로드 및 언로드

`modprobe` 명령은 커널 모듈을 로드합니다.

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

종속성을 해결하기 위해 추가 모듈이 로드되는지 여부를 표시하려면 `-v` \(verbose\) 옵션을 포함합니다.

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

`modprobe` 명령은 이미 로드된 모듈을 다시 로드하지 않습니다. 모듈을 다시 로드하려면 먼저 모듈을 언로드해야 합니다.

커널 모듈을 언로드하려면 `-r` 옵션을 사용하십시오.

```
sudo modprobe -rv nfs
```

```nocopybutton
rmmod /lib/modules/4.18.0-80.el8.x86_64/kernel/fs/nfs/nfs.ko
rmmod /lib/modules/4.18.0-80.el8.x86_64/kernel/fs/lockd/lockd.ko
rmmod /lib/modules/4.18.0-80.el8.x86_64/kernel/fs/fscache/fscache.ko
...
```

모듈은 처음 로드된 순서와 반대로 언로드됩니다. 프로세스나 로드된 다른 모듈에 필요한 경우 모듈은 언로드되지 않습니다.

자세한 내용은 `modprobe(8)` 및 `modules.dep(5)` 매뉴얼 페이지를 참조하세요.

## 모듈 매개변수

모듈의 동작을 수정하려면 `modprobe` 명령에서 모듈에 대한 매개변수를 지정하세요.

```
sudo modprobe *module\_name* *parameter*=*value* ...
```

여러 매개변수와 값 쌍을 공백으로 구분하세요. 배열 값은 쉼표로 구분된 목록으로 표시됩니다.

```
sudo modprobe foo arrayparm=1,2,3,4
```

또는 예를 들어 `/sys/module/*module\_name*/parameters` 아래의 파일에 새 값을 기록하여 로드된 모듈 및 내장 드라이버에 대한 일부 매개변수의 값을 변경합니다.:

```
echo 0 | sudo tee /sys/module/ahci/parameters/skip_host_reset
```

구성 파일 \(`/etc/modprobe.d/*.conf`\)은 모듈 옵션을 지정하고, 모듈 별칭을 만들고, 특별한 요구 사항이 있는 모듈에 대해 `modprobe`의 일반적인 동작을 재정의합니다. 이전 버전의 `modprobe`에서 사용된 `/etc/modprobe.conf` 파일도 있으면 유효합니다. `/etc/modprobe.conf` 및 `/etc/modprobe.d/*.conf` 파일의 항목은 동일한 구문을 사용합니다.

다음은 `modprobe` 구성 파일에서 일반적으로 사용되는 명령입니다.:

- **`alias`**

  모듈의 대체 이름을 만듭니다. 별칭에는 셸 와일드카드가 포함될 수 있습니다. `sd-mod` 모듈의 별칭을 만들려면:

  ```
  alias block-major-8-* sd_mod
  ```

- **`blacklist`**

  `modinfo` 명령으로 표시되는 모듈의 내부 별칭을 무시합니다. 이 명령은 일반적으로 다음 조건에서 사용됩니다.:

  - 관련 하드웨어가 필요하지 않는 경우

  - 두 개 이상의 모듈이 모두 동일한 장치를 지원하는 경우

  - 모듈이 장치를 지원한다고 잘못 요청하는 경우

  예를 들어 프레임 버퍼 드라이버 `cirrusfb`의 별칭을 내리려면 다음을 입력합니다.:

  ```
  blacklist cirrusfb
  ```

  `/etc/modprobe.d/blacklist.conf` 파일은 핫플러그 스크립트가 모듈을 로드하는 것을 방지하여 어떤 드라이버가 먼저 검색되는지에 관계없이 다른 드라이버가 모듈을 바인딩하도록 합니다. 아직 존재하지 않는 경우 새로 만들어야 합니다.

- **`install`**

  커널에 모듈을 로드하는 대신 쉘 명령을 실행합니다. 예를 들어 `snd-emu10k1` 대신 `snd-emu10k1-synth` 모듈을 로드하세요.:

  ```
  install snd-emu10k1 /sbin/modprobe --ignore-install snd-emu10k1 && /sbin/modprobe snd-emu10k1-synth
  ```

- **`options`**

  모듈에 대한 옵션을 정의합니다. 예를 들어, `b43` 모듈에 대한 `nohwcrypt` 및 `qos` 옵션을 정의하려면 다음을 입력하세요.:

  ```
  options b43 nohwcrypt=1 qos=0
  ```

- **`remove`**

  모듈을 언로드하는 대신 쉘 명령을 실행합니다. `nfsd` 모듈을 언로드하기 전에 `/proc/fs/nfsd`를 마운트 해제하려면 다음을 입력하십시오.:

  ```
  remove nfsd { /bin/umount /proc/fs/nfsd > /dev/null 2>&1 || :; } ;
  /sbin/modprobe -r --first-time --ignore-remove nfsd
  ```

  자세한 내용은 `modprobe.conf(5)` 매뉴얼 페이지를 참조하세요.

## 부팅 시 로드할 모듈 지정

시스템은 부팅 시 대부분의 모듈을 자동으로 로드합니다. 또한 `/etc/modules-load.d` 디렉토리에 모듈에 대한 구성 파일을 생성하여 로드할 모듈을 추가할 수도 있습니다. 파일 이름의 확장자는 `.conf`여야 합니다.

예를 들어 부팅 시 `bnxt_en.conf`를 강제로 로드하려면 다음 명령을 실행하세요.:

```
echo *bnxt\_en* | sudo tee /etc/modules-load.d/*bnxt\_en*.conf
```

`/etc/modules-load.d` 디렉토리에 대한 변경 사항은 재부팅 후에도 유지됩니다.

## 부팅 시 모듈이 로드되지 않도록 방지

`/etc/modprobe.d` 디렉터리의 구성 파일에 거부 규칙을 추가한 다음 부팅 시 커널을 로드하는 데 사용되는 초기 램디스크를 다시 빌드하면 부팅 시 모듈이 로드되지 않도록 할 수 있습니다.

1. 모듈이 로드되지 않도록 구성 파일을 만듭니다. 예를 들어:

   ```
   sudo tee /etc/modprobe.d/*bnxt\_en*-deny.conf <<'EOF'
   #DENY *bnxt\_en*
   blacklist *bnxt\_en*
   install *bnxt\_en* /bin/false
   EOF
   ```

2. 초기 램디스크 이미지 재구축:

   ```
   sudo dracut -f -v
   ```

3. 변경 사항을 적용하려면 시스템을 재부팅하세요.:

   ```
   sudo reboot
   ```

**주의:**

모듈을 비활성화하면 의도하지 않은 결과가 발생할 수 있으며 시스템이 제대로 부팅되지 않거나 부팅 후 완전히 작동하지 않을 수 있습니다. 가장 좋은 방법은 변경하기 전에 백업 램디스크 이미지를 생성하고 구성이 올바른지 확인하는 것입니다.

## Weak Update 모듈

드라이버 업데이트 디스크를 사용하여 설치되거나 독립 패키지에서 설치되는 드라이버와 같은 외부 모듈은 일반적으로 `/lib/modules/*kernel-version*/extra` 디렉터리에 설치됩니다. 이 디렉토리에 저장된 모듈은 이러한 모듈이 로드될 때 커널에 포함된 일치하는 모듈보다 선호됩니다. 설치된 외부 드라이버 및 모듈은 기존 커널 모듈을 재정의하여 하드웨어 문제를 해결할 수 있습니다. 각 커널 업데이트에 대해 이러한 외부 모듈을 호환되는 각 커널에서 사용할 수 있도록 해야 영향을 받는 하드웨어와의 드라이버 비호환성으로 인한 잠재적인 부팅 문제를 방지할 수 있습니다.

각 호환 가능한 커널 업데이트와 함께 외부 모듈을 로드해야 하는 요구 사항은 시스템에 매우 중요하므로 외부 모듈을 호환 가능한 커널에 대한 Weak Update 모듈로 로드하는 메커니즘이 존재합니다.

`/lib/modules/*kernel-version*/weak-updates` 디렉토리에 있는 호환 모듈에 대한 심볼릭 링크를 생성하여 Weak Update 모듈을 사용할 수 있게 만듭니다. 패키지 관리자는 호환되는 커널에 대해 `/lib/modules/*kernel-version*/extra` 디렉터리에 설치된 드라이버 모듈을 감지하면 이 프로세스를 자동으로 처리합니다.

예를 들어, 최신 커널이 이전 커널용으로 설치된 모듈과 호환되는 경우 외부 모듈\(예: `kmod-kvdo`\)은 다음과 같이 `weak-updates` 디렉토리에 심볼릭 링크로 자동 추가됩니다.

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

심볼릭 링크를 사용하면 커널 업데이트를 위해 외부 모듈을 로드할 수 있습니다.

Weak Update는 유익하며 커널 업데이트를 통해 외부 모듈을 수행하는 데 추가 작업이 필요하지 않도록 보장합니다. 커널 업그레이드 후 잠재적인 드라이버 관련 부팅 문제가 방지되므로 시스템과 하드웨어의 실행을 보다 예측 가능하게 제공됩니다.

어떤 경우에는 최신 커널 대신 Weak Update 모듈을 제거할 수도 있습니다. 이 경우 드라이버 업데이트의 일부로 설치한 외부 모듈보다는 새 드라이버를 사용하는 것이 더 나을 수 있습니다.

Weak Update 모듈을 제거하려면 다음 명령을 사용하십시오.:

```
rm -rf /lib/modules/*4.18.0-80.el8.x86\_64*/weak-updates/*kmod-kvdo/*
```

이 명령을 실행하면 각 커널에 대한 심볼릭 링크가 수동으로 제거됩니다.

또는 `weak-modules` 명령을 사용하여 호환 커널에 대해 지정된 Weak Update 모듈을 안전하게 제거하거나 이 명령을 사용하여 현재 커널에 대한 Weak Update 모듈을 제거할 수 있습니다. Weak Update 모듈을 추가하려면 `weak-modules` 명령을 유사하게 사용할 수 있습니다.

또한 `dry-run` 옵션과 함께 `weak-modules` 명령을 사용하여 실제 변경 없이 결과를 테스트할 수도 있습니다.

```
weak-modules --remove-kernel --dry-run --verbose
rm -rf kmod-kvdo
```
