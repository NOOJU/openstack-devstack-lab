# DevStack 설치 가이드 

테스트, 학습 환경으로 구성하기에 적합한 **DevStack**을 통해 **OpenStack**을 설치하는 과정을 정리합니다.

---

## 설치 환경

- 가상화 소프트웨어: **Oracle VirtualBox**
- OS: **Ubuntu 22.04 LTS Server**
- 자원 설정:
  - 메모리: **6GB (6144 MB)**
  - 디스크: **40GB**
  - CPU: **4 Core**

---

## 설치 절차

### 1. Repository Update 및 필수 패키지 설치

```bash
sudo apt update
sudo apt install python3 python3-pip virtualenv git -y
```

### 2. stack 사용자 생성 및 권한 설정

```bash
sudo useradd -s /bin/bash -d /opt/stack -m stack
sudo chmod +x /opt/stack
echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
sudo -u stack -i
```

### 3. DevStack 다운로드

```bash
git clone https://opendev.org/openstack/devstack
cd devstack
```

### 4. local.conf 설정

#### (1) local.conf 파일 생성 및 수정

```bash
cp ./samples/local.conf local.conf
vim local.conf
```

#### (2) local.conf 파일 수정 내용

```bash
ADMIN_PASSWORD=stack
DATABASE_PASSWORD=stack
RABBIT_PASSWORD=stack
SERVICE_PASSWORD=stack

HOST_IP=192.168.x.x  # ip a 명령어를 통해 확인한 2번째 IP 입력
```

---

### 5. DevStack 설치 실행

```bash
./stack.sh
```

> **주의:** 설치 과정은 다소 시간이 소요될 수 있습니다. 설치 중 오류가 발생하면 로그를 분석하여 문제를 해결해야 합니다.

---

## OpenStack CLI를 이용한 기본 리소스 조회 실습

DevStack 설치 후, OpenStack CLI를 사용하여 리소스를 간단히 조회할 수 있습니다.

### 1. 인증 환경 설정

```bash
source openrc admin admin
```

### 2. 네트워크 리스트 확인

```bash
openstack network list
```

### 3. 특정 네트워크 상세 정보 확인

```bash
openstack network show network1100
```

### 4. 서버(인스턴스) 리스트 확인

```bash
openstack server list
```

### 5. 특정 서버 상세 정보 확인

```bash
openstack server show vm1100-1
```

### 6. 라우터 리스트 확인

```bash
openstack router list
```

---

## virsh를 이용한 OpenStack VM 관리

설치가 완료된 후, DevStack에서 생성된 가상 머신들을 `virsh` 명령어로 관리할 수 있습니다.

### 1. VM 리스트 확인

```bash
sudo virsh list --all
```
➔ 현재 libvirt가 관리하는 모든 VM (실행 중/중지된 것 포함) 리스트 출력

### 2. 특정 VM 상세 정보 확인

```bash
sudo virsh dominfo [도메인이름]
```
예시:
```bash
sudo virsh dominfo instance-00000001
```
➔ 특정 VM(도메인)의 상세 정보를 조회

### 3. VM 콘솔 접속

```bash
sudo virsh console [도메인이름]
```
예시:
```bash
sudo virsh console instance-00000001
```
➔ SSH 없이 직접 VM의 터미널로 접속

> **참고:** 콘솔 접속 종료는 `Ctrl + ]` 키를 입력하여 종료할 수 있습니다.

---

## 참고 사항

- 설치 완료 후 Horizon 대시보드에 접속하려면 브라우저에서 `http://<HOST_IP>/dashboard`로 접근하세요.
- DevStack 환경은 재부팅 시 초기화될 수 있으니, 중요한 변경 사항은 별도로 백업해두세요.
