<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# 커널 및 시스템 부팅 관리

이 장에서는 Enterprise Linux 부팅 프로세스와 GRand Unified Bootloader\(GRUB\) 버전 2 및 부팅 관련 커널 매개 변수를 구성하고 사용하는 방법에 대해 설명합니다.

## 부팅 프로세스

Enterprise Linux 부팅 프로세스를 이해하면 시스템 부팅 시 문제를 해결하는 데 도움이 될 수 있습니다. 부팅 프로세스에는 여러 파일이 포함되며 이러한 파일의 오류는 부팅 문제의 일반적인 원인입니다. 부팅 프로세스 및 구성은 하드웨어가 시스템 부팅을 처리하기 위해 UEFI 펌웨어를 사용하는지 아니면 레거시 BIOS를 사용하는지에 따라 다릅니다.

### UEFI 기반 부팅

Enterprise Linux 릴리스를 실행하는 UEFI 기반 시스템에서 시스템 부팅 프로세스는 다음 순서를 사용합니다.:

1. 시스템의 UEFI 펌웨어는 전원 공급 시 자체 테스트\(POST\)를 수행한 다음 주변 장치와 하드 디스크를 감지하고 초기화합니다.

2. UEFI는 EFI 시스템 파티션\(ESP\)으로 식별하는 특정 전역 고유 식별자\(GUID\)를 사용하여 GPT 파티션을 검색합니다. 이 파티션에는 부트로더와 같은 EFI 응용 프로그램이 포함되어 있습니다. 여러 부팅 장치가 있는 경우 UEFI 부팅 관리자는 부팅 관리자에 정의된 순서에 따라 적절한 ESP를 사용합니다. 기본 정의를 사용하지 않으려면 `efibootmgr` 도구를 사용하여 다른 순서를 정의할 수 있습니다.

3. UEFI 부팅 관리자는 보안 부팅이 활성화되어 있는지 확인합니다. 보안 부팅이 비활성화된 경우 부팅 관리자는 ESP에서 GRUB 2 부트로더를 실행합니다.

   그렇지 않으면 부팅 관리자가 부트 로더에서 인증서를 요청하고 UEFI 보안 부팅 키 데이터베이스에 저장된 키와 비교하여 이를 검증합니다. 인증서 유효성 검사 프로세스를 처리하기 위해 2단계 부팅 프로세스를 수행하도록 환경이 구성되고 GRUB 2 부트로더를 로드하기 전에 인증을 담당하는 `shim.efi` 애플리케이션이 먼저 로드됩니다. 인증서가 유효하면 부트 로더가 실행되고 로드하도록 구성된 커널의 유효성을 검사합니다.

4. 부트 로더는 `vmlinuz` 커널 이미지 파일을 메모리에 로드하고 `initramfs` 이미지 파일의 내용을 임시 메모리 기반 파일 시스템\(`tmpfs`\)으로 추출합니다.

5. 커널은 루트 파일 시스템에 액세스하는 데 필요한 `initramfs` 파일 시스템에서 드라이버 모듈을 로드합니다.

6. 커널은 프로세스 ID 1\(PID 1\)로 `systemd` 프로세스를 시작합니다. 참조: [systemd 서비스 관리자](ko-osmanage-WorkingWithSystemServices.md#systemd-서비스-관리자).

7. `systemd`는 정의된 추가 프로세스를 실행합니다.

   **Note:**

   자신만의 `systemd` 유닛을 정의하여 부팅 프로세스 중에 처리할 다른 작업을 지정하세요. 이 방법은 `/etc/rc.local` 파일을 사용하는 것보다 선호되는 접근 방식입니다.

### BIOS 기반 부팅

Enterprise Linux 릴리스를 실행하는 BIOS 기반 시스템에서 부팅 프로세스는 다음과 같습니다.

1. 시스템의 BIOS는 전원 공급 시 자체 테스트\(POST\)를 수행한 다음 모든 주변 장치와 하드 디스크를 감지하고 초기화합니다.

2. BIOS는 부팅 장치의 마스터 부트 레코드\(MBR\)를 메모리로 읽습니다. MBR은 해당 장치의 파티션 구성, 파티션 테이블 및 오류 감지에 사용되는 부팅 서명에 대한 정보를 저장합니다. MBR에는 부트 로더 프로그램\(GRUB 2\)에 대한 포인터도 포함되어 있습니다. 부팅 프로그램 자체는 동일한 장치에 있을 수도 있고 다른 장치에 있을 수도 있습니다.

3. 부트 로더는 `vmlinuz` 커널 이미지 파일을 메모리에 로드하고 `initramfs` 이미지 파일의 내용을 임시 메모리 기반 파일 시스템\(`tmpfs`\)으로 추출합니다.

4. 커널은 루트 파일 시스템에 액세스하는 데 필요한 `initramfs` 파일 시스템에서 드라이버 모듈을 로드합니다.

5. 커널은 프로세스 ID 1\(PID 1\)을 사용하여 `systemd` 프로세스를 시작합니다. [systemd 서비스 관리자](ko-osmanage-WorkingWithSystemServices.md#systemd-서비스-관리자)를 참조하세요.

6. [System-State Targets](ko-osmanage-WorkingWithSystemServices.md#system-state-targets).

   **Note:**

   사용자 `systemd` 단위를 정의하여 부팅 프로세스 중에 처리할 다른 작업을 지정합니다. 이 방법은 `/etc/rc.local` 파일을 사용하는 것보다 선호되는 접근 방식입니다.

## GRUB 2 부트로더

Enterprise Linux 외에도 GRUB 2는 많은 독점 운영 체제를 로드하고 체인 로드할 수 있습니다. GRUB 2는 파일 시스템과 커널 실행 파일의 형식을 이해합니다. 따라서 부팅 장치에서 커널의 정확한 위치를 알 필요 없이 임의의 OS를 로드할 수 있습니다. GRUB 2에서는 커널을 로드하기 위해 파일 이름과 드라이브 파티션만 필요합니다. GRUB 2 메뉴를 사용하거나 커맨드라인에 입력하여 이 정보를 구성할 수 있습니다.

GRUB 2 동작은 구성 파일을 기반으로 합니다. BIOS 기반 시스템에서 구성 파일은 `/boot/grub2/grub.cfg`입니다. UEFI 기반 시스템에서 구성 파일은 `/boot/efi/EFI/redhat/grub.cfg`입니다. 각 커널 버전의 부팅 매개변수는 `/boot/loader/entries`의 독립적인 구성 파일에 저장됩니다. 각 커널 구성은 `*machine\_id*-*kernel\_version*.el8.*arch*.conf` 파일 이름으로 저장됩니다.

**Note:**

GRUB 2 구성 파일을 직접 편집하지 마십시오.

`grub2-mkconfig` 명령은 `/etc/grub.d`의 템플릿 스크립트와 구성 파일 `/etc/default/grub`에서 가져온 메뉴 구성 설정을 사용하여 구성 파일을 생성합니다.

기본 메뉴 항목은 `/etc/default/grub`의 `GRUB_DEFAULT` 매개변수 값으로 설정됩니다. `GRUB_DEFAULT`가 `saved`로 설정된 경우 `grub2-set-default` 및 `grub2-reboot` 명령을 사용하여 기본 항목을 지정할 수 있습니다. `grub2-set-default` 명령은 모든 후속 재부팅에 대한 기본 항목을 설정하는 반면, `grub2-reboot`는 다음 재부팅에 대한 기본 항목만 설정합니다.

숫자 값을 `GRUB_DEFAULT` 값이나 `grub2-reboot` 또는 `grub2-set-default`에 대한 인수로 지정하면 GRUB 2는 첫 번째 항목에 대해 0부터 시작하는 구성 파일의 메뉴 항목 수를 계산합니다.

GRUB 2 사용, 구성, 사용자 정의에 대한 자세한 내용은 \`로도 설치되는 [GNU GRUB 매뉴얼](https://www.gnu.org/software/grub/manual/grub/grub.html)을 참조하세요.

## Linux 커널

Linux Foundation은 오픈 소스 개발자가 다양한 개방형 기술 프로젝트를 코딩, 관리 및 확장할 수 있는 허브를 제공합니다. 또한 Enterprise Linux에서 사용되는 것을 포함하여 모든 Linux 배포판의 핵심인 Linux 커널의 다양한 버전을 배포하기 위해 존재하는 Linux 커널 조직을 관리합니다. Linux 커널은 Enterprise Linux에서 실행되는 컴퓨터 하드웨어와 사용자 공간 애플리케이션 간의 상호 작용을 관리합니다.

OpenELA에는 해당 Red Hat Enterprise Linux\(RHEL\) 릴리스에 배포된 Linux 커널과 완벽하게 호환되는 **Red Hat 호환 커널** \(RHCK\)이 포함되어 있습니다. RHCK를 사용하면 Red Hat Enterprise Linux에서 실행되는 애플리케이션과의 완전한 호환성을 보장할 수 있습니다.

**중요:** Linux 커널은 Enterprise Linux 사용자 공간에서 애플리케이션을 실행하는 데 중요합니다. 따라서 OpenELA에서 제공하는 최신 버그 수정, 개선 사항 및 보안 업데이트를 통해 커널을 최신 상태로 유지해야 합니다.

## `grubby`를 사용하여 GRUB 2에서 커널 관리

`grubby` 명령을 사용하여 커널을 보고 관리할 수 있습니다.

다음 명령을 사용하여 시스템에 설치 및 구성된 커널을 표시하십시오.:

```
sudo grubby --info=ALL
```

특정 커널을 기본 부팅 커널로 구성하려면 다음을 실행하세요.:

```
sudo grubby --set-default /boot/vmlinuz-4.18.0-80.el8.x86_64
```

또한 `grubby` 명령을 사용하여 커널 구성 항목을 업데이트하여 커널 부팅 인수를 추가하거나 제거할 수도 있습니다.

```
sudo grubby --remove-args="rhgb quiet" --args=rd_LUKS_UUID=luks-39fec799-6a6c-4ac1-ac7c-1d68f2e6b1a4 \
--update-kernel /boot/vmlinuz-4.18.0-80.el8.x86_64
```

`grubby` 명령에 대한 자세한 내용은 `grubby(8)` 매뉴얼 페이지를 참조하세요.

## 커널 부팅 매개변수

다음 표에서는 일반적으로 사용되는 몇 가지 커널 부팅 매개변수에 대해 설명합니다.

<table><thead><tr><th>

옵션

</th><th>

설명

</th></tr></thead><tbody><tr><td>

`0`, `1`, `2`, `3`, `4`, `5`, or `6`, or `systemd.unit=runlevel*N*.target`

</td><td>

레거시 SysV 실행 수준과 일치하도록 가장 가까운 `systemd`와 동등한 시스템 상태 대상을 지정합니다. _N_은 0에서 6 사이의 정수 값을 사용할 수 있습니다.

Systemd는 레거시 SysV init 시스템을 모방하기 위해 시스템 상태 대상을 매핑합니다. 시스템 상태 대상에 대한 설명은 다음을 참조하세요.

</td></tr><tr><td>

`1`, `s`, `S`, `single`, or `systemd.unit=rescue.target`

</td><td>

복구 셸을 지정합니다. 시스템이 단일 사용자 모드로 부팅되면 `root` 비밀번호를 묻는 메시지가 나타납니다.

</td></tr><tr><td>

`3` or `systemd.unit=multi-user.target`

</td><td>

다중 사용자, 비그래픽 로그인을 위한 `systemd` 대상을 지정합니다.

</td></tr><tr><td>

`5` or `systemd.unit=graphical.target`

</td><td>

다중 사용자, 그래픽 로그인을 위한 `systemd` 대상을 지정합니다.

</td></tr><tr><td>

`-b`, `emergency`, or `systemd.unit=emergency.target`

</td><td>

비상 모드를 지정합니다. 시스템이 단일 사용자 모드로 부팅되고 `root` 비밀번호를 묻는 메시지가 표시됩니다. 복구 모드에 있을 때보다 시작되는 서비스 수가 적습니다.

</td></tr><tr><td>

`KEYBOARDTYPE=*kbtype*`

</td><td>

`initramfs`의 `/etc/sysconfig/keyboard`에 기록되는 키보드 유형을 지정합니다.

</td></tr><tr><td>

`KEYTABLE=*kbtype*`

</td><td>

`initramfs`의 `/etc/sysconfig/keyboard`에 기록되는 키보드 레이아웃을 지정합니다.

</td></tr><tr><td>

`LANG=*language*_*territory*.*codeset*`

</td><td>

`initramfs`의 `/etc/sysconfig/i18n`에 기록되는 시스템 언어 및 코드 세트를 지정합니다.

</td></tr><tr><td>

`max_loop=*N*`

</td><td>

블록 장치로 파일에 액세스하는 데 사용할 수 있는 루프 장치 수\(`/dev/loop*`\)를 지정합니다. _N_의 기본값과 최대값은 8과 255입니다.

</td></tr><tr><td>

`nouptrack`

</td><td>

Ksplice Uptrack 업데이트가 커널에 적용되지 않도록 합니다.

</td></tr><tr><td>

`quiet`

</td><td>

디버깅 출력을 줄입니다.

</td></tr><tr><td>

`rd_LUKS_UUID=*UUID*`

</td><td>

지정된 UUID를 사용하여 암호화된 Linux 통합 키 설정\(LUKS\) 파티션을 활성화합니다.

</td></tr><tr><td>

`rd_LVM_VG=*vg*/lv_*vol*`

</td><td>

활성화할 LVM 볼륨 그룹과 볼륨을 지정합니다.

</td></tr><tr><td>

`rd_NO_LUKS`

</td><td>

암호화된 LUKS 파티션 감지를 비활성화합니다.

</td></tr><tr><td>

`rhgb`

</td><td>

Red Hat 그래픽 부팅 디스플레이를 사용하여 부팅 진행 상황을 표시하도록 지정합니다.

</td></tr><tr><td>

`rn_NO_DM`

</td><td>

Device-Mapper\(DM\) RAID 감지를 비활성화합니다.

</td></tr><tr><td>

`rn_NO_MD`

</td><td>

다중 장치\(MD\) RAID 감지를 비활성화합니다.

</td></tr><tr><td>

`ro root=/dev/mapper/*vg*-lv_root`

</td><td>

루트 파일 시스템이 읽기 전용으로 마운트되도록 지정하고 LVM 볼륨\(여기서 _vg_는 볼륨 그룹의 이름\)의 장치 경로로 루트 파일 시스템을 지정합니다.

</td></tr><tr><td>

`rw root=UUID=*UUID*`

</td><td>

부팅 시 루트 \(`/`\) 파일 시스템이 읽기/쓰기 가능하게 마운트되도록 지정하고 해당 UUID로 루트 파티션을 지정합니다.

</td></tr><tr><td>

`selinux=0`

</td><td>

SELinux를 비활성화합니다.

</td></tr><tr><td>

`SYSFONT=*font*`

</td><td>

`initramfs`의 `/etc/sysconfig/i18n`에 기록되는 콘솔 글꼴을 지정합니다.

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

자세한 내용은 `kernel-command-line(7)` 매뉴얼 페이지를 참조하세요.

## 부팅하기 전에 커널 부팅 매개변수 수정

커널을 부팅하기 전에 부팅 매개변수를 수정하려면 다음 단계를 따르세요.:

1. 부팅 프로세스 시작 시 GRUB 부팅 메뉴가 나타나면 화살표 키를 사용하여 필요한 커널을 강조 표시하고 스페이스바를 누릅니다.

2. E를 눌러 커널의 부팅 구성을 편집합니다.

3. 화살표 키를 사용하여 커널의 부팅 구성 라인인 `linux`로 시작하는 라인의 끝으로 커서를 이동합니다.

4. 부팅 매개변수를 수정합니다.

   시스템이 복구 셸로 부팅하도록 지시하는 `systemd.target=runlevel1.target`과 같은 매개변수를 추가할 수 있습니다.

5. Ctrl X를 눌러 시스템을 부팅합니다.

## GRUB 2 기본 커널 부팅 매개변수 수정

재부팅할 때마다 이러한 매개변수가 기본적으로 적용되도록 GRUB 2 구성에 대한 부팅 매개변수를 수정하려면 다음 단계를 수행하십시오.:

1. 예를 들어 `/etc/default/grub`을 편집하고 `GRUB_CMDLINE_LINUX` 정의에 매개변수 설정을 추가합니다.:

   ```
   GRUB_CMDLINE_LINUX="vconsole.font=latarcyrheb-sun16 vconsole.keymap=uk 
   crashkernel=1G-4G:192M,4G-64G:256M,64G-:512M  rd.lvm.lv=ol/swap rd.lvm.lv=ol/root biosdevname=0 
   rhgb quiet **systemd.unit=runlevel3.target**"
   ```

   이 예에서는 시스템이 기본적으로 다중 사용자, 비그래픽 모드로 부팅되도록 `systemd.unit=runlevel3.target` 매개변수를 추가합니다.

2. `/boot/grub2/grub.cfg`를 다시 빌드하세요:

   ```
   sudo grub2-mkconfig -o /boot/grub2/grub.cfg
   ```

   변경 사항은 구성된 모든 커널이 다음에 시스템을 재부팅할 때 적용됩니다.

**Note:**

UEFI로 부팅하는 시스템의 경우 부팅 구성이 전용 FAT32 형식 파티션에 저장되기 때문에 `grub.cfg` 파일은 `/boot/efi/EFI/redhat` 디렉터리에 있습니다.

시스템이 성공적으로 부팅되면 해당 파티션의 `EFI` 폴더가 Enterprise Linux용 루트 파일 시스템의 `/boot/efi` 디렉터리 내에 마운트됩니다.
