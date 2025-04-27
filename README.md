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

#### (1) IP 확인 및 편집기 설치

```bash
sudo apt install net-tools
sudo apt install vim
```

#### (2) local.conf 파일 생성 및 수정

```bash
cp ./samples/local.conf local.conf
vim local.conf
```

#### (3) local.conf 파일 수정 내용

```bash
ADMIN_PASSWORD=stack
DATABASE_PASSWORD=stack
RABBIT_PASSWORD=stack
SERVICE_PASSWORD=stack

HOST_IP=192.168.x.x  # ifconfig 명령어를 통해 확인한 IP 입력
```

---

### 5. DevStack 설치 실행

```bash
./stack.sh
```

> **주의:** 설치 과정은 다소 시간이 소요될 수 있습니다. 설치 중 오류가 발생하면 로그를 분석하여 문제를 해결해야 합니다.

---

## 참고 사항

- 설치 완료 후 Horizon 대시보드에 접속하려면 브라우저에서 `http://<HOST_IP>/dashboard`로 접근하세요.
- DevStack 환경은 재부팅 시 초기화될 수 있으니, 중요한 변경 사항은 별도로 백업해두세요.
