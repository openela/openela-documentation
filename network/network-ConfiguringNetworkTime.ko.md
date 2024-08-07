<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# 네트워크 시간 구성

이 장에서는 `ntp` 대신 네트워크 시간 프로토콜\(NTP\) 기능의 구현으로 `chrony`를 사용하도록 시스템을 구성하는 방법을 설명합니다. 또한 이 장에서는 시스템 시간을 설정하는 데 사용되는 Precision Time Protocol\(PTP\) 데몬에 대해 설명합니다.

## Chrony Suite

`chrony`는 네트워크에서 시간을 정확하게 유지하기 위해 NTP를 구현하는 기능입니다. Enterprise Linux 8에서는 `chrony` 데몬 서비스가 NTP 관리를 위해 `ntpd`를 대체합니다.

`chrony` has two components, which are provided in the `chrony` package:

- `chronyd` service daemon

- `chornyc` service utility

### chronyd 서비스 데몬

`chronyd` 서비스 데몬은 일정 기간 동안 네트워크가 중단되거나 연결이 끊어진 후 모바일 시스템과 가상 머신의 시스템 시계를 업데이트합니다. 이 서비스는 기본 NTP 클라이언트 또는 NTP 서버를 구현하는 데에도 사용할 수 있습니다. NTP 서버로서 `chronyd`는 상위 계층 Stratum NTP 서버와 동기화하거나 GPS(Global Positioning System)\(GPS\) 또는 DCF77, MSF 또는 WWVB와 같은 무선 방송에서 수신된 시간 신호를 사용하여 Stratum 1 서버로 작동할 수 있습니다.

Enterprise Linux 시스템에서는 이 서비스 데몬이 기본적으로 활성화됩니다.

**Note:**

`chronyd`는 NTP 버전 4와 호환되는 기능을 갖춘 NTP 버전 3\([RFC 1305](https://datatracker.ietf.org/doc/html/rfc1305)\)을 사용합니다\([RFC 5905](https //datatracker.ietf.org/doc/html/rfc5905)\). 그러나 `chronyd`는 NTP 버전 4의 몇 가지 중요한 기능을 지원하지 않으며 PTP 사용도 지원하지 않습니다.

자세한 내용은 `chrony(1)` 매뉴얼 페이지와 `/usr/share/doc/chrony/` 디렉토리에 있는 파일을 참조하세요.

### chronyc 서비스 유틸리티 사용

`chronyc` 유틸리티는 `chronyd` 서비스를 관리하고, 서비스 작동에 대한 정보를 표시하거나, 서비스 구성을 변경하는 도구입니다.

이 명령은 두 가지 모드로 작동합니다.:

- 비대화형 모드: 이 모드에서는 다음 구문을 사용합니다.:

  ```
  sudo chronyc *subcommand*
  ```

- 대화형 모드: 명령만 입력하면 대화형 모드가 활성화되고 `chronyc>` 프롬프트가 표시됩니다. 이 프롬프트에서 `chronyc` 하위 명령을 실행할 수 있습니다.

  ```
  sudo chronyc
  ```

  ```nocopybutton
  chronyc>
  ```

  프롬프트에서 필요에 따라 다양한 `chronyc` 하위 명령을 실행할 수 있습니다. 다음 예는 `sources` 및 `sourcestats` 하위 명령으로 생성된 정보를 보여줍니다.:

  ```
  chronyc> sources
  ```

  ```nocopybutton
  210 Number of sources = 4
  MS Name/IP address         Stratum Poll Reach LastRx Last sample
  ===============================================================================
  ^+ service1-eth3.debrecen.hp     2   6    37    21  -2117us[-2302us] +/-   50ms
  ^* ns2.telecom.lt                2   6    37    21   -811us[ -997us] +/-   40ms
  ^+ strato-ssd.vpn0.de            2   6    37    21   +408us[ +223us] +/-   78ms
  ^+ kvm1.websters-computers.c     2   6    37    22  +2139us[+1956us] +/-   54ms
  ```

  ```
  chronyc> sourcestats
  ```

  ```nocopybutton
  210 Number of sources = 4
  Name/IP Address            NP  NR  Span  Frequency  Freq Skew  Offset  Std Dev
  ==============================================================================
  service1-eth3.debrecen.hp   5   4   259     -0.394     41.803  -2706us   502us
  ns2.telecom.lt              5   4   260     -3.948     61.422   +822us   813us
  strato-ssd.vpn0.de          5   3   259      1.609     68.932   -581us   801us
  kvm1.websters-computers.c   5   5   258     -0.263      9.586  +2008us   118us
  chronyc> `tracking`
  Reference ID    : 212.59.0.2 (ns2.telecom.lt)
  Stratum         : 3
  Ref time (UTC)  : Tue Sep 30 12:33:16 2014
  System time     : 0.000354079 seconds slow of NTP time
  Last offset     : -0.000186183 seconds
  RMS offset      : 0.000186183 seconds
  Frequency       : 28.734 ppm slow
  Residual freq   : -0.489 ppm
  Skew            : 11.013 ppm
  Root delay      : 0.065965 seconds
  Root dispersion : 0.007010 seconds
  Update interval : 64.4 seconds
  Leap status     : Normal
  ```

  대화형 모드 사용을 종료하려면 `exit`를 입력하세요.

**Note:**

`chronyc` 명령으로 구현한 모든 변경 사항은 다음에 `chronyd` 데몬을 다시 시작할 때까지만 유효합니다. 변경 사항을 영구적으로 적용하려면 `/etc/chrony.conf` 파일에 이를 입력해야 합니다. [chronyd 구성 파일 편집](ko-network-ConfiguringNetworkTime.md#chronyd-구성-파일-편집)을 참조하세요.

자세한 내용은 `chronyc(1)` 매뉴얼 페이지와 `/usr/share/doc/chrony/` 디렉토리에 있는 파일을 참조하세요.

### chronyd 서비스 구성

시스템에서 `chronyd` 서비스를 구성하려면:

1. `chrony` 패키지를 설치합니다.

   ```
   sudo dnf install chrony
   ```

2. 로컬 NTP 서비스에 대한 원격 액세스가 필요한 경우 적절한 영역에서 NTP 서비스에 대한 액세스를 허용하도록 시스템 방화벽을 구성하십시오.

   ```
   sudo firewall-cmd --zone=*zone* --add-service=ntp
   ```

   ```
   sudo firewall-cmd --zone=*zone* --permanent --add-service=ntp
   ```

3. `chronyd` 서비스를 시작하고 시스템 재부팅 후 시작되도록 구성합니다.

   기본적으로 `chrony`는 설치 후 활성화됩니다.

   ```
   sudo systemctl start chronyd
   ```

   ```
   sudo systemctl enable chronyd
   ```

### chronyd 구성 파일 편집

`/etc/chrony.conf` 파일에서 기본 구성은 시스템이 동기화할 수 있는 공용 NTP 서버에 대한 네트워크 액세스를 가지고 있다고 가정합니다.

다음 예에서는 세 개의 NTP 서버에 액세스하도록 시스템을 구성합니다.:

```
pool *NTP\_server\_1*
pool *NTP\_server\_2*
pool *NTP\_server\_3*
driftfile /var/lib/chrony/drift
keyfile /etc/chrony.keys
...
```

지정된 클라이언트 또는 서브넷에 대해 NTP 서버 역할을 하도록 `chronyd`를 구성하려면 다음 예에서 굵게 표시된 대로 `allow` 지시어를 사용하세요.:

```
pool *NTP\_server\_1*
pool *NTP\_server\_2*
pool *NTP\_server\_3*
**allow 192.168.2/24**
driftfile /var/lib/chrony/drift
keyfile /etc/chrony.keys
...
```

공개 키 암호화를 기반으로 인증 메커니즘을 위한 키를 생성하려면 `chronyc keygen` 명령을 사용하세요.

**Note:**

`ntp`의 `Autokey`는 더 이상 `chrony`에서 작동하지 않습니다.

시스템이 NTP 서버에 간헐적으로만 액세스하는 경우 다음 구성이 적절할 수 있습니다.:

```
pool *NTP\_server\_1* offline
pool *NTP\_server\_2* offline
pool *NTP\_server\_3* offline
driftfile /var/lib/chrony/drift
keyfile /etc/chrony.keys
...
```

`offline` 키워드를 지정하면 `chronyd`는 네트워크 액세스가 가능하다는 통신을 수신할 때까지 NTP 서버를 폴링하지 않습니다. `chronyc online` 및 `chronyc 오프라인` 명령을 사용하여 `chronyd`에 네트워크 액세스 상태를 알릴 수 있습니다.

구성 파일과 해당 지시문에 대한 자세한 내용은 `chrony.conf(5)` 매뉴얼 페이지를 참조하세요.

### ntp에서 chrony로 변환

다음 표는 `ntp`와 `chrony` 사이에 해당하는 파일, 명령 및 용어를 보여줍니다.

| ntp                            | chrony                                 |
| ------------------------------ | -------------------------------------- |
| `/etc/ntp.conf`                | `/etc/chrony.conf`                     |
| `/etc/ntp/keys`                | `/etc/chrony.keys`                     |
| `ntpd`                         | `chronyd`                              |
| `ntpq` command                 | `chronyc` command                      |
| `ntpd.service`                 | `chronyd.service`                      |
| `ntp-wait.service`             | `chrony-wait.service`                  |
| `ntpdate` and `sntp` utilities | `chronyd -q` and `chronyd -t` commands |

`ntpstat` 패키지에 포함된 `ntpstat` 유틸리티가 이제 `chronyd`를 지원합니다. 따라서 Enterprise Linux 8에서 이 유틸리티를 계속 사용할 수 있습니다. 이 명령은 `ntp`와 함께 사용될 때와 유사한 출력을 생성합니다.

`/usr/share/doc/chrony/ntp2chrony.py` 스크립트를 사용하면 기존 `ntp` 구성을 `chrony`로 변환하는 데 도움이 됩니다.

```
sudo python3 /usr/share/doc/chrony/ntp2chrony.py -b -v
```

이 스크립트는 `/etc/ntp.conf`에서 가장 일반적인 지시문을 `chrony`로 변환하는 것을 지원합니다. 예제에서 `-b` 옵션은 변환 전에 백업 구성 파일을 생성하도록 지정하고, `-v` 옵션은 마이그레이션 프로세스 중에 자세한 메시지를 표시하도록 지정합니다.

스크립트와 함께 사용할 수 있는 다양한 옵션을 나열하려면 다음 명령을 입력하십시오.:

```
sudo python3 /usr/share/doc/chrony/ntp2chrony.py --help
```

## PTP

PTP를 사용하면 NTP보다 더 정확하게 Lan 상의 시스템의 시계를 동기화할 수 있습니다. 네트워크 드라이버가 하드웨어 또는 소프트웨어 타임스탬프를 지원하는 경우 PTP 시계는 PTP 메시지의 타임스탬프를 사용하여 네트워크 전체의 전파 지연을 해결할 수 있습니다. 소프트웨어 타임 스탬핑을 통해 PTP는 수십 마이크로초 이내에 시스템을 동기화합니다. 하드웨어 타임 스탬프를 사용하면 PTP는 수십 마이크로초 이내에 시스템을 동기화할 수 있습니다. 시스템의 고정밀 시간 동기화가 필요한 경우 하드웨어 타임 스탬프를 사용하십시오.

엔터프라이즈 LAN 상의 일반적인 PTP 구성은 다음과 같이 구성됩니다.:

- 하나 이상의 _그랜드마스터 시계_ 시스템.

  그랜드마스터 시계는 일반적으로 고정확도 GPS 신호 또는 저정확도 코드 분할 여러 액세스 \(CDMA\) 신호, 무선 시계 신호 또는 NTP를 시간 참조 소스로 사용할 수 있는 특수 하드웨어로 구현됩니다. 여러 개의 그랜드마스터 시계를 사용할 수 있는 경우 최고의 마스터 시계\(BMC\) 알고리즘은 `priority1`, `clockClass`, `clockAccuracy`, `offsetScaledLogVariance` 및 `priority2` 매개변수의 설정을 기반으로 그랜드마스터 시계를 선택합니다.

- 여러 _경계 시계_ 시스템.

  각 경계 시계는 하나의 하위 네트워크에 있는 그랜드마스터 시계에 백업되고 하나 이상의 추가된 하위 네트워크에 PTP 메시지를 중계합니다. 경계 클록은 일반적으로 네트워크 스위치의 기능으로 구현됩니다.

- 여러 개의 _보조 시계_ 시스템.

  서브 네트워크의 각 보조 시계는 해당 보조 시계의 _마스터 시계_ 역할을 하는 경계 시계에 백업됩니다.

기본 구성의 경우 동일한 네트워크 세그먼트에 단일 그랜드마스터 클럭과 여러 보조 클럭을 설정하면 경계 클럭의 중간 레이어가 필요하지 않습니다.

PTP에 하나의 네트워크 인터페이스만 사용하는 그랜드마스터 및 보조 시계 시스템을 _일반 시계_라고 합니다.

경계 시계에는 PTP용 네트워크 인터페이스가 두 개 이상 필요합니다.

경계 및 보조 시계 시스템의 동기화는 PTP 메시지에 타임스탬프를 전송하여 이루어집니다. 기본적으로 PTP 메시지는 UDPv4 데이터그램으로 전송됩니다. UDPv6 데이터그램이나 이더넷 프레임을 전송 메커니즘으로 사용하도록 PTP를 구성할 수도 있습니다.

시스템에서 PTP를 사용하려면 시스템 네트워크 인터페이스 중 하나 이상에 대한 드라이버가 소프트웨어 또는 하드웨어 타임스탬프를 지원해야 합니다. 네트워크 인터페이스용 드라이버가 타임스탬프를 지원하는지 확인하려면 `ethtool` 명령을 사용하세요.:

```
sudo ethtool -T en1
```

```nocopybutton
Time stamping parameters for en1:
Capabilities:
	hardware-transmit     (SOF_TIMESTAMPING_TX_HARDWARE)
	software-transmit     (SOF_TIMESTAMPING_TX_SOFTWARE)
	hardware-receive      (SOF_TIMESTAMPING_RX_HARDWARE)
	software-receive      (SOF_TIMESTAMPING_RX_SOFTWARE)
	software-system-clock (SOF_TIMESTAMPING_SOFTWARE)
	hardware-raw-clock    (SOF_TIMESTAMPING_RAW_HARDWARE)
...
```

예제의 출력은 `en1` 인터페이스가 하드웨어 및 소프트웨어 타임스탬프 기능을 모두 지원함을 보여줍니다.

소프트웨어 타임스탬프를 사용하면 `ptp4l`은 시스템 시계를 외부 그랜드마스터 시계와 동기화합니다.

하드웨어 타임 스탬프를 사용할 수 있는 경우 `ptp4l`은 PTP 하드웨어 시계를 외부 그랜드마스터 시계와 동기화할 수 있습니다. 이 경우 `phc2sys` 데몬을 사용하여 시스템 시계를 PTP 하드웨어 시계와 동기화합니다.

### PTP 서비스 구성

시스템에서 PTP 서비스를 구성하려면:

1. `linuxptp` 패키지를 설치합니다.

   ```
   sudo dnf install linuxptp
   ```

2. `/etc/sysconfig/ptp4l`을 편집하고 `ptp4l` 데몬에 대한 시작 옵션을 정의합니다.

   Grandmaster 클럭과 보조 클럭에서는 하나의 인터페이스만 정의해야 합니다.

   예를 들어, 보조 시계에서 'en1' 인터페이스와 함께 하드웨어 타임스탬프를 사용하려면 다음을 수행하세요.

   ```
   OPTIONS="-f /etc/ptp4l.conf -i en1 -s"
   ```

   하드웨어 타임스탬프 대신 소프트웨어 타임스탬프를 사용하려면 `-S` 옵션을 지정하세요.

   ```
   OPTIONS="-f /etc/ptp4l.conf -i en1 -S -s"
   ```

   **Note:**

   `-s` 옵션은 시계가 보조 \(`slaveOnly` 모드\)로만 작동하도록 지정합니다. 그랜드마스터 시계나 경계 시계에는 이 옵션을 지정하지 마세요.

   그랜드마스터 시계의 경우 `-s` 옵션을 생략하세요.

   ```
   OPTIONS="-f /etc/ptp4l.conf -i en1"
   ```

   경계 시계를 사용하려면 최소한 두 개의 인터페이스를 정의해야 합니다.

   ```
   OPTIONS="-f /etc/ptp4l.conf -i en1 -i en2"
   ```

   예를 들어 `ptp4l`을 추가로 사용자 정의하려면 `/etc/ptp4l.conf` 파일을 편집해야 할 수도 있습니다.:

   - 그랜드마스터 시계의 경우 `priority1` 매개변수 값을 0에서 127 사이의 값으로 설정합니다. 단일 그랜드마스터 시계가 있는 구성의 경우 값 127이 제안됩니다.

   - `summary_interval` 값을 0 대신 정수 값 _N_으로 설정하면 `ptp4l`은 1초가 아닌 2_N_초마다 요약 시계 통계를 `/var/log/messages`에 기록합니다 \(20 = 1\). 예를 들어 값 10은 210초 또는 1024초 간격에 해당합니다.

   - `logging_level` 매개변수는 `ptp4l`이 기록하는 로깅 정보의 양을 제어합니다. `logging_level`의 기본값은 `6`이며 `LOG_INFO`에 해당합니다. 로깅을 끄려면 `logging_level` 값을 `0`으로 설정하세요. 또는 `ptp4l`에 `-q` 옵션을 지정하세요.

   `ptp4l(8)` 매뉴얼 페이지를 참조하세요.

3. 예를 들어 적절한 영역의 UDP 포트 319 및 320에 대한 PTP 이벤트 및 일반 메시지의 액세스를 허용하도록 시스템 방화벽을 구성합니다.:

   ```
   sudo firewall-cmd --zone=*zone* --add-port=319/udp --add-port=320/udp
   ```

   ```
   sudo firewall-cmd --permanent --zone=*zone* --add-port=319/udp --add-port=320/udp
   ```

4. `ptp4l` 서비스를 시작하고 시스템 재부팅 후 시작되도록 구성합니다.

   ```
   sudo systemctl start ptp4l
   ```

   ```
   sudo systemctl enable ptp4l
   ```

5. 하드웨어 타임스탬프를 사용하는 시계 시스템에서 `phc2sys`를 구성하려면:

   1. `/etc/sysconfig/phc2sys` 파일을 편집하고 `phc2sys` 데몬에 대한 시작 옵션을 정의합니다.

      경계 시계 또는 보조 시계에서 시스템 시계를 보조 네트워크 인터페이스와 연결된 PTP 하드웨어 시계와 동기화합니다.

      ```
      OPTIONS="-c CLOCK_REALTIME -s en1 -w"
      ```

      **Note:**

      바운더리 시계의 보조 네트워크 인터페이스는 그랜드마스터 시계와 통신하는 데 사용되는 인터페이스입니다.

      `-w` 옵션은 `phc2sys`가 시스템 시계를 동기화하기 전에 `ptp4l`이 PTP 하드웨어 시계를 동기화할 때까지 기다리도록 지정합니다.

      GPS, CDMA, NTP 또는 무선 시간 신호와 같은 참조 시간 소스에서 시스템 시간을 가져오는 그랜드마스터 시계에서는 시스템 시계에서 네트워크 인터페이스의 PTP 하드웨어 시계를 동기화합니다.:

      ```
      OPTIONS="-c en1 -s CLOCK_REALTIME -w"
      ```

      `phc2sys(8)` 매뉴얼 페이지를 참조하세요.

   2. `phc2sys` 서비스를 시작하고 시스템 재부팅 후 시작되도록 구성합니다.

      ```
      sudo systemctl start phc2sys
      ```

      ```
      sudo systemctl enable phc2sys
      ```

`pmc` 명령을 사용하여 `ptp4l` 작업 상태를 쿼리할 수 있습니다. 다음 예는 중간 경계 시계 없이 그랜드마스터 시계 시스템에 직접 연결된 슬레이브 시계 시스템에서 `pmc`를 실행한 결과를 보여줍니다.:

```
sudo pmc -u -b 0 'GET TIME_STATUS_NP'
```

```nocopybutton
sending: GET TIME_STATUS_NP
	080027.fffe.7f327b-0 seq 0 RESPONSE MANAGEMENT TIME_STATUS_NP 
		master_offset              -98434
		ingress_time               1412169090025854874
		cumulativeScaledRateOffset +1.000000000
		scaledLastGmPhaseChange    0
		gmTimeBaseIndicator        0
		lastGmPhaseChange          0x0000'0000000000000000.0000
		gmPresent                  true
		gmIdentity                 080027.fffe.d9e453
```

```
sudo pmc -u -b 0 'GET CURRENT_DATA_SET'
```

```nocopybutton
sending: GET CURRENT_DATA_SET
	080027.fffe.7f327b-0 seq 0 RESPONSE MANAGEMENT CURRENT_DATA_SET 
		stepsRemoved     1
		offsetFromMaster  42787.0
		meanPathDelay    289207.0
```

이 출력 예시에는 다음과 같은 유용한 정보가 포함되어 있습니다.:

- **`gmIdentity`**

  네트워크 인터페이스의 MAC 주소를 기반으로 하는 그랜드마스터 시계의 고유 식별자입니다.

- **`gmPresent`**

  외부 그랜드마스터 시계를 사용할 수 있는지 여부입니다. 이 값은 그랜드마스터 시계 자체에 'false'로 표시됩니다.

- **`meanPathDelay`**

  동기화 메시지가 지연되는 예상 시간(나노초)입니다.

- **`offsetFromMaster`**

  그랜드마스터 시계를 기준으로 나노초 단위의 시간 차이를 가장 최근에 측정한 것입니다.

- **`stepsRemoved`**

  이 시스템과 그랜드마스터 클럭 사이의 네트워크 단계 수입니다.

자세한 내용은 `phc2sys(8)`, `pmc(8)`, `ptp4l(8)` 매뉴얼 페이지와 [IEEE 1588](https://www.nist.gov/el/intelligent-)을 참조하세요.

### PTP를 NTP의 시간 소스로 사용

NTP 클라이언트가 사용할 수 있는 NTP 서버의 PTP 조정 시스템 시간을 만들려면 NTP 서버의 `/etc/chrony.conf` 파일에 다음 항목을 포함하십시오.:

```
server    127.127.1.0
fudge     127.127.1.0 stratum 0
```

이러한 항목은 로컬 시스템 시계를 시간 참조로 정의합니다.

**Note:**

파일에 추가된 `server` 줄을 구성하지 마세요.
