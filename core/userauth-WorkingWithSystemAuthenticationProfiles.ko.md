<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# 시스템 인증 프로필 작업

`authselect` 명령에는 생성, 삭제, 다른 프로필로 전환 및 프로필 기능 수정을 위한 다양한 하위 명령, 인수 및 옵션이 있습니다. 이 구성 도구를 사용하려면 사용자에게 적절한 권한이 있어야 합니다.

## 프로필 정보 표시

시스템에서 현재 활성화된 프로필을 확인하려면 다음을 입력합니다.:

```
sudo authselect current
```

```
Profile ID: sssd
Enabled features:
- with-fingerprint
- with-silent-lastlog
```

명령의 출력은 `ssd` 프로필이 현재 활성화되어 있음을 나타냅니다. 최소한 `pam_fprintd`를 통해 지문 판독기를 사용한 인증이 시행됩니다. 또한 사용자가 로그인할 때 `pam_lastlog` 메시지가 화면에 표시되지 않습니다.

## Configuring Profile Features

프로필의 활성화된 기능에 따라 시스템의 인증 방식이 결정됩니다. 두 가지 방법 중 하나로 프로필 기능을 활성화할 수 있습니다.:

- 현재 프로필에서 활성화할 추가 기능을 지정합니다.

- 선택한 프로필의 현재 기능을 바꿉니다. 이 방법은 다음에서 논의됩니다.[Windbind 프로필 선택](ko-userauth-WorkingWithSystemAuthenticationProfiles.md#windbind-프로필-선택).

### 프로필 기능 활성화

1. \(선택 사항\): 현재 프로필을 식별합니다.

   추가 기능 활성화는 현재 프로필에서만 작동합니다. 선택하지 않은 프로필에서는 이 절차가 작동하지 않습니다.

   ```
   sudo authselect current
   ```

2. 필요한 경우 기능이 제대로 작동하기 위한 기능 요구 사항을 식별합니다.

   ```
   sudo authselect requirements *profile* *feature*
   ```

3. 필요에 따라 표시된 나열된 기능 요구 사항을 완료하십시오..

4. 기능을 활성화합니다.

   ```
   sudo authselect enable-feature *feature*
   ```

   한 번에 하나씩만 기능을 활성화할 수 있습니다.

### 프로필 기능 비활성화

`disable-feature` 하위 명령을 사용하세요.

```
sudo authselect disable-feature *feature*                  
```

### 프로필에 기능 추가

다음 예에서는 계정 잠금을 설정하고 홈 디렉터리를 기본 `sssd` 프로필의 추가 기능으로 정의하는 방법을 보여줍니다.

1. 너무 많은 인증 실패 후 계정을 자동으로 잠그기 위한 요구 사항을 결정합니다\(`with-faillock`\):

   ```
   sudo authselect requirements sssd with-faillock
   ```

   ```
   Make sure that SSSD service is configured and enabled. See SSSD documentation for more 
   information.
   ```

2. 사용자가 처음으로 \(`with-mkhomedir`\) 로그인할 때 사용자 홈 디렉터리를 자동으로 생성하기 위한 요구 사항을 결정합니다..

   ```
   sudo authselect requirements sssd with-mkhomedir
   ```

   ```
   Make sure that SSSD service is configured and enabled. See SSSD documentation for more 
   information.
    
   - with-mkhomedir is selected, make sure pam_oddjob_mkhomedir module
     is present and oddjobd service is enabled
     - systemctl enable oddjobd.service
     - systemctl start oddjobd.service
   ```

3. 활성화하려는 기능의 요구 사항을 충족합니다.

4. 두 프로필 기능을 모두 활성화합니다.:

   ```
   sudo authselect enable-feature with-faillock
   ```

   ```
   sudo authselect enable-feature with-mkhomedir
   ```

5. 두 프로필 기능이 모두 활성화되었는지 확인하세요.:

   ```
   sudo authselect current
   ```

   ```
   Profile ID: sssd
   Enabled features:
   - with-fingerprint
   - with-silent-lastlog
   - with-faillock
   - with-mkhomedir
   ```

### PAM 액세스 기능 활성화

다음 예는 사용자를 인증하고 권한을 부여하기 위해 `/etc/security/access.conf`를 확인하도록 시스템에 지시하는 방법을 보여줍니다. 이 경우 `sssd`에 대해 활성화된 기능으로 PAM 액세스 기능을 추가해야 합니다.

1. PAM 액세스 자동 활성화:

   ```
   sudo authselect requirements sssd with-pamaccess
   ```

   ```
   Make sure that SSSD service is configured and enabled. See SSSD documentation for more 
   information.
   ```

2. PAM 액세스 프로필 기능 활성화:

   ```
   sudo authselect enable-feature sssd with-pamaccess
   ```

3. PAM 액세스 프로필 기능이 활성화되었는지 확인하세요.:

   ```
   sudo authselect current
   ```

   ```
   Profile ID: sssd
   Enabled features:
   - with-fingerprint
   - with-silent-lastlog
   - with-faillock
   - with-mkhomedir
   - with-pamaccess
   ```

**Note:**

이전 예에서는 기능이 올바르게 작동하도록 `/etc/security/access.conf`를 구성했다고 가정합니다. 자세한 내용은 `access.conf(5)` 매뉴얼 페이지를 참조하세요.

## Windbind 프로필 선택

Winbind는 Windows 서버에서 사용자 및 그룹 정보를 확인하는 클라이언트 측 서비스입니다. Enterprise Linux가 Windows 사용자 및 그룹과 작업할 수 있도록 하려면 이 프로필을 사용하십시오.

1. `samba-winbind` 패키지 설치.

   ```
   sudo dnf install samba-winbind -y
   ```

2. `winbind` 프로필을 선택합니다.

   프로필을 선택할 때 동일한 명령으로 여러 기능을 활성화할 수 있습니다.

   ```
   sudo authselect select winbind with-faillock with-mkhomedir [*options*]
   ```

   ```
   Profile "winbind" was selected.
   The following nsswitch maps are overwritten by the profile:
   - passwd
   - group

   Make sure that winbind service is configured and enabled. See winbind documentation for more information.
    
   - with-mkhomedir is selected, make sure pam_oddjob_mkhomedir module
     is present and oddjobd service is enabled
     - systemctl enable oddjobd.service
     - systemctl start oddjobd.service
   ```

   `authselect select` 명령과 함께 사용할 수 있는 다른 옵션에 대해서는 `authselect(8)` 매뉴얼 페이지를 참조하세요.

3. 프로필에 활성화한 기능의 요구 사항을 충족합니다.

4. `winbin` 서비스를 시작합니다.

   ```
   sudo systemctl start winbind
   ```

   ```
   sudo systemctl enable winbind
   ```

**Note:**

이미 활성화되어 있는 현재 프로필의 기능을 수정하는 경우 수정된 기능은 이전에 활성화된 모든 기능을 대체합니다.

## 기성 프로파일 수정

또한 프로필은 `/etc/nsswitch.conf` 파일에 저장된 정보를 사용하여 인증을 시행합니다. 그러나 이미 만들어진 프로필을 수정하고 사용자 정의하려면 `/etc/user-nsswitch.conf` 파일에서 해당 구성 속성을 지정하십시오. `/etc/nsswitch.conf`를 직접 편집하지 마십시오.

1. 필요한 경우 프로파일을 선택하여 현재로 만듭니다.

   ```
   sudo authselect select sssd
   ```

2. 필요에 따라 `/etc/authselect/user-nsswitch.conf` 파일을 편집합니다.

   **Note:**

   파일에서 다음 구성을 수정하지 마십시오. 그렇게 하면 해당 수정 사항이 무시됩니다.

   - `passwd`

   - `group`

   - `netgroup`

   - `automount`

   - `services`

3. 변경 사항을 적용합니다.

   ```
   sudo authselect apply-changes
   ```

   `/etc/authselect/user-nsswitch.conf`의 변경 사항은 `/etc/nsswitch.conf`에 적용되며 현재 프로필에서 사용됩니다.

**중요:**

시스템이 ID 관리 또는 Active Directory를 사용하는 환경의 일부인 경우 `authselect`를 사용하여 인증을 관리하지 마십시오. 호스트가 ID 관리 또는 Active Directory에 가입되면 해당 도구가 환경 인증 관리를 담당합니다.

## 사용자 정의 프로필 만들기

Enterprise Linux에 포함된 프로필이나 공급업체에서 제공한 프로필을 사용하지 않으려는 경우 고유한 특정 프로필을 만들 수 있습니다.

1. 프로필을 만듭니다.

   ```
   sudo authselect create-profile *newprofile* -b *template* --symlink-meta --symlink-pam
   ```

   - **_newprofile_**

     사용자 정의 프로필의 이름입니다.

   - **_template_**

     `sssd` 또는 `winbind`인 사용자 정의 프로필에 사용되는 기본입니다.

   - **--symlink-meta**

     기본으로 사용 중인 템플릿 프로필의 원래 디렉터리에 있는 메타 파일에 대한 심볼릭 링크를 생성합니다.

   - **--symlink-pam**

     기본으로 사용 중인 템플릿 프로필의 원래 디렉터리에 PAM 템플릿에 대한 기호 링크를 생성합니다.

   이 명령은 기본 디렉토리의 파일에 대한 심볼릭 링크가 포함된 `/etc/authselect/custom/*newprofile*` 디렉토리를 생성합니다. 이 디렉토리에서 심볼릭 링크가 **아닌** 유일한 파일은 `nsswitch.conf`입니다.

2. 원하는 대로 `/etc/authselect/custom/*newprofile*/nsswitch.conf` 파일을 편집합니다.

3. 사용자 정의 프로필을 선택하세요.

   ```
   sudo authselect select custom/*newprofile*                        
   ```

   또한 이 명령은 원본 `/etc/nsswitch.conf` 파일의 백업을 생성하고 이를 사용자 정의 프로필 디렉터리의 해당 파일에 대한 심볼릭 링크로 대체합니다.

   심볼릭 링크 `/etc/nsswitch.conf`를 원본 `/etc/nsswitch.conf.bak`와 비교하고 원본 파일의 내용이 그대로 유지되는지 확인하여 이 결과를 테스트할 수 있습니다.

4. 필요에 따라 새 프로필의 기능을 활성화합니다.

   [프로필 기능 활성화](ko-userauth-WorkingWithSystemAuthenticationProfiles.md#프로필-기능-활성화)을 참조하세요.

5. \(선택 사항\) 사용자 정의 프로필의 구성을 확인합니다.

   ```
   sudo authselect current
   ```
