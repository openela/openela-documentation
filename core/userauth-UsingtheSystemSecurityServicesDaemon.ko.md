<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# 시스템 보안 서비스 데몬 사용

시스템 보안 서비스 데몬\(SSSD\) 기능은 클라이언트 시스템에서 원격 ID 및 인증 공급자에 대한 액세스를 제공합니다. SSSD는 로컬 클라이언트와 사용자가 구성하는 백엔드 공급자 간의 중개자 역할을 합니다.

SSSD를 구성하면 다음과 같은 이점이 있습니다.

- 시스템 부하 감소

  클라이언트는 식별 또는 인증 서버에 직접 접속할 필요가 없습니다.

- 오프라인 인증

  사용자 ID 및 자격 증명의 캐시를 유지하도록 SSSD를 구성할 수 있습니다.

- 싱글 사인온 액세스

  네트워크 자격 증명을 저장하도록 SSSD를 구성하는 경우 사용자는 로컬 시스템에서 세션당 한 번만 인증하면 네트워크 리소스에 액세스할 수 있습니다.

Enterprise Linux `sssd` 프로필이 기본적으로 사용되므로 SSSD 서비스도 새로 설치된 시스템에 자동으로 설치되고 활성화됩니다. 기본 구성에서는 시스템의 액세스 및 인증을 관리하기 위해 플러그형 인증 모듈\(PAM\) 및 이름 서비스 스위치\(NSS\)를 사용합니다. 다른 인증 서비스를 사용하거나 기본 설정에 대한 대체 값을 사용하도록 구성을 사용자 정의하려는 경우가 아니면 추가 구성이 필요하지 않습니다.

SSSD에 대한 자세한 내용은 [https://sssd.io/](https://sssd.io/)를 참조하세요.

## SSSD 사용자 정의

기본적으로 `sssd` 프로필에 사용되는 SSSD 서비스는 시스템 액세스 및 인증 관리를 위해 플러그형 인증 모듈\(PAM\)과 이름 서비스 스위치\(NSS\)를 사용합니다. SSSD 인증을 사용자 정의하기 위해 프로필에 대한 추가 기능을 활성화하면 활성화된 기능에 대해 SSSD도 구성해야 합니다.

`/etc/sssd/conf.d` 디렉토리 내에 구성 파일을 생성하여 SSSD 구성을 사용자 정의합니다. SSSD가 시작될 때 활성화하려면 각 구성 파일에 `.conf` 접미사가 있어야 합니다. 구성 파일은 ini 스타일 구문을 형식으로 사용합니다. 콘텐츠는 대괄호로 식별되는 섹션과 `키 = 값` 항목으로 나열되는 매개변수로 구성됩니다. SSSD용으로 제공되는 매뉴얼 페이지는 포괄적이며 사용 가능한 옵션에 대한 자세한 정보를 제공합니다.

다음 예에서는 Kerberos가 구성된 LDAP 공급자에 대해 인증하도록 SSSD를 구성하는 방법을 보여줍니다.:

1. 기능에 대한 구성 파일을 생성하고 `/etc/sssd/conf.d`에 저장합니다(예: `/etc/sssd/conf.d/00-ldap.conf`).

2. 다음과 같은 매개변수 정의로 `/etc/sssd/conf.d/00-ldap.conf`를 구성합니다.:

   ```
   [sssd]
   config_file_version = 2
   domains = LDAP
   services = nss, pam

   [domain/LDAP]
   id_provider = ldap
   ldap_uri = ldap://ldap.mydom.com
   ldap_search_base = dc=mydom,dc=com

   auth_provider = krb5
   krb5_server = krbsvr.mydom.com
   krb5_realm = MYDOM.COM
   cache_credentials = true

   min_id = 5000
   max_id = 25000
   enumerate = false

   [nss]
   filter_groups = root
   filter_users = root
   reconnection_retries = 3
   entry_cache_timeout = 300

   [pam]
   reconnection_retries = 3
   offline_credentials_expiration = 2
   offline_failed_login_attempts = 3
   offline_failed_login_delay = 5
   ```

   - **`[sssd]`**

     SSSD 모니터 옵션, 도메인 및 서비스에 대한 구성 설정이 포함되어 있습니다. SSSD 모니터 서비스는 SSSD가 제공하는 서비스를 관리합니다.

     - 여기에는 이름 서비스 스위치용 `nss`와 플러그형 인증 모듈용 `pam`이 포함되어야 합니다.

     - `domains` 항목은 인증 도메인을 정의하는 섹션의 이름을 지정합니다.

   - **`[domain/LDAP]`**

     Kerberos 인증을 사용하는 LDAP ID 공급자에 대한 도메인을 정의합니다. 각 도메인은 사용자 정보가 저장되는 위치, 인증 방법 및 구성 옵션을 정의합니다. SSSD는 OpenLDAP, Red Hat Directory Server, IPA 및 Microsoft Active Directory와 같은 LDAP ID 공급자와 작동할 수 있으며 기본 LDAP 또는 Kerberos 인증을 사용할 수 있습니다.

     - `id_provider`는 공급자 유형\(이 예에서는 LDAP\)을 지정합니다.

     - `ldap_uri`는 SSSD가 연결할 수 있는 LDAP 서버의 Universal Resource Identifiers\(URIs\) 목록을 쉼표로 구분하여 지정합니다.

     - `ldap_search_base`는 일반 이름 \(`cn`\)과 같은 상대 고유 이름 \(RDN\)에 대해 LDAP 사용자 작업을 수행할 때 SSSD가 사용해야 하는 기본 고유 이름 \(`dn`\)을 지정합니다.

     - `auth_provider` 항목은 인증 공급자\(이 예에서는 Kerberos\)를 지정합니다.

     - `krb5_server`는 SSSD가 연결할 수 있는 Kerberos 서버의 목록을 선호도 순으로 쉼표로 구분하여 지정합니다.

     - `krb5_realm`은 Kerberos 영역을 지정합니다.

     - `cache_credentials`는 SSSD가 티켓, 세션 키, 기타 식별 정보와 같은 사용자 자격 증명을 캐시하여 오프라인 인증 및 Single Sign-On을 지원하는지 여부를 지정합니다.

       **Note:**

       SSSD가 LDAP 서버에서 Kerberos 인증을 사용하도록 허용하려면 단순 인증 및 보안 계층\(SASL\)과 일반 보안 서비스 API\(GSSAPI\)를 모두 사용하도록 LDAP 서버를 구성해야 합니다. OpenLDAP용 SASL 및 GSSAPI 구성에 대한 자세한 내용은 [https://www.openldap.org/doc/admin24/sasl.html](https://www.openldap.org/doc/admin24/sasl.html)을 참조하세요.

     - `min_id` 및 `max_id`는 사용자 및 그룹 ID 값의 상한 및 하한을 지정합니다.

     - `enumerate`는 SSSD가 공급자에서 사용할 수 있는 사용자 및 그룹의 전체 목록을 캐시할지 여부를 지정합니다. 도메인에 상대적으로 적은 수의 사용자나 그룹이 포함되어 있지 않는 한 권장 설정은 `False`입니다.

   - **`[nss]`**

     SSS 데이터베이스를 NSS와 통합하는 이름 서비스 스위치\(NSS\) 모듈을 구성합니다.

     - `filter_users` 및 `filter_groups`는 NSS가 SSS에서 검색되는 지정된 사용자 및 그룹에 대한 정보를 추출하는 것을 방지합니다.

     - `reconnection_retries`는 데이터 공급자가 충돌할 경우 SSSD가 다시 연결을 시도해야 하는 횟수를 지정합니다.

     - `enum_cache_timeout`은 SSSD가 사용자 정보 요청을 캐시하는 시간(초)을 지정합니다.

   - **`[pam]`**

     SSSD를 PAM과 통합하는 PAM 모듈을 구성합니다.

     - `offline_credentials_expiration`은 인증 공급자가 오프라인인 경우 캐시된 로그인을 허용하는 일수를 지정합니다.

     - `offline_failed_login_attempts`는 인증 공급자가 오프라인인 경우 허용되는 로그인 시도 실패 횟수를 지정합니다.

     - `offline_failed_login_delay`는 허용된 실패한 로그인 시도 제한이 초과된 후 새로운 로그인 시도가 허용되기까지의 시간(분)을 지정합니다.

3. `/etc/sssd/conf.d/00-ldap.conf` 모드를 0600으로 변경합니다.:

   ```
   sudo chmod 0600 /etc/sssd/conf.d/00-ldap.conf
   ```

4. 아직 시작되지 않은 경우 SSSD 서비스를 활성화하십시오.:

5. `sssd` 프로필을 선택하세요.

   ```
   sudo authselect select sssd
   ```

SSSD 서비스에 대한 자세한 내용은 `sssd(8)` 매뉴얼 페이지와 [https://pagure.io/SSSD/sssd/](https://pagure.io/SSSD/sssd/)를 참조하세요. 또한 `sssd.conf(5)`, `sssd-ldap(5)`, `sssd-krb5(5)`, `sssd-ipa(5)` 및 기타 매뉴얼 페이지를 참조할 수 있습니다.

## 플러그형 인증 모듈

플러그형 인증 모듈\(PAM\) 기능은 `sssd` 프로필에서 사용하는 인증 메커니즘으로, 애플리케이션에서 인증을 사용하여 사용자의 신원을 확인하는 방법을 구성할 수 있습니다. `/etc/pam.d` 디렉토리에 있는 PAM 구성 파일은 애플리케이션에 대한 인증 절차를 설명합니다. 각 구성 파일의 이름은 모듈이 인증을 제공하는 애플리케이션의 이름과 동일하거나 유사합니다. 예를 들어 `passwd` 및 `sudo`에 대한 구성 파일의 이름은 `passwd` 및 `sudo`입니다.

각 PAM 구성 파일에는 인증 모듈에 대한 호출 목록 또는 _스택_이 포함되어 있습니다. 예를 들어 다음 목록은 `login` 구성 파일의 기본 콘텐츠를 보여줍니다.:

```
#%PAM-1.0
auth [user_unknown=ignore success=ok ignore=ignore default=bad] pam_securetty.so
auth       include      system-auth
auth       include      postlogin
account    required     pam_nologin.so
account    include      system-auth
password   include      system-auth
# pam_selinux.so close should be the first session rule
session    required     pam_selinux.so close
session    required     pam_loginuid.so
session    optional     pam_console.so
# pam_selinux.so open should only be followed by sessions to be executed in the user context
session    required     pam_selinux.so open
session    required     pam_namespace.so
session    optional     pam_keyinit.so force revoke
session    include      system-auth
session    include      postlogin
-session   optional     pam_ck_connector.so
```

파일의 주석은 `#` 다음으로 시작합니다. 나머지 행은 각각 작업 유형, 제어 플래그, `pam_rootok.so`와 같은 모듈 이름 또는 포함된 구성 파일(`system-auth`와 같은) 이름, 모듈에 대한 인수를 정의합니다. PAM은 인증 모듈을 `/usr/lib64/security`에서 공유 라이브러리로 제공합니다.

특정 작업 유형의 경우 PAM은 스택을 위에서 아래로 읽고 구성 파일에 나열된 모듈을 호출합니다. 각 모듈은 호출 시 성공 또는 실패 결과를 생성합니다.

다음 작업 유형이 사용되도록 정의됩니다.:

- **`auth`**

  모듈은 사용자가 서비스나 애플리케이션을 사용할 수 있도록 인증되었는지 또는 권한이 있는지 테스트합니다. 예를 들어 모듈은 비밀번호를 요청하고 확인할 수 있습니다. 이러한 모듈은 그룹 멤버십이나 Kerberos 티켓과 같은 자격 증명을 설정할 수도 있습니다.

- **`account`**

  이 모듈은 인증된 사용자가 서비스나 애플리케이션에 액세스할 수 있는지 여부를 테스트합니다. 예를 들어 모듈은 사용자 계정이 만료되었는지 또는 사용자가 특정 시간에 서비스를 사용할 수 있는지 확인할 수 있습니다.

- **`password`**

  모듈은 인증 토큰 업데이트를 처리합니다.

- **`session`**

  모듈은 사용자 세션을 구성 및 관리하여 사용자의 홈 디렉터리 마운트 또는 마운트 해제와 같은 작업을 수행합니다.

작업 유형 앞에 대시 \(`-`\)가 붙는 경우 PAM은 모듈이 누락된 경우 시스템 로그 항목 생성을 추가하지 않습니다.

`include`를 제외하고 제어 플래그는 PAM에게 모듈 실행 결과로 무엇을 할지 알려줍니다. 다음 제어 플래그는 사용을 위해 정의됩니다.:

- **`optional`**

  서비스에 대해 나열된 유일한 모듈인 경우 인증을 위해 모듈이 필요합니다.

- **`required`**

  액세스 권한을 부여하려면 모듈이 성공해야 합니다. PAM은 모듈의 성공 여부에 관계없이 스택의 나머지 모듈을 계속해서 처리합니다. PAM은 사용자에게 오류를 즉시 알리지 않습니다.

- **`requisite`**

  액세스 권한을 부여하려면 모듈이 성공해야 합니다. 모듈이 성공하면 PAM은 스택의 나머지 모듈을 계속 처리합니다. 그러나 모듈이 실패하면 PAM은 사용자에게 즉시 알리고 스택의 나머지 모듈을 계속 처리하지 않습니다.

- **`sufficient`**

  모듈이 성공하면 PAM은 동일한 작업 유형의 나머지 모듈을 처리하지 않습니다. 모듈이 실패하면 PAM은 동일한 작업 유형의 나머지 모듈을 처리하여 전반적인 성공 또는 실패를 결정합니다.

제어 플래그 필드는 모듈이 반환하는 값에 따라 PAM이 수행하는 작업을 지정하는 하나 이상의 규칙을 정의할 수도 있습니다. 각 규칙은 `*value*=*action*` 형식을 취하고 규칙은 대괄호로 묶입니다.

```
[user_unknown=ignore success=ok ignore=ignore default=bad]
```

모듈에서 반환된 결과가 값과 일치하면 PAM은 해당 작업을 사용하고, 일치하는 항목이 없으면 기본 작업을 사용합니다.

`include` 플래그는 PAM이 인수로 지정된 PAM 구성 파일도 참조해야 함을 지정합니다.

대부분의 인증 모듈과 PAM 구성 파일에는 자체 매뉴얼 페이지가 있습니다. 관련 파일은 `/usr/share/doc/pam` 디렉토리에 저장됩니다.

자세한 내용은 `pam(8)` 매뉴얼 페이지를 참조하세요. 또한 각 PAM 모듈에는 `pam_unix(8)`, `postlogin(5)` 및 `system-auth(5)`와 같은 자체 매뉴얼 페이지가 있습니다.
