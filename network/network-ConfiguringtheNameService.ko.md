<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# 이름 서비스 구성

이 장에서는 Berkeley Internet Name Domain\(BIND\)을 사용하여 Domain Name System\(DNS\) 이름 서버를 설정하는 방법을 설명합니다.

## DNS 및 BIND

DNS는 도메인 이름을 IP 주소로 확인하는 네트워크 기반 서비스입니다. 소규모의 격리된 네트워크의 경우 `/etc/hosts` 파일의 항목을 사용하여 이름-주소 매핑을 제공할 수 있습니다. 그러나 인터넷에 연결된 대부분의 네트워크는 DNS를 사용합니다.

DNS는 계층적이고 분산된 데이터베이스입니다.

정규화된 도메인 이름 \(FQDN\) `wiki.us.mydom.com`을 고려하세요. 이 예에서 최상위 도메인은 `com`이고, `mydom`은 `com`의 하위 도메인이고, `us`는 `mydom`의 하위 도메인이며, `wiki`는 호스트 이름입니다.

이러한 각 도메인은 관리 목적으로 zone으로 그룹화됩니다. DNS 서버 또는 _네임 서버_는 zone 내부의 구성 요소 도메인을 확인하는 데 필요한 정보를 저장합니다. 또한 zone의 DNS 서버는 각 하위 도메인 확인을 담당하는 다른 DNS 서버에 대한 포인터를 저장합니다.

외부 클라이언트가 `wiki.us.mydom.com`과 같은 FQDN을 해당 서버가 신뢰할 수 없는 IP 주소로 확인하도록 로컬 이름 서버에 요청하는 경우 서버는 주소에 대해 `루트` 이름 서버를 쿼리합니다. 그런 다음 이 서버는 `mydom.com` 도메인에 대해 권한이 있는 다른 이름 서버의 IP 주소를 제공하고, 이 서버는 차례로 `us.mydom.com`에 대해 권한 있는 이름 서버의 IP 주소를 제공합니다.

쿼리 프로세스는 요청을 수행한 외부 클라이언트에 제공되는 FQDN의 IP 주소로 끝납니다. 이 프로세스를 재귀 쿼리라고 하며, 여기서 로컬 이름 서버는 확인자를 대신하여 외부 이름 서버에서 다른 이름 서버로의 각 참조를 처리합니다.

반복 쿼리는 FQDN에 대해 권한이 있는 이름 서버를 추적하기 위해 각 외부 이름 서버의 조회를 처리할 수 있는 확인자에 의존합니다. 대부분의 확인자는 재귀 쿼리를 사용하므로 반복 쿼리만 지원하는 이름 서버를 사용할 수 없습니다.

대부분의 Enterprise Linux 릴리스는 DNS의 BIND 구현을 제공합니다. `bind` 패키지에는 DNS 서버 데몬 \(`named`\), `rndc`와 같은 DNS 작업 도구 및 다음을 포함한 일부 구성 파일이 포함되어 있습니다.:

- **`/etc/named.conf`**

  `named`에 대한 설정을 포함하고 도메인에 대한 ZONE 파일의 위치와 특성을 나열합니다. ZONE 파일은 일반적으로 `/var/named`에 저장됩니다.

- **`/etc/named.rfc1912.zones`**

  로컬 루프백 이름과 주소를 확인하기 위한 여러 ZONE 섹션이 포함되어 있습니다.

- **`/var/named/named.ca`**

  루트 권한이 있는 DNS 서버 목록이 포함되어 있습니다.

## 네임서버 유형

BIND를 사용하여 다음을 포함하여 여러 유형의 이름 서버를 구성할 수 있습니다.:

- **마스터 네임서버**

  하나 이상의 도메인에 대해 권한이 있는 기본 \(마스터\) 이름 서버는 여러 데이터베이스 파일에 ZONE 데이터를 유지 관리하고 이 정보를 ZONE에 구성된 모든 백업 이름 서버에 정기적으로 전송할 수 있습니다. 조직은 다음 두 가지를 유지 관리할 수 있습니다. ZONE의 기본 이름 서버: 공개적으로 액세스할 수 있는 호스트 및 서비스에 대해 ZONE에 대한 제한된 정보를 제공하는 방화벽 외부의 기본 서버 하나와 내부 호스트 및 서비스의 세부 정보가 포함된 방화벽 내부의 숨겨진 또는 _스텔스_ 기본 서버입니다. ZONE 파일의 리소스 레코드 예: serial 값이 증가하는 경우 `named`에게 ZONE 파일을 다시 로드하라고 지시하는 카운터입니다. 예를 들어: 예를 들어:

- **보조 또는 백업 네임서버**

  기본 이름 서버에 대한 백업 역할을 하는 백업 이름 서버는 ZONE 데이터의 복사본을 유지 관리하며, 이 복사본은 기본 서버의 복사본에서 주기적으로 새로 고쳐집니다.

- **Stub 네임서버**

  ZONE의 기본 이름 서버는 하위 ZONE의 기본 및 백업 이름 서버에 대한 정보를 유지 관리하는 Stub 이름 서버로 구성될 수도 있습니다.

- **캐싱 전용 네임서버**

  클라이언트를 대신하여 쿼리를 수행하고 결과를 클라이언트에 반환한 후 응답을 캐시에 저장합니다. 이 서버는 어떤 도메인에 대해서도 권한이 없으며 기록되는 정보는 캐시된 쿼리 결과로 제한됩니다.

- **포워딩 네임서버**

  모든 쿼리를 다른 이름 서버로 전달하고 결과를 캐시하므로 로컬 처리, 외부 액세스 및 네트워크 트래픽이 줄어듭니다.

실제로 이름 서버는 복잡한 구성에서 이러한 여러 유형의 조합이 될 수 있습니다.

## 네임서버 설치 및 구성

기본적으로 BIND 설치를 통해 `/etc/named.conf` 파일과 여기에 포함된 파일에 제공된 구성 설정을 사용하여 캐싱 전용 이름 서버를 구성할 수 있습니다. 다음 절차에서는 기본 설정을 사용하거나 새로운 `named` 구성 및 ZONE 파일을 구성한다고 가정합니다.

이름 서버를 구성하려면:

1. Bind 패키지를 설치합니다.

   ```
   sudo dnf install bind
   ```

2. 시스템에서 `NetworkManager`가 활성화된 경우 `/etc/sysconfig/network-scripts/ifcfg-*interface*` 파일을 편집하고 다음 항목을 추가하십시오.:

   ```
   DNS1=127.0.0.1
   ```

   이 줄은 네트워크 서비스가 시작될 때 `NetworkManager`가 `/etc/resolv.conf`에 다음 항목을 추가하도록 합니다.:

   ```
   nameserver 127.0.0.1
   ```

   이 항목은 로컬 이름 서버의 확인자를 가리킵니다.

3. `NetworkManager`를 비활성화한 경우 `nameserver 127.0.0.1` 항목을 포함하도록 `/etc/resolv.conf` 파일을 편집하세요.

4. 필요한 경우 `named` 구성 및 ZONE 파일을 변경하세요.

   [named 데몬 구성](ko-network-ConfiguringtheNameService.md#named-데몬-구성)을 참조하세요.

5. 포트 53으로 들어오는 TCP 연결과 포트 53에서 들어오는 UDP 데이터를 허용하도록 시스템 방화벽을 구성합니다.:

   ```
   sudo firewall-cmd --zone=*zone* --add-port=53/tcp --add-port=53/udp
   ```

   ```
   sudo firewall-cmd --permanent --zone=*zone* --add-port=53/tcp --add-port=53/udp
   ```

6. `NetworkManager` 서비스와 `named` 서비스를 다시 시작한 다음 시스템 재부팅 후 시작되도록 `named` 서비스를 구성합니다.:

   ```
   sudo systemctl restart NetworkManager
   ```

   ```
   sudo systemctl start named
   ```

   ```
   sudo systemctl enable named
   ```

## DNS 구성 파일 작업

도메인은 ZONE 파일을 통해 구성된 ZONE으로 그룹화됩니다. ZONE 파일은 DNS 데이터베이스의 도메인에 대한 정보를 저장합니다. 각 ZONE 파일에는 지시문과 리소스 레코드가 포함되어 있습니다. 선택적 지시문은 ZONE에 설정을 적용하거나 이름 서버에 특정 작업을 수행하도록 지시합니다. 리소스 레코드는 ZONE 매개변수를 지정하고 ZONE의 시스템 또는 호스트에 대한 정보를 정의합니다.

BIND 구성 파일의 예는 `/usr/share/doc/bind/sample/etc` 파일에서 찾을 수 있습니다.

### named 데몬 구성

`named` 서비스의 기본 구성 파일은 `/etc/named.conf`입니다. 다음 예는 `bind` 패키지와 함께 설치되고 캐싱 전용 이름 서버를 구성하는 기본 `/etc/named.conf` 파일에서 가져온 것입니다.:

```
options {
    listen-on port 53 { 127.0.0.1; };
    listen-on-v6 port 53 { ::1; };
    directory       "/var/named";
    dump-file       "/var/named/data/cache_dump.db";
    statistics-file "/var/named/data/named_stats.txt";
    memstatistics-file "/var/named/data/named_mem_stats.txt";
    secroots-file   "/var/named/data/named.secroots";
    recursing-file  "/var/named/data/named.recursing";
    allow-query { localnets; };
    recursion yes;

    dnssec-enable yes;
    dnssec-validation yes;

    /* Path to ISC DLV key */
    bindkeys-file "/etc/named.iscdlv.key";

    managed-keys-directory "/var/named/dynamic";

﻿   pid-file "/run/named/named.pid";
    session-keyfile "/run/named/session.key";

    /* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
    include "/etc/crypto-policies/back-ends/bind.config";

};

logging {
    channel default_debug {
        file "data/named.run";
        severity dynamic;
    };
};

zone "." IN {
    type hint;
    file "named.ca";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

`option` 섹션은 전역 서버 구성 옵션을 정의하고 다른 섹션의 기본값을 설정합니다.

- **`listen-on`**

  `named`가 쿼리를 수신하는 포트입니다.

- **`directory`**

  상대 경로 이름이 지정된 경우 ZONE 파일의 기본 디렉터리를 지정합니다.

- **`dump-file`**

  `named`가 충돌할 경우 캐시를 덤프하는 위치를 지정합니다.

- **`statistics-file`**

  `rndc stats` 명령의 출력 파일을 지정합니다.

- **`memstatistics-file`**

  `named` 메모리 사용량 통계에 대한 출력 파일을 지정합니다.

- **`allow-query`**

  서버에 쿼리할 수 있는 IP 주소를 지정합니다. `localnets`는 로컬로 연결된 모든 네트워크를 지정합니다.

- **`recursion`**

  이름 서버가 재귀 쿼리를 수행하는지 여부를 지정합니다.

- **`dnssec-enable`**

  보안 DNS\(DNSSEC\)를 사용할지 여부를 지정합니다.

- **`dnssec-validation`**

  이름 서버가 DNSSEC 지원 ZONE의 응답을 검증할지 여부를 지정합니다.

- **`dnssec-lookaside`**

  `bindkeys-file`에 정의된 `/etc/named.iscdlv.key`의 키를 사용하여 DNSSEC Lookaside Validation\(DLV\)을 활성화할지 여부를 지정합니다.

`logging` 섹션은 `/var/named/data/named.run`에 대한 메시지 로깅을 활성화합니다. `severity` 매개변수는 로깅 수준을 제어하며, `dynamic` 값은 이 수준이 `rndc Trace` 명령을 사용하여 제어될 수 있음을 의미합니다.

`zone` 섹션은 hint zone을 사용하여 초기 루트 서버 세트를 지정합니다. 이 zone은 `named`가 루트 도메인 \(`.`\)에 대한 권한 있는 서버의 IP 주소에 대해 `/var/named/named.ca`를 참조하도록 지정합니다.

네트워크 환경에 적합한 구성 파일에 정의를 추가할 수 있습니다. 다음 예에서는 서비스에 대한 설정과 ZONE에 대한 최상위 정의를 정의합니다.:

```
include "/etc/rndc.key";

controls {
    inet 127.0.0.1 allow { localhost; } keys { "rndc-key"; }
};

zone "us.mydom.com" {
    type master;
    file "master-data";
    allow-update { key "rndc-key"; };
    notify yes;
};

zone "mydom.com" IN {
    type slave;
    file "sec/slave-data";
    allow-update { key "rndc-key"; };
    masters {10.1.32.1;};
};

zone "2.168.192.in-addr.arpa" IN {
    type master;
    file "reverse-192.168.2";
    allow-update { key “rndc-key”; };
    notify yes;
};
```

`include` 문을 사용하면 외부 파일을 참조할 수 있으므로 키 해시와 같은 민감한 데이터를 제한된 권한으로 별도의 파일에 배치할 수 있습니다.

`controls` 문은 `named` 서버에서 `rndc` 명령을 사용하는 데 필요한 액세스 정보와 보안 요구 사항을 정의합니다.:

- **`inet`**

  named 제어를 위해 `rndc`를 실행할 수 있는 호스트를 지정합니다. 이 예에서 `rndc`는 로컬 호스트 \(`127.0.0.1`\)에서 실행되어야 합니다.

- **`keys`**

  사용할 수 있는 키의 이름을 지정합니다. 이 예에서는 `/etc/rndc.key`에 정의된 `rndc-key`라는 키를 사용하여 지정합니다. 키는 `named`으로 다양한 작업을 인증하며 원격 액세스 및 관리를 제어하는 ​​기본 방법입니다.

`zone` 문은 다양한 ZONE에서 서버의 역할을 정의합니다.

다음 ZONE 옵션이 사용됩니다.:

- **`type`**

  이 시스템이 `us.mydom.com` ZONE의 기본 이름 서버이자 `mydom.com`의 백업 서버임을 지정합니다. `2.168.192.in-addr.arpa`는 IP 주소를 호스트 이름으로 확인하기 위한 역방향 ZONE입니다. [역방항 이름 확인을 위한 리소스 레코드](ko-network-ConfiguringtheNameService.md#역방항-이름-확인을-위한-리소스-레코드)를 참조하세요.

- **`file`**

  `/var/named`를 기준으로 ZONE 파일의 경로를 지정합니다. `us.mydom.com`에 대한 ZONE 파일은 `/var/named/master-data`에 저장되고 `mydom.com`에 대해 전송된 ZONE 데이터는 `/var/named/sec/slave-data`에 캐시됩니다.

- **`allow-update`**

  기본 이름 서버에서 백업 이름 서버로 ZONE 전송이 이루어지려면 기본 이름 서버와 백업 이름 서버 모두에 공유 키가 있어야 함을 지정합니다. 다음은 `/etc/rndc.key` 파일의 키에 대한 예제 레코드입니다.:

  ```
  key "rndc-key" {
      algorithm hmac-md5;
      secret "XQX8NmM41+RfbbSdcqOejg==";
  };
  ```

  `rndc-confgen -a` 명령을 사용하여 키 파일을 생성할 수 있습니다.

- **`notify`**

  ZONE 정보가 업데이트될 때 백업 네임서버에 알릴지 여부를 지정합니다.

- **`masters`**

  백업 이름 서버의 기본 이름 서버를 지정합니다.

자세한 내용은 `named.conf(5)` 매뉴얼 페이지와 `/usr/share/doc/bind-*version*/arm`의 BIND 설명서를 참조하세요.

### About Resource Records in Zone Files

ZONE 파일의 리소스 레코드에는 다음 필드가 포함되어 있으며, 그 중 일부는 레코드 유형에 따라 선택 사항입니다.:

- **Name**

  도메인 이름 또는 IP 주소.

- **TTL \(time to live\)**

  이름 서버가 최신 레코드를 사용할 수 있는지 확인하기 전에 레코드를 캐시하는 최대 시간입니다.

- **Class**

  인터넷의 경우 항상 `IN`입니다.

- **Type**

  레코드 유형입니다.

  - **`A` \(address\)**

    호스트에 해당하는 IPv4 주소입니다.

  - **`AAAA` \(address\)**

    호스트에 해당하는 IPv6 주소입니다.

  - **`CNAME` \(canonical name\)**

    호스트 이름에 해당하는 별칭 이름입니다.

  - **`MX` \(mail exchange\)**

    도메인으로 주소가 지정된 이메일의 대상입니다.

  - **`NS` \(name server\)**

    도메인에 대한 권한 있는 이름 서버의 정규화된 도메인 이름입니다.

  - **`PTR` \(pointer\)**

    주소-이름 조회\(역이름 확인\)를 위한 IP 주소에 해당하는 호스트 이름입니다.

  - **`SOA` \(start of authority\)**

    기본 이름 서버, 도메인 관리자의 이메일 주소, 도메인 일련번호 등 ZONE에 대한 신뢰할 수 있는 정보입니다. `SOA` 레코드 뒤에 오는 모든 레코드는 다음 `SOA` 레코드까지 정의하는 ZONE과 관련됩니다.

- **Data**

  `A` 레코드의 IP 주소, `CNAME` 또는 `PTR` 레코드의 호스트 이름 등 레코드에 저장되는 정보입니다.

다음 예는 `/var/named/master-data`와 같은 일반적인 ZONE 파일의 내용을 보여줍니다.:

```
$TTL 86400        ; 1 day
@ IN SOA dns.us.mydom.com. root.us.mydom.com. (
            57 ; serial
            28800 ; refresh (8 hours)
            7200 ; retry (2 hours)
            2419200 ; expire (4 weeks)
            86400 ; minimum (1 day)
            )
              IN  NS      dns.us.mydom.com.

dns           IN  A       192.168.2.1
us.mydom.com  IN  A       192.168.2.1
svr01         IN  A       192.168.2.2
www           IN  CNAME   svr01
host01        IN  A       192.168.2.101
host02        IN  A       192.168.2.102
host03        IN  A       192.168.2.103
...
```

한 줄의 주석 앞에는 세미콜론 \(`;`\)이 옵니다.

`$TTL` 지시문은 ZONE의 모든 리소스 레코드에 대한 기본 TTL(Time-To-Live) 값을 정의합니다. 각 리소스 레코드는 전역 설정을 재정의하는 자체 TTL(Time-To-Live) 값을 정의할 수 있습니다.

`SOA` 레코드는 필수이며 다음 정보를 포함합니다.:

- **`us.mydom.com`**

  도메인의 이름입니다.

- **`dns.us.mydom.com.`**

  루트 도메인의 후행 마침표 \(`.`\)를 포함하는 이름 서버의 정규화된 도메인 이름입니다.

- **`root.us.mydom.com.`**

  도메인 관리자의 이메일 주소입니다.

- **_serial_**

  A counter that, if incremented, tells `named` to reload the zone file.

- **_refresh_**

  기본 이름 서버가 백업 이름 서버에 데이터베이스를 새로 고쳐야 함을 알리는 시간입니다.

- **_retry_**

  새로 고침이 실패하는 경우 백업 이름 서버가 다른 새로 고침을 시도하기 전에 기다려야 하는 시간입니다.

- **_expire_**

  ZONE 레코드가 더 이상 신뢰할 수 있는 것으로 간주되지 않고 쿼리 응답을 중지하기 전에 백업 이름 서버가 새로 고침을 완료해야 하는 최대 경과 시간입니다.

- **_minimum_**

  다른 서버가 이 ZONE에서 얻은 정보를 캐시해야 하는 최소 시간입니다.

`NS` 레코드는 도메인에 대한 권한 있는 이름 서버를 선언합니다.

각 `A` 레코드는 도메인의 호스트 이름에 해당하는 IP 주소를 지정합니다.

`CNAME` 레코드는 `svr01`에 대한 별칭 `www`를 생성합니다.

자세한 내용은 `/usr/share/doc/bind-*version*/arm`의 BIND 설명서를 참조하세요.

### 역방항 이름 확인을 위한 리소스 레코드

정방향 확인은 지정된 도메인 이름에 대한 IP 주소를 반환합니다. 역방향 이름 확인은 지정된 IP 주소에 대한 도메인 이름을 반환합니다. DNS는 IPv4 및 IPv6용 특수 `in-addr.arpa` 및 `ip6.arpa` 도메인을 사용하여 역방향 이름 확인을 구현합니다.

ZONE의 `in-addr.arpa` 또는 `ip6.arpa` 도메인의 특성은 일반적으로 `/etc/named.conf`에 정의됩니다.

```
zone "2.168.192.in-addr.arpa" IN {
    type master;
    file "reverse-192.168.2";
    allow-update { key “rndc-key”; };
    notify yes;
};
```

ZONE의 이름은 `in-addr.arpa`로 구성되며 그 앞에는 도메인 IP 주소의 네트워크 부분이 오고 점으로 구분된 쿼드는 역순으로 작성됩니다.

네트워크에 8의 배수인 접두사 길이가 없는 경우 대신 사용해야 하는 형식은 [RFC 2317](https://datatracker.ietf.org/doc/html/rfc2317)을 참조하세요.

`in-addr.arpa` 또는 `ip6.arpa` 도메인의 `PTR` 레코드는 IP 주소의 호스트 부분에 해당하는 호스트 이름을 정의합니다. 다음 예는 `/var/named/reverse-192.168.2` ZONE 파일에서 가져온 것입니다.:

```
$TTL 86400        ;
@ IN SOA dns.us.mydom.com. root.us.mydom.com. (
            57 ;
            28800 ;
            7200 ;
            2419200 ;
            86400 ;
            )
              IN  NS      dns.us.mydom.com.

1             IN  PTR     dns.us.mydom.com.
1             IN  PTR     us.mydom.com.
2             IN  PTR     svr01.us.mydom.com.
101           IN  PTR     host01.us.mydom.com.
102           IN  PTR     host02.us.mydom.com.
103           IN  PTR     host03.us.mydom.com.
...
```

자세한 내용은 `/usr/share/doc/bind-*version*/arm`의 BIND 설명서를 참조하세요.

## 이름 서비스 관리

`rndc` 명령을 사용하면 `named` 서비스를 관리할 수 있습니다. 서비스는 로컬에서 관리됩니다. 서비스가 `/etc/named.conf` 파일의 `controls` 섹션에 구성된 경우 커맨드라인을 사용하여 `named`를 원격으로 관리할 수도 있습니다. 서비스에 대한 무단 액세스를 방지하려면 `rndc`가 선택한 포트(기본적으로 포트 953\)를 수신하도록 구성해야 하며 named 포트와 `rndc` 모두 동일한 키에 액세스할 수 있어야 합니다. 적합한 키를 생성하려면 `rndc-confgen` 명령을 사용하세요.:

```
sudo rndc-confgen -a
```

이 명령은 `/etc/rndc.key` 파일을 생성합니다.

다음과 같이 `named` 서비스의 상태를 확인하세요.:

```
sudo rndc status
```

```nocopybutton
number of zones: 3
debug level: 0
xfers running: 0
xfers deferred: 0
soa queries in progress: 0
query logging is OFF
recursive clients: 0/1000
tcp clients: 0/100
server is up and running
```

`named` 구성 파일이나 ZONE 파일을 변경하는 경우 `rndc reload` 명령은 `named`에 파일을 다시 로드하도록 지시합니다.:

```
sudo rndc reload
```

자세한 내용은 `named(8)`, `rndc(8)` 및 `rndc-confgen(8)` 매뉴얼 페이지를 참조하세요.

## DNS 조회 수행

DNS 조회를 수행하려면 `host` 유틸리티를 사용하는 것이 좋습니다. 인수가 없으면 명령은 커맨드라인 인수 및 옵션의 요약을 표시합니다.

예를 들어 `host01`에 대한 IP 주소를 검색해 보세요.:

```
host host01
```

IP 주소에 해당하는 도메인 이름에 대해 역방향 조회를 수행합니다.:

```
sudo host 192.168.2.101
```

도메인에 해당하는 IP 주소에 대한 DNS 쿼리:

```
sudo host dns.us.mydoc.com
```

특정 유형의 레코드에 대한 자세한 정보를 표시하려면 `-v` 및 `-t` 옵션을 사용하십시오.:

```
sudo host -v -t MX  www.mydom.com
```

```nocopybutton
Trying "www.mydom.com"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 49643
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 0

;; QUESTION SECTION:
;www.mydom.com.			IN	MX

;; ANSWER SECTION:
www.mydom.com.		135	IN	CNAME	www.mydom.com.acme.net.
www.mydom.com.acme.net. 1240 IN	CNAME	d4077.c.miscacme.net.

;; AUTHORITY SECTION:
c.miscacme.net.	2000	IN	SOA	m0e.miscacme.net. hostmaster.misc.com. ...

Received 163 bytes from 10.0.0.1#53 in 40 ms
```

`-v`, `-t` 및 `ANY` 옵션과 동일한 `-a` 옵션은 ZONE에 사용 가능한 모든 레코드를 표시합니다.

```
sudo host -a www.us.mydom.com
```

```nocopybutton
Trying "www.us.mydom.com"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 40030
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;www.us.mydom.com.			IN	ANY

;; ANSWER SECTION:
www.us.mydom.com.		263	IN	CNAME	www.us.mydom.acme.net.

Received 72 bytes from 10.0.0.1#53 in 32 ms
```

자세한 내용은 `host(1)` 매뉴얼 페이지를 참조하세요.
