# BuildThing™ ️On-premise Server

![BuildThing](https://buildit.kr/dist/img/icon-40@2x.png)

BuildThing™ On-premise Server는 BuildThing Gateway와 연동되어 데이터를 데이터베이스에 저장하는 설치형 서버 입니다.
[Java 17](https://www.oracle.com/java/technologies/javase/jdk17-archive-downloads.html) 기반으로 동작하며, 데이터베이스 툴을 통해 데이터베이스([SQLite](https://www.sqlite.org/))에 저장된 센서 데이터를  조회할 수 있습니다.
서버에서 제공하는 주요 기능들은 다음과 같습니다.

  - BuildThing Gateway/Beacon/IAQ의 상태 정보 수집
  - BuildThing Gateway로부터 BuildThing Beacon/IAQ 센서 이력 수집

`아래 가이드 문서는 Windows를 기반으로 작성되었습니다.`

## 목차
- [BuildThing™ ️On-premise Server](#buildthing-️on-premise-server)
  - [목차](#목차)
  - [사전 설치](#사전-설치)
  - [서버 실행](#서버-실행)
  - [방화벽 허용](#방화벽-허용)
  - [BuildThing Admin 앱을 통한 BuildThing Gateway의 서버 연결](#buildthing-admin-앱을-통한-buildthing-gateway의-서버-연결)
    - [BuildThing Gateway 스캔 및 연결](#buildthing-gateway-스캔-및-연결)
    - [BuildThing Gateway (Universal Type) 설정](#buildthing-gateway-universal-type-설정)
    - [BuildThing Gateway (IAQ+Universal Type) 설정](#buildthing-gateway-iaquniversal-type-설정)
    - [BuildThing Gateway 지원 펌웨어](#buildthing-gateway-지원-펌웨어)
  - [데이터베이스에 수집된 데이터 조회](#데이터베이스에-수집된-데이터-조회)
    - [각 테이블 정보 설명](#각-테이블-정보-설명)
      - [device\_gateway](#device_gateway)
      - [device\_beacon](#device_beacon)
      - [device\_iaq](#device_iaq)
      - [beacon\_telemetry\_history](#beacon_telemetry_history)
      - [iaq\_telemetry\_history](#iaq_telemetry_history)
    - [다양한 데이터를 조회하기 위한 SQL문 예시](#다양한-데이터를-조회하기-위한-sql문-예시)
      - [특정 시각에 수집된 특정 비콘의 센서 이력 조회하기](#특정-시각에-수집된-특정-비콘의-센서-이력-조회하기)
      - [특정 시각에 수집된 특정 IAQ의 센서 이력 조회하기](#특정-시각에-수집된-특정-iaq의-센서-이력-조회하기)
    - [데이터 CSV 추출](#데이터-csv-추출)
  - [Windows 서비스에 등록하기 (Optional)](#windows-서비스에-등록하기-optional)
  - [고객 문의](#고객-문의)

## 사전 설치
1. BuildThing™ On-premise Server의 실행을 위해서는 [Java 17](https://www.oracle.com/java/technologies/javase/jdk17-archive-downloads.html)을 필요로합니다. 
2. 서버에 수집된 데이터 조회를 위해서는 데이터베이스 조회 툴인 [DBeaver](https://dbeaver.io/download/)을 필요로합니다.

## 서버 실행
1. data-server.jar파일을 다운로드합니다.
2. Windows용 [Java 17 인스톨러](https://download.oracle.com/java/17/archive/jdk-17.0.9_windows-x64_bin.exe)를 설치합니다.
3. 명령 프롬프트(찾기 > cmd 입력)에서 data-server.jar가 다운로드된 디렉토리로 이동합니다.
```sh
C:\> cd C:\Users\buildit\Downloads
```
4. 아래 명령을 실행합니다.
```sh
C:\Users\buildit\Downloads>java -jar data-server.jar
```

## 방화벽 허용
BuildThing Gateway를 연결하기 위해서 아래의 절차에 따라 PC 또는 서버의 방화벽을 허용합니다. `BuildThing Gateway를 PC 또는 서버에 연결하기 위해서는 BuildThing Gateway와 PC 또는 서버가 같은 네트워크 망에 연결되어 있어야합니다.`

1. 명령 프롬프트(찾기 > cmd 입력)에서 ipconfig를 입력하여 내 IPv4 주소를 확인합니다.

```sh
Windows IP 구성
이더넷 어댑터 이더넷:

   연결별 DNS 접미사. . . . : 
   ... (생략)
   IPv4 주소 . . . . . . . . . : 192.168.0.200
   ... (생략)
```
2. 방화벽 및 네트워크 보호 > 고급 설정을 실행합니다.
![](https://buildit.kr/dist/img/image-20240312-004141.png)
![](https://buildit.kr/dist/img/image-20240312-004238.png)


3. 인바운드 규칙 > 새 규칙을 클릭하여 새 인바운드 규칙을 생성합니다. `예시는 192.168.0.x IP 주소 대역 기준으로 설정하고 있습니다. 상기 ipconfig를 통해 조회된 IP 주소 대역에 맞추어 설정합니다.`
![](https://buildit.kr/dist/img/image-20240312-004354.png)
![](https://buildit.kr/dist/img/image-20240312-063349.png)
![](https://buildit.kr/dist/img/image-20240312-063409.png)
![](https://buildit.kr/dist/img/image-20240312-063956.png)
![](https://buildit.kr/dist/img/image-20240312-063629.png)
![](https://buildit.kr/dist/img/image-20240312-063732.png)
![](https://buildit.kr/dist/img/image-20240312-063756.png)

4. 규칙 이름을 입력하여 새 규칙 생성을 완료합니다.

## BuildThing Admin 앱을 통한 BuildThing Gateway의 서버 연결
### BuildThing Gateway 스캔 및 연결
- BuildThing Gateway의 스캔 및 연결은 [BuildThing Admin 매뉴얼 > 게이트웨이 스캔 및 등록, 게이트웨이 연결 및 설정](https://amazing-armchair-09d.notion.site/BuildThing-Admin-74c0b96d608947d48198739d6dc540e7#b196794bf1174726b30a5a78a5970053)문서를 참고해주세요.

### BuildThing Gateway (Universal Type) 설정
- 아래의 스크린샷과 같이 설정합니다.
![](https://buildit.kr/dist/img/IMG_9498.PNG)
![](https://buildit.kr/dist/img/IMG_9499.PNG)

|항목|설정 값| 값 예시|
|---|---|--|
|스캔 기기 타입|Beacon 또는 Other Beacon|Beacon|
|서버 Host 주소|On-Premise 서버의 IP 주소|192.168.0.200|
|서버 포트|On-Premise 서버의 포트|8088|
|토큰 인증 방식|OFF||
|통신 프로토콜|HTTP||
|Telemetry URL|devices/telemetry||
|Attributes URL|devices/attributes||

### BuildThing Gateway (IAQ+Universal Type) 설정
- 아래의 스크린샷과 같이 설정합니다.
![](https://buildit.kr/dist/img/IMG_9500.PNG)
![](https://buildit.kr/dist/img/IMG_9502.PNG)

|항목|설정 값| 값 예시|
|---|---|--|
|스캔 기기 타입|빈 값||
|서버 Host 주소|On-Premise 서버의 IP 주소|192.168.0.200|
|서버 포트|On-Premise 서버의 포트|8088|
|토큰 인증 방식|OFF||
|통신 프로토콜|HTTP||
|Telemetry URL|devices/telemetry||
|Attributes URL|devices/attributes||

### BuildThing Gateway 지원 펌웨어
- 구매하신 BuildThing Gateway의 펌웨어 타입은 설정 모드에 진입 후 BuildThing Admin으로 스캔 시 확인할 수 있습니다.
- BuildThing IAQ의 데이터를 수집하기 위해서는 IAQ Type의 BuildThing Gateway에 [IAQ+Universal Type 펌웨어](https://www.notion.so/Gateway-Firmware-IAQ-Universal-Type-88af4057dde64bf08c9caf56b5c133c1?pvs=4#1c7435edf9ed477c8d4339d3afa80043)를 다운로드하셔야합니다.
- BuildThing Beacon의 데이터를 수집하기 위해서는 Universal Type의 BuildThing Gateway에 [Universal Type 펌웨어](https://www.notion.so/Gateway-Firmware-Universal-Type-3a00c272a33340d5868710460d18913f?pvs=4#682c189adf784ef6a09a726ecd2aa459)를 다운로드하셔야합니다.
- BuildThing Gateway 펌웨어 업데이트는 [BuildThing Gateway 매뉴얼 > FOTA를 이용한 펌웨어 업데이트](https://amazing-armchair-09d.notion.site/BuildThing-Gateway-8159838dc24d4d2d8f498a681758fff0#b1e117ff89e34b23b9b584da3815babb)문서를 참고해주세요.


## 데이터베이스에 수집된 데이터 조회
1. Windows용 [DBeaver Installer](https://dbeaver.io/files/dbeaver-ce-latest-x86_64-setup.exe)를 설치합니다.
2. DBeaver를 실행하고 데이터베이스 > 새 데이터베이스 연결(단축키 : Ctrl + Shift + n)을 클릭합니다. 
3. SQLite를 선택하고 data-server.jar 파일이 위치한 디렉토리에 생성된 buildthing.db 파일을 불러옵니다.

![](https://buildit.kr/dist/img/image-20240319-020534.png)
![](https://buildit.kr/dist/img/image-20240319-020643.png)

4. 연결된 DB를 펼치고 테이블에 오른쪽 버튼 클릭 > View 테이블을 선택하면 수집된 데이터를 조회할 수 있습니다.
![](https://buildit.kr/dist/img/db-1.png)
![](https://buildit.kr/dist/img/db-2.png)

### 각 테이블 정보 설명
#### device_gateway
- BuildThing Gateway의 상태 정보를 저장합니다.

|컬럼명|설명|
|---|---|
|device_id|게이트웨이 맥주소|
|type|타입 - 1: 게이트웨이|
|name|게이트웨이 이름|
|updated_time|최근 업데이트 시각|
|scan_type|게이트웨이 스캔 타입 - 0: Beacon, 1: OtherBeacon, 5: IAQ+Universal|
|comm_type|통신 타입 - 0: Ethernet, 1: WiFi|
|error_code|에러 코드 - 0: 정상, 1: 인증 실패, 2: 네트워크 에러, 3: 배터리 부족, 8: 유심 없음|
|firmware_version|펌웨어 버전|
|network_period|네트워크 통신 주기|
|protocol|통신 프로토콜 - 0: MQTT, 1: HTTP, 2: HTTPS|

#### device_beacon
- BuildThing Beacon의 상태 정보를 저장합니다.

|컬럼명|설명|
|---|---|
|device_id|비콘 맥주소|
|type|타입 - 0: 비콘|
|name|비콘 이름|
|updated_time|최근 업데이트 시각|
|advertising_invterval|Advertising 주기|
|sensing_invterval|Sensing 주기|
|tx_power|Tx Power|
|battery|배터리|
|gateway_id|스캔한 게이트웨이 ID|
|major|Major|
|minor|Minor|
|temperature|온도 (섭씨)|
|humidity|습도 (%)|
|tvoc|TVOC (ppb)|
|nh3|암모니아 (ppm)|
|no2|이산화질소 (ppm)|
|motion|모션감지 - 0: 없음, 1: 낙하, 2: 더블 탭, 3: 더블 탭 + 낙하, 4: 탭, 5: 낙하 + 탭, 6: 탭 + 더블 탭, 7: 탭 + 낙하 + 더블 탭|
|x|X축 기울기|
|y|Y축 기울기|
|z|Z축 기울기|
|adc_module_number|ADC 모듈 연결 번호|
|io_module_number|IO 모듈 연결 번호|
|input|IO 모듈 INPUT 값 (8bit, 0~7)|
|output|IO 모듈 OUTPUT 값 (8bit, 0~7)|
|port_0|ADC 모듈 0번 포트 값|
|port_1|ADC 모듈 1번 포트 값|
|port_2|ADC 모듈 2번 포트 값|
|port_3|ADC 모듈 3번 포트 값|

#### device_iaq
- BuildThing IAQ의 상태 정보를 저장합니다.
  
|컬럼명|설명|
|---|---|
|device_id|비콘 맥주소|
|type|타입 - 2: IAQ|
|name|IAQ 이름|
|updated_time|최근 업데이트 시각|
|co2|이산화탄소 (ppm)|
|tvoc|TVOC (ppb)|
|temperature_c|온도 (섭씨)|
|temperature_f|온도 (화씨)|
|humidity|습도 (%)|
|pm_1_0|극초미세먼지 (ug/m3)|
|pm_2_5|초미세먼지 (ug/m3)|
|pm_10_0|미세먼지 (ug/m3)|
|score|IAQ 스코어|
|firmware_version|펌웨어 버전|
|gateway_id|스캔한 게이트웨이 ID|


#### beacon_telemetry_history
- BuildThing Beacon의 센서 이력을 저장합니다.

|컬럼명|설명|
|---|---|
|inserted_time|수집 시각|
|temperature|온도 (섭씨)|
|humidity|습도 (%)|
|tvoc|TVOC (ppb)|
|nh3|암모니아 (ppm)|
|no2|이산화질소 (ppm)|
|motion|모션감지 - 0: 없음, 1: 낙하, 2: 더블 탭, 3: 더블 탭 + 낙하, 4: 탭, 5: 낙하 + 탭, 6: 탭 + 더블 탭, 7: 탭 + 낙하 + 더블 탭|
|x|X축 기울기|
|y|Y축 기울기|
|z|Z축 기울기|
|adc_module_number|ADC 모듈 연결 번호|
|io_module_number|IO 모듈 연결 번호|
|input|IO 모듈 INPUT 값 (8bit, 0~7)|
|output|IO 모듈 OUTPUT 값 (8bit, 0~7)|
|port_0|ADC 모듈 0번 포트 값|
|port_1|ADC 모듈 1번 포트 값|
|port_2|ADC 모듈 2번 포트 값|
|port_3|ADC 모듈 3번 포트 값|


#### iaq_telemetry_history
- BuildThing IAQ의 센서 이력을 저장합니다.
  
|컬럼명|설명|
|---|---|
|inserted_time|수집 시각|
|co2|이산화탄소 (ppm)|
|tvoc|TVOC (ppb)|
|temperature_c|온도 (섭씨)|
|temperature_f|온도 (화씨)|
|humidity|습도 (%)|
|pm_1_0|극초미세먼지 (ug/m3)|
|pm_2_5|초미세먼지 (ug/m3)|
|pm_10_0|미세먼지 (ug/m3)|
|score|IAQ 스코어|


### 다양한 데이터를 조회하기 위한 SQL문 예시
- SQL 편집기 > SQL 편집기를 클릭하여 SQL문을 입력을 통해 다양한 방식으로 데이터를 조회할 수 있습니다.
  
#### 특정 시각에 수집된 특정 비콘의 센서 이력 조회하기
- 아래와 같이 SQL문을 입력하고 좌측에 SQL문 실행 버튼을 클릭합니다.
- 조회할 비콘의 맥 주소, 수집 시작 시각, 수집 종료 시각은 형식에 맞추어 임의로 변경할 수 있습니다.
``` sql
SELECT 
	db.name, db.device_id, bth.* 
FROM 
	beacon_telemetry_history bth 
JOIN 
	device_beacon db on db.id = bth.device_id 
WHERE
	db.device_id = 'F8:46:AF:78:EF:0B' /*조회할 비콘의 맥주소*/
AND 
	bth.inserted_time > '2024-03-19T00:00:00' /*수집 시작 시각*/
AND 
	bth.inserted_time < '2024-03-20T00:00:00' /*수집 종료 시각*/
ORDER BY bth.inserted_time DESC; /*DESC : 내림 차순 조회, ASC: 오름 차순 ;
```

#### 특정 시각에 수집된 특정 IAQ의 센서 이력 조회하기
- 아래와 같이 SQL문을 입력하고 좌측에 SQL문 실행 버튼을 클릭합니다.
- 조회할 IAQ의 맥 주소, 수집 시작 시각, 수집 종료 시각은 형식에 맞추어 임의로 변경할 수 있습니다.
``` sql
SELECT 
	ith.inserted_time, di.name, di.device_id, 
	ith.score,
	ith.co_2, ith.temperature_c, ith.temperature_f, ith.humidity, 
	ith.tvoc, ith.pm_1_0, ith.pm_2_5, ith.pm_10_0 
FROM 
	iaq_telemetry_history ith 
JOIN 
	device_iaq di  on di.id = ith.device_id 
WHERE
	di.device_id = 'F9:46:AF:78:EF:0B' /*찾을 IAQ의 맥주소*/
AND 
	ith.inserted_time >= '2024-03-19T14:05:00' /*수집 시작 시각*/
AND 
	ith.inserted_time <= '2024-03-19T14:40:00' /*수집 종료 시각*/
ORDER BY ith.inserted_time DESC; /*DESC : 내림 차순 조회, ASC: 오름 차순 조회*/
```

### 데이터 CSV 추출
- 아래 스크린 샷과 같이 조회한 데이터를 CSV로 추출할 수 있습니다.
![](https://buildit.kr/dist/img/export-1.png)
![](https://buildit.kr/dist/img/export-2.png)
![](https://buildit.kr/dist/img/export-3.png)

## Windows 서비스에 등록하기 (Optional)
- 명령 프롬프트 창에서의 실행 없이 Windows의 백그라운드 서비스로 On-premise 서버를 등록하는 방법을 설명합니다.
1. Windows용 서비스 관리 툴인 [NSSM](https://nssm.cc/download)를 [다운로드](https://nssm.cc/release/nssm-2.24.zip)하고 압축을 해제합니다.
2. 명령 프롬프트(찾기 > cmd 입력)에서 NSSM이 압축 해제된 디렉토리로 이동합니다.
```sh
C:\>cd C:\Users\buildit\Downloads\nssm\nssm-2.24
```
3. 윈도우가 32bit 이면 win32, 64bit이면 win64로 이동합니다. 
```sh
C:\Users\buildit\Downloads\nssm\nssm-2.24>cd win64
```
4. 새로운 서비스 등록을 위해 아래와 같이 실행합니다.
```sh
C:\Users\buildit\Downloads\nssm\nssm-2.24\win64> nssm.exe install "BuildThing"
```
5. 아래 스크린샷과 같이 입력하고 Install Service 버튼을 눌러 서비스를 등록합니다.
![](https://buildit.kr/dist/img/nssm.png)

|항목|값 예시|비고|
|---|---|--|
|Path|C:\Program Files\Java\jdk-17\bin\java.exe|Java 17이 설치된 위치|
|Startup directory|C:\Users\buildit\Downloads|data-server.jar가 위치한 디렉토리|
|Arguments|-jar data-server.jar --spring.profiles.active=prod --server.port=8088|실행 명령어. port는 8088 대신 원하는 포트로 설정 가능. 단, 방화벽 설정에도 반영이 되어야함.|
6. Windows 찾기 > 서비스를 입력하여 실행합니다.
![](https://buildit.kr/dist/img/image-20240308-055215.png)
7. 등록된 서비스인 BuildThing 서비스를 찾아 시작합니다.
![](https://buildit.kr/dist/img/service-2.png)

## 고객 문의
기타 문의 사항은 info@buildit.kr로 문의해주시기 바랍니다.
