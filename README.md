# DevStack 설치 가이드

테스트, 학습 환경으로 구성하기에 적합한 **DevStack**을 통해 **OpenStack**을 설치하는 과정을 정리합니다.

---

## 설치 환경

- 가상화 소프트웨어: **Oracle VirtualBox**
- OS: **Ubuntu 22.04 LTS Server**
- DevStack 버전: **unmaintained/2023.1**
- 자원 설정:
  - 메모리: **6GB (6144 MB)**
  - 디스크: **40GB**
  - CPU: **4 Core**

> 본 가이드는 Ubuntu 22.04 LTS 환경에서 실습 재현성을 확보하기 위해 DevStack `unmaintained/2023.1` 브랜치를 기준으로 작성되었습니다.
> DevStack 최신 master 브랜치는 Ubuntu/Python/패키지 의존성 조합에 따라 설치 오류가 발생할 수 있습니다.

PDF 및 유튜브 영상을 참고하여 환경 설치를 모두 끝낸 후 아래 실습을 진행해주세요!

**[OpenStack 실습 강의 ①] VirtualBox 기반 OpenStack 환경 구축:**
https://youtu.be/PwCHXG9frpg

**[OpenStack 실습 강의 ②] DevStack 설치 & OpenStack VM 생성과 네트워크 통신 실습:**
https://youtu.be/efGyEr54Jyw

---

## 설치 전 주의사항

DevStack은 OpenStack 개발 및 실습용 설치 도구입니다. 운영 환경 구축용으로 사용하지 않습니다.

이 가이드에서 사용하는 `unmaintained/2023.1` 브랜치는 운영 환경 권장 버전이 아니라, Ubuntu 22.04 LTS와 Python 3.10 기반 실습 환경에서 설치 재현성을 확보하기 위한 기준입니다.

최신 master 브랜치 또는 다른 stable 브랜치를 사용할 경우 DevStack 자체 코드, OpenStack 서비스 브랜치, Python 패키지 제약 조건이 서로 맞지 않아 설치가 실패할 수 있습니다.

---

## 설치 절차

### 1. Repository Update 및 필수 패키지 설치

명령어를 실행하기 전에 Ubuntu 패키지 목록을 갱신하고 DevStack 설치에 필요한 기본 패키지를 설치합니다.

```bash
sudo apt update
sudo apt install python3 python3-pip virtualenv git -y
```

### 2. stack 사용자 생성 및 권한 설정

DevStack은 일반적으로 `stack` 사용자로 설치를 진행합니다.

```bash
sudo useradd -s /bin/bash -d /opt/stack -m stack
sudo chmod +x /opt/stack
echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
sudo -u stack -i
```

### 3. DevStack 다운로드

Ubuntu 22.04에서 설치가 확인된 DevStack `unmaintained/2023.1` 브랜치를 다운로드합니다.

```bash
git clone -b unmaintained/2023.1 https://opendev.org/openstack/devstack
cd devstack
```

최신 master 브랜치를 그대로 사용할 경우 Ubuntu 22.04의 기본 Python 3.10 환경에서 Python 버전 또는 패키지 의존성 충돌이 발생할 수 있습니다.

따라서 본 실습에서는 Ubuntu 22.04에서 설치가 확인된 `unmaintained/2023.1` 브랜치를 사용합니다.

### 4. local.conf 설정

#### (1) local.conf 파일 생성 및 수정

DevStack 설정 파일인 `local.conf`를 생성한 뒤 내용을 수정합니다.

```bash
cp ./samples/local.conf local.conf
vim local.conf
```

#### (2) local.conf 파일 수정 내용

아래 예시는 DevStack 설정 파일 형식에 맞춰 `[[local|localrc]]` 섹션을 포함한 기본 설정입니다.

```ini
[[local|localrc]]
ADMIN_PASSWORD=stack
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD

HOST_IP=192.168.x.x

LOGFILE=$DEST/logs/stack.sh.log
VERBOSE=True
LOG_COLOR=False
```

`HOST_IP`에는 VM의 Host-only 또는 Bridged 네트워크 IP를 입력합니다.

VirtualBox 환경에서는 NAT IP와 Host-only IP가 함께 보일 수 있습니다.

예시:

- NAT IP: `10.0.2.15`
- Host-only IP: `192.168.56.x`

호스트 PC 브라우저에서 Horizon 대시보드에 접속하려면 일반적으로 Host-only 또는 Bridged IP를 `HOST_IP`로 지정합니다.

IP 주소와 라우팅 정보는 아래 명령어로 확인할 수 있습니다.

```bash
ip -br addr
ip route
hostname -I
```

### 5. DevStack 설치 실행

설정을 저장한 뒤 DevStack 설치 스크립트를 실행합니다.

```bash
./stack.sh
```

설치 과정은 다소 시간이 소요될 수 있습니다. 설치 중 오류가 발생하면 로그를 확인하여 문제를 해결해야 합니다.

Ubuntu 22.04 환경에서 아래와 같은 경고가 발생할 수 있습니다.

```bash
WARNING: this script has not been tested on jammy
If you wish to run this script anyway run with FORCE=yes
```

이 경우 아래 명령어로 다시 실행합니다.

```bash
FORCE=yes ./stack.sh
```

`FORCE=yes`는 Ubuntu 22.04 Jammy 미검증 경고를 우회하는 옵션입니다.

Python 버전 충돌이나 패키지 의존성 충돌을 해결하는 옵션은 아니므로, DevStack 브랜치는 반드시 `unmaintained/2023.1`로 고정해야 합니다.

### 6. Horizon 접속 확인

설치 완료 후 Horizon 대시보드에 접속하려면 브라우저에서 `http://<HOST_IP>/dashboard`로 접근하세요.

접속 예시:

```text
http://192.168.56.101/dashboard
```

로그인 정보:

- Domain: `Default`
- User Name: `admin`
- Password: `stack`

---

## private 네트워크 및 NAT 라우터 수동 구성

DevStack 2023.1 환경에서는 실습에 필요한 private 네트워크, private subnet, router가 자동으로 생성되지 않을 수 있습니다.

아래 명령어로 네트워크와 라우터가 존재하는지 먼저 확인합니다.

```bash
source openrc admin admin

openstack network list
openstack router list
```

`private` 네트워크 또는 라우터가 없다면 Horizon 대시보드에서 아래 절차를 진행합니다.

### 1. private 네트워크 생성

Horizon 좌측 메뉴에서 [네트워크] > [네트워크]로 이동한 뒤 [네트워크 생성]을 클릭합니다.

- 네트워크 이름: `private`
- 서브넷 이름: `private-subnet`
- 네트워크 주소: `10.0.0.0/26`

### 2. 라우터 생성

[네트워크] > [라우터]로 이동한 뒤 [라우터 생성]을 클릭합니다.

- 라우터 이름: `router1`
- 외부 네트워크: `public`

### 3. 라우터에 private-subnet 연결

생성한 `router1` 상세 페이지로 이동합니다.

[인터페이스] 탭에서 [인터페이스 추가]를 클릭한 뒤 `private-subnet`을 선택합니다.

이렇게 구성하면 `public` 네트워크와 `private` 네트워크가 라우터를 통해 연결되며, 인스턴스가 외부 네트워크와 통신할 수 있는 NAT 구성이 완료됩니다.

---

## OpenStack CLI를 이용한 기본 리소스 조회 실습

DevStack 설치 후, OpenStack CLI를 사용하여 리소스를 간단히 조회할 수 있습니다.

### 1. 인증 환경 설정

```bash
source openrc admin admin
```

### 2. 네트워크 조회

```bash
openstack network list
openstack network show <네트워크_이름_OR_ID>
```

예시:

```bash
openstack network show private
```

### 3. 서버(인스턴스) 조회

```bash
openstack server list
openstack server show <서버_이름_OR_ID>
```

### 4. 라우터 조회

```bash
openstack router list
openstack router show <라우터_이름_OR_ID>
```

예시:

```bash
openstack router show router1
```

---

## virsh를 이용한 OpenStack VM 관리

설치가 완료된 후, DevStack에서 생성된 가상 머신들을 `virsh` 명령어로 관리할 수 있습니다.

### 1. VM 리스트 확인

```bash
sudo virsh list --all
```

현재 libvirt가 관리하는 모든 VM을 확인할 수 있습니다. 실행 중인 VM과 중지된 VM이 함께 표시됩니다.

### 2. 특정 VM 상세 정보 확인

```bash
sudo virsh dominfo [도메인이름]
```

예시:

```bash
sudo virsh dominfo instance-00000001
```

특정 VM 도메인의 상세 정보를 조회합니다.

### 3. VM 콘솔 접속

```bash
sudo virsh console [도메인이름]
```

예시:

```bash
sudo virsh console instance-00000001
```

SSH 없이 직접 VM의 터미널로 접속할 수 있습니다.

> **참고:** 콘솔 접속 종료는 `Ctrl + ]` 키를 입력하여 종료할 수 있습니다.

---

## 설치 실패 시 재시도 방법

설치가 중간에 실패한 경우, 가능하면 VM 스냅샷을 되돌리거나 새 VM에서 다시 진행하는 것을 권장합니다.

기존 VM에서 정리 후 다시 진행하려면 아래 명령어를 사용할 수 있습니다.

```bash
cd /opt/stack/devstack
./unstack.sh || true
./clean.sh || true

sudo rm -f /usr/local/bin/privsep-helper
sudo rm -rf /opt/stack/devstack
```

이후 `stack` 사용자로 다시 접속하여 DevStack을 다시 다운로드합니다.

```bash
sudo -u stack -i
cd /opt/stack
git clone -b unmaintained/2023.1 https://opendev.org/openstack/devstack
cd devstack
```

그다음 `local.conf`를 다시 작성하고 `FORCE=yes ./stack.sh` 또는 `./stack.sh`로 설치를 재시도합니다.

---

## 참고 사항

- DevStack 환경은 재부팅 시 초기화될 수 있으니, 중요한 변경 사항은 별도로 백업해두세요.
- `unmaintained/2023.1` 브랜치는 실습 재현성을 위한 선택이며, 운영 환경용 OpenStack 설치 방법이 아닙니다.
- 최신 master 브랜치는 Ubuntu/Python/패키지 의존성 조합에 따라 설치 오류가 발생할 수 있으므로 이 가이드의 기본 설치 절차에서는 사용하지 않습니다.
