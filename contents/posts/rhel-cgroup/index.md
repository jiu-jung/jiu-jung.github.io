---
title: "리눅스 자원 제어: cgroup"
description: "리눅스 자원 제어: cgroup"
date: 2026-04-01
update: 2026-04-01
tags:
  - Linux
  - RHEL
series: "Red Hat Enterprise Linux 스터디"
---

멀티 유저, 멀티 프로세스 운영 환경에서는 서비스가 얼마나 CPU와 메모리를 사용할 수 있는지,  
그리고 하나의 서비스 문제가 다른 서비스까지 번지지 않도록 어떻게 통제할 것인지가 중요하다.

따라서 프로세스를 실행 대상뿐만 아니라, **자원 관리의 대상**으로도 다뤄야 한다.

이 글에서는 다음 흐름으로 리눅스 자원 제어를 정리한다.
- `cgroup`: 자원 관리의 핵심
- `systemd`와 `cgroup`
- 실제 `cgroup` 구조 (RHEL 기준)
- 자원 제어의 실제 동작

<br>

## `cgroup`: 자원 관리의 핵심

---

### 1)`cgroup`이란?

`cgroup`은 **Control Group**의 약자로,  
- **여러 프로세스**를 하나의 그룹으로 묶고
- **그룹 단위로 자원을 제한하고 추적**
- **리눅스 커널 기능**이다.

<br>

예를 들어 다음과 같은 정책을 적용할 수 있다.
- 이 서비스는 메모리를 512MB까지만 사용
- 이 서비스는 CPU를 일정 비율까지만 사용
- 이 서비스는 생성 가능한 프로세스 수를 제한
- 이 서비스의 디스크 I/O를 제한

> 리눅스는 **프로세스 그룹 단위**로 자원을 제어한다.

<br>


### 2) `cgroup`이 제공하는 기능

`cgroup`은 크게 네 가지 기능을 제공한다.

#### 1. 자원 제한 (Limit)

각 그룹에 사용할 수 있는 자원 한도를 설정할 수 있다.

- CPU 사용량 제한
- 메모리 사용량 제한
- 디스크 I/O 제한
- 프로세스 수 제한

#### 2. 자원 격리 (Isolation)

하나의 서비스가 문제를 일으켜도 
다른 서비스까지 영향을 주지 않도록 분리할 수 있다.

#### 3. 우선순위 제어 (Priority)

특정 서비스에 더 많은 CPU 비중을 주거나, 
덜 중요한 작업의 우선순위를 낮출 수 있다.

#### 4. 사용량 추적 (Accounting)

그룹 단위로 CPU, 메모리, I/O 사용량을 추적할 수 있다.


> `cgroup`은 **제한 + 격리 + 우선순위 + 모니터링**을 함께 제공하는 자원 관리 구조다.

<br>


## `systemd`와 cgroup

---

현대 리눅스에서 서비스 관리는 `systemd`가 담당한다.

`systemd`는 다음과 같은 운영 기능을 제공한다.

- 서비스 시작과 종료
- 상태 관리
- 자동 재시작    
- 의존성 관리
- 로그 연계
- **자원 제어**

이 중 **자원 제어**를 실제로 가능하게 해주는 기반이 바로 `cgroup`이다.

<br>

`systemd`는 서비스를 실행할 때,  
- **해당 서비스 전용 `cgroup`** 을 만들고
- **관련 프로세스를 모두 그 안에 넣는다.**  
- 그 다음 **필요한 자원 정책을 그 `cgroup`에 적용**한다.

<br>

정리하면 내부 동작은 다음과 같다.

```text
1. 서비스 실행
2. 해당 서비스용 cgroup 생성
3. 관련 프로세스를 그 cgroup에 배치
4. 자원 정책 적용
```

<br>

그래서 `systemctl status`를 보면 다음과 같은 항목이 보인다.

![](./source/systemctl_status.png)

```text
CGroup: /system.slice/sshd.service
```

이 의미는 다음과 같다.

- `sshd.service`에 속한 프로세스들이
- `/system.slice/sshd.service`라는 cgroup 아래 묶여 있고
- 그 단위로 자원 관리가 이루어진다는 뜻이다

> `systemd`는 서비스를 실행 단위뿐만 아니라 **자원 관리 단위**로도 다룬다.

<br>


## 실제 cgroup 구조 (RHEL 기준)

---

### 1) cgroup v1과 v2

RHEL에서는 cgroup 버전에 따라 구조가 다르다.

|버전|특징|
|---|---|
|cgroup v1|리소스별로 분리된 구조|
|cgroup v2|하나의 통합된 구조|

cgroup v1에서는 CPU, 메모리, I/O 등이 각각 별도의 계층으로 나뉘어 있었다.  
즉 CPU용 트리, 메모리용 트리가 따로 존재했다.

반면 cgroup v2는 **모든 자원을 하나의 트리에서 통합 관리**한다.

이 구조는 다음 장점이 있다.
- 구조가 단순함
- systemd와의 통합이 쉬움
- 자원 관리 방식이 일관됨

RHEL 9+에서는 이 cgroup v2 구조가 기본이다.

<br>


### 2) 기본 경로

cgroup v2의 기본 경로는 다음과 같다.

```bash
/sys/fs/cgroup/
```

이 아래에서 systemd가 관리하는 서비스와 자원 구조를 확인할 수 있다.

<br>


### 3) 주요 단위: `slice`, `service`, `scope`

cgroup 기본 경로 아래에는 systemd가 만든 단위들이 계층적으로 나타난다.

#### 1. `slice`

`slice`는 systemd가 만든 **상위 논리 그룹**이다.  
자원 분배의 큰 틀을 나누는 단위라고 볼 수 있다.

예를 들어 `system.slice` 아래에 있다는 것은  
해당 unit이 시스템 서비스 영역에 속한다는 뜻이다.

#### 2. `service`

`service`는 실제 서비스 단위다.  
보통 systemd의 service unit과 1:1로 대응한다.

예:
- `sshd.service`
- `nginx.service`

#### 3. `scope`

`scope`는 서비스 외부에서 시작된 프로세스를 systemd 관리 범위 안에 포함시킬 때 사용하는 단위다.

<br>

정리하면 다음과 같다.

> - `slice`: 큰 그룹
> - `service`: 실제 서비스 단위
> - `scope`: 외부 프로세스 관리 단위    

<br>


### 4) controller: 실제 자원 제어 지점

실제 자원 제한은 `controller` 파일을 통해 이루어진다.

**cgroup 디렉터리** 안에는 자원 제한과 상태 확인에 사용하는 `controller`파일들이 있다.  
이 파일들이 **실제 자원 제어 인터페이스** 역할을 한다.

대표적인 controller는 다음과 같다.

|controller|역할|
|---|---|
|`cpu`|CPU 사용량 제어|
|`memory`|메모리 제한 및 사용량 확인|
|`io`|디스크 I/O 제어|
|`pids`|프로세스 수 제한|

<br>

예를 들어 다음 경로를 보자.

```bash
/sys/fs/cgroup/system.slice/sshd.service/
```

이 안에는 다음과 같은 파일이 있을 수 있다.

- `cpu.max`
    
- `memory.max`
    
- `memory.current`
    
- `pids.max`
    

이 파일들의 의미는 다음과 같다.

- `*.max` : 어디까지 허용할 것인가
    
- `*.current` : 지금 얼마나 쓰고 있는가
    

<br>

실제로 출력해보면 아래처럼 값이 나온다.
![](./source/controller.png)

<br>

> cgroup은 **파일 기반** 인터페이스를 통해 자원 정책을 적용하고 상태를 보여주는 구조다.

<br>


## 자원 제어의 실제 동작

---

### 1) `systemd` 설정이 `cgroup`으로 반영되는 방식

운영자는 보통 **cgroup 파일을 직접 수정하기보다,** **systemd 설정을 통해 자원 정책을 적용**한다.  
예를 들어 **unit file**에 다음과 같이 설정할 수 있다.

```ini
[Service]
CPUQuota=20%
MemoryMax=512M
```
이 설정은 내부적으로 cgroup 관련 값으로 반영된다.  
예를 들어 다음과 같은 controller 파일과 연결된다.

- `cpu.max`
    
- `memory.max`
    
<br>

즉 `systemd`는 cgroup을 사람이 더 쉽게 다룰 수 있도록 감싼 **관리 계층**이라고 볼 수 있다.

> 운영자는 익숙한 unit 설정 문법으로 정책을 작성하고,  
> 실제 적용은 systemd가 cgroup에 반영하는 방식이다.

<br>


### 2) 실행 중 자원 정책 변경

cgroup 기반 자원 정책은 실행 중에도 변경할 수 있다.
다음 두가지 방식으로 자원 정책을 변경할 수 있다.

#### 1. systemd 방식

```bash
systemctl set-property nginx.service CPUQuota=30%
```

이 명령은 서비스 재시작 없이 자원 정책을 즉시 적용한다.

#### 2. low-level
```bash
echo "30000 100000" > cpu.max
```
cgroup 파일을 직접 수정할 수도 있다.

> 다만 운영에서는 직접 파일을 수정하기보다, 보통 `systemd`를 통해 관리하는 편이 더 안전하고 일관적이다.

<br>


### 3) 컨테이너와 `cgroup`

컨테이너 기술의 핵심은 두 가지 리눅스 커널 기능의 조합이다.

- `namespace`: 격리
    
- `cgroup`: 자원 제한
    
<br>

각 역할은 다음과 같다.

- `namespace`는 서로 다른 환경처럼 보이게 만드는 기능
    
- `cgroup`은 사용할 수 있는 자원을 제한하는 기능
    

따라서 컨테이너는 본질적으로 **격리된 환경 안에서, 제한된 자원 정책 아래 실행되는 프로세스 집합**이다.

<br>


## 정리

---

- 리눅스에서 자원 관리는 프로세스 그룹 단위로 이루어진다.
- `cgroup`은 프로세스를 그룹 단위로 묶어 자원을 제어하는 커널 기능이다.
- `systemd`는 서비스를 실행할 때 해당 서비스용 cgroup을 만들고 자원 정책을 적용한다.
- `cgroup` 디렉터리의 `controller` 파일들을 통해 실제 제한과 사용량 확인이 이루어진다.
- 운영자가 `unit file`을 통해 자원 정책을 설정하면 `systemd`가 `controller` 파일에 매핑하여 구현한다.
