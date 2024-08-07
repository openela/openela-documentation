<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# 사용자에게 sudo 액세스 권한 부여

Enterprise Linux에서는 관리자만 시스템에서 권한 있는 작업을 수행할 수 있습니다.

사용자에게 추가 권한을 부여하기 위해 관리자는 `visudo` 명령을 사용하여 `/etc/sudoers.d` 디렉터리에 새 구성 파일을 생성하거나 `/etc/sudoers` 파일을 수정할 수 있습니다.

관리자가 `/etc/sudoers.d` 디렉토리의 구성 파일을 사용하여 할당한 권한은 시스템 업그레이드 간에 유지되며 유효하지 않은 경우 `sudo` 명령에 의해 자동으로 건너뜁니다. 관리자는 각 구성 파일에 대한 파일 소유권과 권한을 변경할 수도 있습니다. 자세한 내용은, 참조: [sudoers.d 디렉토리에 사용자 권한 추가](ko-userauth-GrantingsudoAccesstoUsers.md#sudoersd-디렉토리에-사용자-권한-추가).

또는 관리자가 `visudo` 명령을 사용하여 `/etc/sudoers` 파일에서 직접 권한을 할당할 수 있습니다. [sudoers 파일에 사용자 권한 추가](ko-userauth-GrantingsudoAccesstoUsers.md#sudoers-파일에-사용자-권한-추가).

## Enterprise Linux의 관리 액세스

기본적으로 모든 사용자는 `su` 명령을 실행하고 `root` 사용자 비밀번호를 제공하여 `root` 셸로 승격할 수 있습니다.:

```
su
```

```nocopybutton
Password:
```

모든 사용자는 동일한 셸에서 단일 관리 작업을 수행할 수도 있지만 해당 사용자가 `root` 사용자 비밀번호를 제공할 때까지는 해당 명령을 실행할 수 없습니다.:

```
su -c "whoami"
```

```nocopybutton
Password:
```

```nocopybutton
root
```

`su` 명령을 사용하여 `root` 셸로 승격하면 한 사람만 시스템을 관리하고 `root` 사용자 비밀번호를 알아야 하기 때문에 단일 사용자 환경과 워크스테이션에서 작동할 수 있습니다. 그러나 이 접근 방식은 다양한 수준의 액세스가 필요한 여러 사용자 및 관리자가 있는 공유 시스템에는 적합하지 않습니다.

`root` 사용자 비밀번호를 다른 사람과 공유하거나 원격 사용자가 `root` 사용자로 로그인하도록 허용하지 마십시오.

`sudo` 명령은 모든 사용자가 `root` 셸로 승격할 때 자신의 자격 증명을 제공할 수 있기 때문에 공유 시스템에 더 적합합니다.:

```
sudo -s
```

사용자는 `su` 명령으로 직접 권한을 높이고 `root` 사용자 비밀번호를 제공한 경우와 동일한 방식으로 `root` 셸을 종료할 수 있습니다.:

```
exit
```

또한 사용자는 `sudo` 명령을 실행하여 상승된 권한으로 단일 관리 작업을 수행할 수 있습니다.:

```
sudo whoami
```

```nocopybutton
root
```

자세한 내용은 `su(1)`, `sudo(8)` 및 `sudoers(5)` 매뉴얼 페이지를 참조하세요.

**Note:**

Enterprise Linux 설치 프로세스 중에 `root` 사용자를 선택적으로 비활성화하고 첫 번째 사용자에게 `sudo` 관리자 권한을 부여할 수 있습니다.

## sudo 명령 사용

사용자에게 `sudo` 액세스 권한이 부여된 경우 해당 사용자는 높은 권한으로 관리 명령을 실행할 수 있습니다.:

```
sudo *command*
```

`sudoer` 구성에 따라 사용자에게 비밀번호를 묻는 메시지가 표시될 수도 있습니다.

어떤 상황에서는 사용자가 승격된 명령을 실행하는 동안 재사용하거나 보존하려는 환경 변수를 설정했을 수 있으며 `-E` 옵션을 사용하여 그렇게 할 수 있습니다.

예를 들어, Enterprise Linux 시스템이 기업 인트라넷 또는 가상 사설망\(VPN\)에 연결된 경우 아웃바운드 인터넷 액세스를 얻기 위해 프록시 설정이 적용될 수 있습니다.

프록시 액세스를 위해 터미널 명령이 의존하는 환경 변수는 `http_proxy`, `https_proxy` 및 `no_proxy`이며 `~/.bashrc` 구성 파일에서 설정할 수 있습니다.:

```
export http_proxy=http://proxy.example.com:8080
export https_proxy=https://proxy.example.com:8080
export no_proxy=localhost,127.0.0.1
```

로그아웃하지 않고 `source` 명령을 실행하여 세션 환경 변수를 새로 고칩니다.:

```
source ~/.bashrc
```

`sudo` 명령은 사용자 세션 내에서 환경 변수로 구성한 프록시 설정을 사용할 수 있습니다. 예를 들어 관리자 권한으로 `curl` 명령을 실행하려면:

```
sudo -E curl https://www.example.com
```

**Note:**

관리자는 선택적으로 쉘 스크립트에서 구성한 다음 `/etc/profile.d/` 디렉토리에 해당 파일을 저장하여 시스템 전체 프록시 환경 변수를 설정할 수 있습니다.

또한 `sudo` 액세스를 사용하여 승격된 `root` 셸을 시작할 수도 있습니다. `-s` 옵션은 사용자를 `root` 사용자로서 `root` 쉘로 승격시킵니다. `-i` 옵션은 사용자 프로필과 쉘 구성을 모두 유지하면서 사용자를 `root` 쉘로 승격시킵니다.:

```
sudo -i
```

관리 명령 실행을 마쳤으면 `root` 셸을 종료하고 `exit` 명령을 사용하여 표준 사용자 권한 수준으로 돌아갑니다.

## visudo 명령 사용

시스템의 다른 사용자와의 변경 충돌 위험 없이 `vi` 텍스트 편집기에서 `/etc/sudoers` 파일을 편집하려면 `visudo` 명령을 사용하십시오.:

```
sudo visudo
```

[sudoers 파일에 사용자 권한 추가](ko-userauth-GrantingsudoAccesstoUsers.md#sudoers-파일에-사용자-권한-추가) 그리고 `visudo(8)` 매뉴얼 페이지.

관리자는 `visudo` 명령을 사용하여 `/etc/sudoers.d/` 디렉토리에 있는 개별 사용자의 권한 파일을 관리할 수도 있습니다. 자세한 내용은, 참조: sudoers 파일에 사용자 권한 추가
.

## sudoers.d 디렉토리에 사용자 권한 추가

특정 사용자에 대한 권한을 설정하려면 해당 사용자에 대한 파일을 `/etc/sudoers.d` 디렉터리에 추가하세요. 예를 들어 `alice` 사용자에게 `sudo` 권한을 설정하려면:

```
sudo visudo -f /etc/sudoers.d/alice
```

다음 형식으로 `/etc/sudoers.d/alice`에 권한을 추가할 수 있습니다.:

```
*username*        *hostname*=*command*
```

`username`은 사용자의 이름이고 `hostname`은 권한을 정의하는 호스트의 이름이며 `command`는 전체 실행 파일 경로와 옵션이 포함된 허용되는 명령입니다. 옵션을 지정하지 않으면 사용자는 전체 옵션을 사용하여 명령을 실행할 수 있습니다.

예를 들어, 모든 호스트에 `sudo dnf` 명령을 사용하여 패키지를 설치할 수 있는 `alice` 권한을 사용자에게 부여하려면:

```
alice           ALL = /usr/bin/dnf
```

같은 줄에 여러 개의 쉼표로 구분된 명령을 추가할 수도 있습니다. 사용자 `alice`가 모든 호스트에서 `sudo dnf` 및 `sudo yum` 명령을 모두 실행할 수 있도록 허용하려면:

```
alice           ALL = /usr/bin/dnf, /usr/bin/yum
```

`alice` 사용자는 권한 있는 명령을 실행할 때 여전히 `sudo`를 사용해야 합니다.:

```
sudo dnf install *package*
```

## sudoers 파일에 사용자 권한 추가

`/etc/sudoers` 파일에서 직접 사용자 권한을 설정하려면 파일 위치를 지정하지 않고 `visudo` 명령을 실행하세요.:

```
sudo visudo
```

`/etc/sudoers.d/` 디렉토리의 사용자 파일에 해당 권한을 추가하는 경우와 동일한 형식으로 `/etc/sudoers` 파일에 권한을 추가할 수 있습니다.

두 경우 모두 각 명령을 개별적으로 지정하는 대신 별칭을 사용하여 더 넓은 권한 범주를 허용할 수 있습니다. `ALL` 별칭은 모든 권한에 대한 와일드카드로 작동하므로 사용자 `bob`이 모든 호스트의 모든 명령에 대해 sudo 권한을 갖도록 설정합니다.:

```
bob             ALL=(ALL)       ALL
```

더 많은 별칭 카테고리는 `/etc/sudoers` 파일과 `sudoers(5)` 매뉴얼 페이지에 나열되어 있습니다. 다음 형식으로 별칭을 만들 수 있습니다.:

```
Cmnd_Alias *ALIAS* = *command*
```

또한 같은 줄에 여러 개의 쉼표로 구분된 별칭을 추가할 수도 있습니다. 예를 들어 사용자 `alice`에게 시스템 서비스 및 소프트웨어 패키지를 관리할 수 있는 권한을 부여하려면:

```
Cmnd_Alias SOFTWARE = /bin/rpm, /usr/bin/up2date, /usr/bin/yum
Cmnd_Alias SERVICES = /sbin/service, /sbin/chkconfig, /usr/bin/systemctl start, /usr/bin/systemctl stop, /usr/bin/systemctl reload, /usr/bin/systemctl restart, /usr/bin/systemctl status, /usr/bin/systemctl enable, /usr/bin/systemctl disable
alice           ALL= SERVICES, SOFTWARE
```

두 사용자 모두 권한 있는 명령을 실행할 때 `sudo`를 사용해야 합니다.:

```
sudo systemctl restart *service*
```

## 그룹을 사용하여 사용자 권한 관리

각 개별 사용자에 대해 서로 다른 `sudo` 액세스 수준을 지정하는 대신 그룹 이름에 `%` 기호를 추가하여 선택적으로 그룹 수준에서 `sudo` 액세스를 관리할 수 있습니다.

예를 들어, `/etc/sudoers.d/` 디렉토리에 `example`이라는 기존 그룹에 대한 권한을 정의한 다음 해당 그룹에 `alice` 사용자를 추가하려면:

1. `visudo` 명령을 사용하여 `/etc/sudoers.d/example` 파일을 생성합니다.:

   ```
   sudo visudo /etc/sudoers.d/example
   ```

2. 시스템 서비스 및 소프트웨어 패키지를 관리하기 위해 `example` 그룹 권한을 부여합니다.:

   ```
   %example        ALL= SERVICES, SOFTWARE
   ```

3. `example` 그룹에 `alice` 사용자를 추가합니다.:

   ```
   sudo usermod -aG example alice
   ```

또는 `/etc/sudoers` 파일에서 직접 그룹 권한을 설정할 수 있습니다. 예를 들어 `bob` 사용자에게 모든 호스트에서 전체 `sudo` 액세스 권한을 부여하려면 기존 그룹 `wheel`을 활성화한 다음 이 그룹에 `bob` 사용자를 추가합니다.:

1. `visudo` 명령을 사용하여 `/etc/sudoers` 파일을 엽니다.:

   ```
   sudo visudo
   ```

2. `/etc/suiders` 파일에서 다음 행의 처음부터 주석 `#` 기호를 제거합니다.:

   ```
   %wheel          ALL=(ALL)       ALL
   ```

3. `bob` 사용자를 `wheel` 그룹에 추가하여 모든 호스트에서 전체 `sudo` 액세스 권한을 부여하세요.:

   ```
   sudo usermod -aG wheel bob
   ```
