<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# DHCP 서비스 구성

동적 호스트 구성 프로토콜\(DHCP\)을 사용하면 클라이언트 시스템이 네트워크에 연결할 때마다 DHCP 서버에서 네트워크 구성 정보를 얻을 수 있습니다. DHCP 서버는 클라이언트에 필요한 다양한 IP 주소와 기타 네트워크 구성 매개변수로 구성됩니다.

Enterprise Linux 시스템을 DHCP 클라이언트로 구성하면 클라이언트 데몬 `dhclient`가 DHCP 서버에 연결하여 네트워킹 매개변수를 얻습니다. DHCP는 브로드캐스트 기반이므로 클라이언트는 서버 또는 릴레이 에이전트와 동일한 서브넷에 있어야 합니다. 클라이언트가 서버와 동일한 서브넷에 있을 수 없는 경우 DHCP 릴레이 에이전트를 사용하여 서브넷 간에 DHCP 메시지를 전달할 수 있습니다.

서버는 클라이언트에 할당하는 IP 주소에 대한 임대를 제공합니다. 클라이언트는 기간과 같은 임대에 대한 특정 조건을 요청할 수 있습니다. 임대에 부여할 수 있는 조건을 제한하도록 DHCP 서버를 구성할 수 있습니다. 클라이언트가 네트워크에 계속 연결되어 있으면 `dhclient`는 임대가 만료되기 전에 자동으로 갱신합니다. 네트워크 인터페이스의 MAC 주소를 기반으로 클라이언트에 동일한 IP 주소를 제공하도록 DHCP 서버를 구성할 수 있습니다.

DHCP를 사용하면 다음과 같은 이점이 있습니다.:

- IP 주소의 중앙 집중식 관리

- 네트워크에 새 클라이언트를 쉽게 추가할 수 있음

- IP 주소 재사용으로 필요한 총 IP 주소 수 감소

- 각 클라이언트를 재구성할 필요 없이 DHCP 서버에서 IP 주소 공간 재구성

DHCP에 대한 자세한 내용은 [RFC 2131](https://datatracker.ietf.org/doc/html/rfc2131)을 참조하세요. 마찬가지로 다음 매뉴얼 페이지를 참조하십시오.:

- `dhcpd(8)`
- `dhcp-options(5)`

## 서버의 네트워크 인터페이스 설정

기본적으로 `dhcpd` 서비스는 DHCP 구성 파일에 정의된 서브넷에 연결하는 네트워크 인터페이스에 대한 요청을 처리합니다.

DHCP 서버에 여러 인터페이스가 있다고 가정합니다. `enp0s1` 인터페이스를 통해 서버는 서버가 서비스를 제공하도록 구성된 클라이언트와 동일한 서브넷에 연결됩니다. 이 경우 서버가 해당 인터페이스에서 들어오는 요청을 모니터링하고 처리할 수 있도록 DHCP 서비스에 `enp0s1`을 설정해야 합니다.

다음 절차 중 하나를 진행하기 전에 다음 요구 사항을 충족하는지 확인하십시오.

- DHCP를 구성할 수 있는 적절한 관리 권한이 있습니다.
- `dhcp-server` 패키지를 설치했습니다.

  그렇지 않은 경우 다음 명령을 사용하여 패키지를 설치하십시오.:

  ```
  sudo dnf install dhcp-server
  ```

다음과 같이 네트워크 인터페이스를 구성합니다.:

- IPv4 네트워크의 경우:
  1. `/usr/lib/systemd/system/dhcpd.service` 파일을 `/etc/systemd/system/` 디렉토리에 복사합니다.

     ```
     sudo cp /usr/lib/systemd/system/dhcpd.service /etc/systemd/system/
     ```

  2. `ExecStart` 매개변수를 정의하는 줄을 찾아 `/etc/systemd/system/dhcpd.service`를 편집합니다.

  3. `dhcpd` 서비스가 수신해야 하는 인터페이스 이름을 추가합니다.

     굵게 표시된 샘플 항목을 참조하세요.

     ```
     ExecStart=/usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid $DHCPDARGS **_int1-name_** **_int2-name_**
     ```

  4. `systemd` 관리자 구성을 다시 로드하세요.

     ```
     sudo systemctl daemon-reload
     ```

  5. `dhcpd` 서비스 구성을 다시 시작하세요.

     ```
     sudo systemctl restart dhcpd.service
     ```

     또는 다음을 입력할 수도 있습니다.:

     ```
     sudo systemctl restart dhcpd
     ```

- IPv6 네트워크의 경우:
  1. `/usr/lib/systemd/system/dhcpd6.service` 파일을 `/etc/systemd/system/` 디렉터리에 복사합니다.

     ```
     sudo cp /usr/lib/systemd/system/dhcpd6.service /etc/systemd/system/
     ```

  2. `ExecStart` 매개변수를 정의하는 줄을 찾아 `/etc/systemd/system/dhcpd6.service` 파일을 편집합니다.

  3. `dhcpd6` 서비스가 수신해야 하는 인터페이스의 이름을 추가합니다.

     굵게 표시된 샘플 항목을 참조하세요.

     ```
     ExecStart=/usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd6.conf -user dhcpd -group dhcpd --no-pid $DHCPDARGS **_int1-name_** **_int2-name_**
     ```

  4. `systemd` 관리자 구성을 다시 로드하세요.

     ```
     sudo systemctl daemon-reload
     ```

  5. `dhcpd` 서비스 구성을 다시 시작하세요.

     ```
     sudo systemctl restart dhcpd6.service
     ```

     또는 다음을 입력할 수도 있습니다.

     ```
     sudo systemctl restart dhcpd6
     ```

## DHCP 선언 이해

DHCP가 클라이언트에 서비스를 제공하는 방식은 IPv4 네트워크의 경우 `/etc/dhcp/dhcpd.conf` 파일, IPv6 네트워크의 경우 `/etc/dhcp/dhcpd6.conf` 파일의 매개변수 및 선언을 통해 정의됩니다. 파일에는 클라이언트 네트워크, 주소 임대, IP 주소 풀 등과 같은 세부 정보가 포함됩니다.

**참고:** 새로 설치된 Enterprise Linux 시스템에서는 `dhcpd.conf` 및 `dhcpd6.conf` 파일이 모두 비어 있습니다. 서버가 처음으로 DHCP에 대해 구성되는 경우 템플릿을 사용하여 파일 구성에 대한 안내를 받을 수 있습니다. 다음 명령 중 하나를 입력하십시오.:

- IPv4의 경우

  ```
  cp /usr/share/doc/dhcp-server/dhcpd.conf.example /etc/dhcp/dhcpd.conf
  ```

- IPv6의 경우

  ```
  cp /usr/share/doc/dhcp-server/dhcpd6.conf.example /etc/dhcp/dhcpd6.conf
  ```

그런 다음 두 파일 중 하나를 열면 참조를 위해 예제와 설명을 사용할 수 있습니다.

구성 파일의 정보는 다음 선언의 조합으로 구성됩니다.:

- [전역 설정](ko-network-ConfiguringDHCPServices.md#전역-설정)
- [서브넷 선언](ko-network-ConfiguringDHCPServices.md#서브넷-선언)
- [공유 네트워크 선언](ko-network-ConfiguringDHCPServices.md#공유-네트워크-선언)
- [호스트 선언](ko-network-ConfiguringDHCPServices.md#호스트-선언)
- [그룹 선언](ko-network-ConfiguringDHCPServices.md#그룹-선언)

### 전역 설정

전역 매개변수는 DHCP 서버가 지원하거나 서비스하는 모든 네트워크에 적용되는 설정을 정의합니다.

전체 네트워크에 전역적으로 적용되는 다음 설정을 고려하십시오.

- 회사 네트워크의 도메인 이름: `example.com`.d
- 네트워크의 DNS 서버: `dn1.example.com` 및 `dns2.example.com`
- 모든 클라이언트에게 할당된 임대 시간: 12시간\(43200초\)
- 할당 가능한 최대 임대 시간: 24시간\(86400초\)

이 경우 구성 파일에서 전역 설정을 다음과 같이 구성합니다.:

```
option domain-name "example.com";
default-lease-time 43200;
max-lease-time 86400;

authoritative;
```

`authoritative` 매개변수는 서버를 DHCP 서비스의 공식 서버 또는 기본 서버로 식별합니다. 이 매개변수는 일반적으로 여러 DHCP 서버가 있는 설정에서 사용됩니다. `authoritative` 매개변수가 있는 서버는 매개변수가 없는 서버보다 요청을 처리하는 데 우선순위를 갖습니다.

### 서브넷 선언

'서브넷' 선언은 DHCP 서버가 직접 연결된 서브넷과 해당 서브넷의 시스템이 클라이언트 역할을 하는 위치에 대한 세부 정보를 제공합니다.

DHCP 서버의 다음 구성을 고려하십시오.

- 서버의 `enp0s1` 인터페이스는 192.0.2.0/24 네트워크에 직접 연결됩니다.
- 192.0.2.0/24 네트워크의 시스템은 DHCP 클라이언트입니다.
- 이 클라이언트 서브넷의 토폴로지는 다음과 같습니다.
  - 서브넷의 DNS 서버: 192.0.2.1.
  - 서브넷 게이트웨이: 192.0.2.1.
  - 방송주소 : 192.0.2.255.
  - 클라이언트 주소 범위: 192.0.2.10 ~ 192.0.2.100.
  - 각 클라이언트의 최대 임대 시간: 86,400초\(1일\).

이 경우 `dhcp.conf`에 다음 선언을 입력합니다.:

```

subnet 192.0.2.0 netmask 255.255.255.0 {
  range 192.0.2.10 192.0.2.100;
  option domain-name-servers 192.0.2.1;
  option routers 192.0.2.1;
  option broadcast-address 192.0.2.255;
  max-lease-time 86400;
}
```

IPv6 네트워크 환경에서 `dhcpd6.conf` 파일의 `서브넷` 선언은 다음 예와 유사합니다.:

```
subnet6 2001:db8:0:1::/64 {
  range6 2001:db8:0:1::20 2001:db8:0:1::100;
  option dhcp6.name-servers 2001:db8:0:1::1;
  max-lease-time 172800;
}
```

### 공유 네트워크 선언

DHCP 서버가 서버에 직접 연결되지 않은 다른 서브넷의 클라이언트에 서버를 제공해야 하는 경우 '공유 네트워크' 선언을 정의합니다.

확장되었지만 이전 섹션의 시나리오와 약간 다른 다음 예제를 고려하십시오.:

- DHCP 서버는 192.0.2.0/24 네트워크에 속하지만 이 네트워크의 시스템에 서비스를 제공하지 않습니다.
- 서버는 다음 원격 서브넷에 있는 클라이언트의 요청을 처리합니다.:
  - 192.168.5.0/24.
  - 198.51.100.0/24.
- 원격 서브넷은 동일한 DNS 서버를 공유하지만 각 서브넷에는 자체 라우터와 IP 주소 범위가 있습니다.

이 경우 `dhcp.conf`에 다음 선언을 입력합니다.:

```
shared-network example {
 
  option domain-name-servers 192.168.2.1;
  ...

  subnet 192.168.5.0 netmask 255.255.255.0 {
    range 192.168.5.10 192.168.5.100;
    option routers 192.168.5.1;
  }

  subnet 198.51.100.0 netmask 255.255.255.0 {
    range 198.51.100.10 198.51.100.100;
    option routers 198.51.100.1;
  }
  ...
}

subnet 192.0.2.0 netmask 255.255.255.0 {
}

```

앞의 예에서 마지막 `서브넷` 선언은 서버 자체 네트워크를 참조하며 `공유 네트워크` 범위 외부에 있습니다. 선언은 서버의 서브넷을 정의하므로 빈 선언이라고 합니다. 서버는 이 서브넷에 서비스를 제공하지 않으므로 임대, 주소 범위, DNS 정보 등과 같은 추가 항목이 추가되지 않습니다. 그렇지 않으면 `dhcpd` 서비스가 시작되지 않습니다.

IPv6 네트워크 환경에서 `dhcpd6.conf` 파일의 `shared-network` 선언은 다음 예와 유사합니다.:

```
shared-network example {
  option domain-name-servers 2001:db8:0:1::1:1
  ...

  subnet6 2001:db8:0:1::1:0/120 {
    range6 2001:db8:0:1::1:20 2001:db8:0:1::1:100
  }

  subnet6 2001:db8:0:1::2:0/120 {
    range6 2001:db8:0:1::2:20 2001:db8:0:1::2:100
  }
  ...
}

subnet6 2001:db8:0:1::50:0/120 {
}
```

### 호스트 선언

클라이언트가 고정 IP 주소를 가져야 하는 경우 `host` 선언을 정의합니다.

서버의 192.0.2.0/24 네트워크에 있는 클라이언트 프린터의 다음 예를 고려하십시오. 이번에는 서버가 서브넷에 DHCP 서비스를 제공합니다.

- 프린터의 MAC 주소: 52:54:00:72:2f:6e.
- 프린터의 IP 주소: 192.0.2.130

  **중요:** 클라이언트의 고정 IP 주소는 다른 클라이언트에 배포된 동적 IP 주소 풀 외부에 있어야 합니다. 그렇지 않으면 주소 충돌이 발생할 수 있습니다.

이 경우 `dhcp.conf`에 다음 선언을 입력합니다.:

```
host printer.example.com {
	hardware ethernet 52:54:00:72:2f:6e;
	fixed-address 192.0.2.130; 
}
```

시스템은 '호스트' 선언의 이름이 아닌 하드웨어 이더넷 주소로 식별됩니다. 따라서 호스트 이름은 변경될 수 있지만 클라이언트는 이더넷 주소를 통해 계속해서 서비스를 받습니다.

IPv6 네트워크 환경에서 `dhcpd6.conf` 파일의 `host` 선언은 다음 예와 유사합니다.:

```
host server.example.com {
	hardware ethernet 52:54:00:72:2f:6e;
	fixed-address6 2001:db8:0:1::200;
}
```

### 그룹 선언

여러 공유 네트워크, 서브넷 및 호스트에 동일한 매개변수를 동시에 적용하려면 '그룹' 선언을 정의합니다.

다음 예를 고려하십시오.

- DHCP 서버는 서브넷 192.0.2.0/24에 속하고 서비스를 제공합니다.
- 한 클라이언트에는 고정 주소가 필요하고 나머지 클라이언트는 서버의 동적 IP 주소를 사용합니다.
- 모든 클라이언트는 동일한 DNS 서버를 사용합니다.

이 경우 `dhcp.conf`에 다음 선언을 입력합니다.:

```

group {
  option domain-name-servers 192.0.2.1;

  host server1.example.com {
    hardware ethernet 52:54:00:72:2f:6e;
    fixed-address 192.0.2.130;
  }

  subnet 192.0.2.0 netmask 255.255.255.0 {
  range 192.0.2.10 192.0.2.100;
  option routers 192.0.2.1;
  option broadcast-address 192.0.2.255;
  max-lease-time 86400;
  }
}
```

IPv6 네트워크 환경에서 `dhcpd6.conf` 파일의 `group` 선언은 다음 예와 유사합니다.:

```
group {
  option dhcp6.domain-search "example.com";

  host server1.example.com {
    hardware ethernet 52:54:00:72:2f:6e;
    fixed-address 2001:db8:0:1::200;
  }

  host server2.example.com {
    hardware ethernet 52:54:00:1b:f3:cf;
    fixed-address 2001:db8:0:1::ba3;
  }
}

subnet6 2001:db8:0:1::/64 {
  range6 2001:db8:0:1::20 2001:db8:0:1::100;
  option dhcp6.name-servers 2001:db8:0:1::1;
  max-lease-time 172800;
}
```

## DHCP 서비스 활성화

모든 DHCP 서비스는 서버의 `/etc/dhcp/dhcpd.conf` 또는 `/etc/dhcp/dhcpd6.conf` 파일에 정의되어 있습니다. 구성된 서비스를 구성한 후 활성화하려면 다음 단계를 수행하십시오.

- IPv4 네트워크의 경우:
  1. `/etc/dhcp/dhcpd.conf` 파일을 엽니다.

  2. 파일에 매개변수와 선언을 추가합니다.

     지침은 [DHCP 선언 이해](ko-network-ConfiguringDHCPServices.md#dhcp-선언-이해)를 참조하세요.

  3. 선택적으로 'dhcpd' 서비스가 서버 재부팅 시 자동으로 시작되도록 설정하세요.

     ```
     sudo systemctl enable dhcpd
     ```

  4. `dhcpd` 서비스를 시작하거나 다시 시작하세요.

     ```
     sudo systemctl start dhcpd
     ```

- IPv6 네트워크의 경우:
  1. `/etc/dhcp/dhcpd6.conf` 파일을 엽니다.

  2. 파일에 매개변수와 선언을 추가합니다.

     지침은 [DHCP 선언 이해](ko-network-ConfiguringDHCPServices.md#dhcp-선언-이해) 또는 `/usr/share/doc/dhcp-server/dhcpd6.conf.example` 템플릿의 설명 및 메모를 참조하세요.

  3. 선택적으로 서버 재부팅 시 `dhcpd6` 서비스가 자동으로 시작되도록 설정합니다.

     ```
     sudo systemctl enable dhcpd6
     ```

  4. `dhcpd` 서비스를 시작하거나 다시 시작하세요.

     ```
     sudo systemctl start dhcpd6
     ```

## 손상된 임대 데이터베이스에서 복구

`dhcpd` 서비스는 다음 플랫 파일 데이터베이스에 IP 주소, MAC 주소, 임대 만료 시간과 같은 임대 정보를 유지합니다.:

- DHCPv4의 경우: `/var/lib/dhcpd/dhcpd.leases`.
- DHCPv6의 경우: `/var/lib/dhcpd/dhcpd6.leases`.

오래된 데이터로 인해 임대 데이터베이스 파일이 너무 커지는 것을 방지하기 위해 'dhcpd' 서비스는 다음 메커니즘을 통해 주기적으로 파일을 재생성합니다.

1. 서비스는 기존 임대 파일의 이름을 바꿉니다.
   - `/var/lib/dhcpd/dhcpd.leases`는 `/var/lib/dhcpd/dhcpd.leases~`로 이름이 변경되었습니다.
   - `/var/lib/dhcpd/dhcpd6.leases`는 `/var/lib/dhcpd/dhcpd6.leases~`로 이름이 변경되었습니다.
2. 이 서비스는 새로운 `dhcpd.leases` 및 `dhcpd6.leases` 파일을 다시 생성합니다.

임대 데이터베이스 파일이 손상된 경우 마지막으로 알려진 데이터베이스 백업에서 임대 데이터베이스를 복원해야 합니다.

일반적으로 임대 데이터베이스의 가장 최근 백업은 `*filename*.leases~` 파일입니다.

**Note:** 백업 인스턴스는 특정 시점에 생성된 스냅샷이므로 시스템의 최신 상태를 반영하지 않을 수 있습니다.

필요한 관리 권한이 있는지 확인하고 다음 단계를 완료하세요.:

- DHCPv4의 경우
  1. `dhcpd` 서비스 중지:

     ```
     sudo systemctl stop dhcpd
     ```

  2. 손상된 임대 데이터베이스 이름 바꾸기:

     ```
     sudo mv /var/lib/dhcpd/dhcpd.leases /var/lib/dhcpd/dhcpd.leases.corrupt
     ```

  3. 해당 `*filename*.leases~` 백업 파일에서 임대 데이터베이스를 복원합니다.

     ```
     sudo cp -p /var/lib/dhcpd/dhcpd.leases~ /var/lib/dhcpd/dhcpd.leases
     ```

  4. `dhcpd` 서비스를 시작하세요:

     ```
     sudo systemctl start dhcpd
     ```

- DHCPv6의 경우
  1. `dhcpd` 서비스 중지:

     ```
     sudo systemctl stop dhcpd6
     ```

  2. 손상된 임대 데이터베이스 이름 바꾸기:

     ```
     sudo mv /var/lib/dhcpd/dhcpd6.leases /var/lib/dhcpd/dhcpd6.leases.corrupt
     ```

  3. 해당 `*filename*.leases~` 백업 파일에서 임대 데이터베이스를 복원합니다.

     ```
     sudo cp -p /var/lib/dhcpd/dhcpd6.leases~ /var/lib/dhcpd/dhcpd6.leases
     ```

  4. `dhcpd6` 서비스를 시작하세요:

     ```
     sudo systemctl start dhcpd6
     ```
