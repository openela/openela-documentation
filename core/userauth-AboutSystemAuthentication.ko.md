<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# 시스템 인증

인증은 사용자와 같은 개체의 신원을 시스템에 확인하여 시스템 보안을 구현하는 방법입니다. 사용자는 사용자 이름과 비밀번호를 제공하여 로그인하고, OS는 이 정보를 시스템에 저장된 데이터와 비교하여 사용자의 신원을 인증합니다. 로그인 자격 증명이 일치하고 사용자 계정이 활성화된 경우 사용자가 인증되고 시스템에 성공적으로 액세스할 수 있습니다.

## Enterprise Linux에서 인증

Enterprise Linux에서 인증은 프로필 기반입니다. 각 프로필에는 다양한 메커니즘을 사용하여 시스템 액세스를 인증하는 사전 정의된 기능이 있습니다.

다음 프로필을 사용할 수 있습니다.

- `sssd` 프로필: `sssd` 서비스를 사용하여 시스템 인증을 수행합니다.

- `winbind` 프로필: `winbind` 서비스를 사용하여 시스템 인증을 수행합니다.

- `minimal` 프로필: 시스템 파일을 사용하여 로컬 사용자에 대한 시스템 인증을 수행합니다.

Enterprise Linux 설치 후 기본적으로 `sssd` 프로필이 선택되어 시스템 인증을 관리합니다. 이 프로필은 PAM 인증, Kerberos 등을 포함한 대부분의 인증 사례를 다룹니다.

시스템 인증은 Enterprise Linux의 프로필만 사용하도록 제한되지 않습니다. 원하는 경우 공급업체에서 제공하는 프로필을 사용할 수도 있습니다. 조직 요구 사항을 준수하는 인증을 시행하기 위해 사용자 정의 프로필을 만들 수도 있습니다.

유연성을 더하기 위해 활성 기능을 수정하여 프로필을 재구성할 수도 있습니다. 예를 들어 LDAP, FreeIPA, Active Directory와 같은 다양한 백엔드 디렉터리 서비스를 사용하도록 프로필을 설정할 수 있습니다. 또한 디렉터리 서비스와 함께 SSSD를 사용하면 액세스 요구 사항이 서로 다른 많은 사용자와 시스템이 존재하는 환경에서 사용자 및 그룹 관리를 중앙 집중화하고 단순화할 수 있습니다.

## 프로필 및 지원되는 기능

각 프로필에는 프로필 서비스가 스마트 카드 인증, 지문 인증, Kerberos 등과 같은 특정 인증 방법을 수행하도록 활성화할 수 있는 관련 기능이 있습니다. 프로필을 선택하고 기본 기능을 활성화하면 `authselect`는 해당 기능의 적절한 구성 파일을 자동으로 읽어 관련 인증 프로세스를 실행합니다. 호스트에 로그인하는 모든 사용자는 구성된 프로필을 기반으로 인증됩니다.

다음 표에는 프로필과 해당 지원 기능이 나와 있습니다.:

| Feature Name                     | Description                                                                             |
| -------------------------------- | --------------------------------------------------------------------------------------- |
| `with-faillock`                  | 인증 실패 횟수가 너무 많으면 계정을 잠급니다.                                              |
| `with-mkhomedir`                 | 사용자가 처음 로그인할 때 홈 디렉터리를 만듭니다.                                            |
| `with-ecryptfs`                  | 자동 사용자별 ecryptfs를 활성화합니다.                                               |
| `with-smartcard`                 | SSSD를 통해 스마트 카드를 인증합니다.                                                 |
| `with-smartcard-lock-on-removal` | 스마트 카드를 제거하면 화면을 잠급니다. `with-smartcard`도 활성화되어 있어야 합니다. |
| `with-smartcard-required`        | 스마트 카드 인증만 작동됩니다. `with-smartcard`도 활성화되어 있어야 합니다.      |
| `with-fingerprint`               | 지문 판독기를 통해 인증합니다.                                                       |
| `with-silent-lastlog`            | 로그인 중 `pam_lostlog` 메시지 생성 비활성화                                                         |
| `with-sudo`                      | `/etc/sudoers` 이외의 규칙에 SSSD를 사용하려면 `sudo`를 활성화하세요.                      |
| `with-pamaccess`                 | 계정 인증은 `/etc/access.conf`를 참조하세요.                                       |
| `without-nullock`                | `pam_unix`에 `nullock` 매개변수를 추가하지 마세요.                                   |

| Feature Name          | Description                                           |
| --------------------- | ----------------------------------------------------- |
| `with-faillock`       | 인증 실패 횟수가 너무 많으면 계정을 잠급니다.            |
| `with-mkhomedir`      | 사용자가 처음 로그인할 때 홈 디렉터리를 만듭니다.          |
| `with-ecryptfs`       | 자동 사용자별 ecryptfs를 활성화합니다.             |
| `with-fingerprint`    | 지문 판독기를 통해 인증합니다.                     |
| `with-krb5`           | Kerberos 인증을 사용합니다.                   |
| `with-silent-lastlog` | 로그인 중 pam\_lostlog 메시지 생성 비활성화  |
| `with-pamaccess`      | 계정 인증은 `/etc/access.conf`를 참조하세요.     |
| `without-nullock`     | `pam_unix`에 `nullock` 매개변수를 추가하지 마세요. |

각 프로필에 대한 자세한 내용은 프로필의 해당 `/usr/share/authselect/default/*profile*/README` 파일을 참조하세요. `authselect-profiles(5)` 매뉴얼 페이지도 참조하세요.

## authselect 유틸리티

`authselect` 유틸리티는 시스템에서 인증을 구성하기 위한 Enterprise Linux 도구입니다. 이 도구는 시스템 인증 프로필을 관리하며 모든 Enterprise Linux 8 설치에 자동으로 포함됩니다.

`authselect` 유틸리티는 다음 구성 요소로 구성됩니다.

- 시스템 인증을 관리하는 `authselect` 명령. 적절한 관리자 권한이 있는 사용자만 이 명령을 실행할 수 있습니다.

- 특정 인증 메커니즘을 적용하는 프로필. 이러한 프로필은 OpenELA에서 제공하거나, 공급업체에서 제공하거나, 조직에서 생성한 프로필일 수 있습니다.

다양한 프로필을 효율적으로 관리하기 위해 `authselect`는 해당 파일에 다양한 유형의 프로필을 저장합니다.

- `/usr/share/authselect/default`에는 Enterprise Linux에서 제공하는 OpenELA 제공 프로필이 포함되어 있습니다.

- `/usr/share/authselect/vendor`에는 공급업체에서 제공하는 프로필이 포함되어 있습니다. 이러한 프로필은 기본 디렉터리에 있는 프로필을 재정의할 수 있습니다.

- `/etc/authselect/custom`에는 특정 환경에 대해 생성한 모든 프로필이 포함됩니다.

**중요:**

`authselect` 유틸리티는 선택한 프로필의 사양을 적용합니다. 그러나 `authselect`는 프로필의 기반이 되는 서비스의 구성 파일을 변경하지 않습니다. 예를 들어 `sssd` 프로필을 사용하는 경우 서비스가 제대로 작동하려면 SSSD를 구성해야 합니다. 프로필 서비스를 구성하려면 적절한 설명서를 참조하세요. 또한 서비스가 시작되고 활성화되었는지 확인해야 합니다.

유틸리티에 대한 자세한 내용은 `authselect(8)` 매뉴얼 페이지를 참조하세요.
