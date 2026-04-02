---
title: "리눅스 자원 제어: cgroup"
description: "리눅스 자원 제어: cgroup"
date: 2026-04-01
update: 2026-04-01
tags:
  - Linux
  - RHEL
series:
---

이전 글에서 `systemctl`은 **systemd가 관리하는 서비스 단위**를 다루는 도구라고 정리했다.  
그렇다면 **systemd는 서비스를 실제로 어떻게 관리할까?**

`systemctl status`를 보면 그 힌트를 찾을 수 있다.

	CGroup: /system.slice/sshd.service

이 경로는 sshd 서비스에 속한 프로세스들이 `/system.slice/sshd.service`라는 cgroup 안에 배치되어 있다는 뜻이다.

> 즉, systemd는 **서비스를 cgroup 단위로 묶고, 그 단위로 자원을 통제한다.**


이 글에서는 그 cgroup이 무엇인지, 왜 systemd와 함께 이해해야 하는지, 그리고 RHEL 9/10 기준으로 실제 시스템에서는 어떤 구조로 보이는지 정리해보겠다.

 이 글에서는 아래 흐름으로 cgroup을 정리한다.
 - cgroup이란?
 - systemd와 cgroup
 - 실제 구조

## cgroup 이란?
---
cgroup(Control Group)은 **리눅스 커널 기능**으로,

> 여러 프로세스를 하나의 그룹으로 묶고,  
> 그 그룹 단위로 자원을 제한하고 격리하는 메커니즘이다.

#### 1) 핵심 특징
- 프로세스를 **그룹 단위로 관리**
- 그룹 단위로 **자원 할당 / 제한 / 모니터링**
- 커널 레벨에서 동작 → 매우 강력한 제어

예를 들어, “이 서비스는 메모리를 512MB까지만 써라”, “이 서비스는 CPU를 30%까지만 써라” 같은 정책을 **커널이 직접 적용**할 수 있다.

#### 2) cgroup의 필요성
운영 환경에서는 다음 문제가 반드시 발생한다.
- 특정 프로세스가 CPU를 과도하게 사용
- 메모리 leak으로 시스템 전체 장애
- 여러 서비스가 서로 자원을 침범
cgroup은 이런 상황에서 필요한 **자원 통제**를 지원한다.

#### cgroup이 제공하는 기능
---
#### 1) 자원 제한 (Limit)

- CPU 사용량 제한
- 메모리 사용량 제한
- 디스크 I/O 제한
- 네트워크 대역폭 제한
#### 2) 자원 격리 (Isolation)

- 하나의 서비스가 문제를 일으켜도 다른 서비스에 영향이 가지 않도록 분리

#### 3) 우선순위 제어 (Priority)

- 특정 서비스에 CPU를 더 할당 가능

#### 4) 모니터링 (Accounting)

- 그룹 단위로 자원 사용량 추적 가능

#### cgroup 계층 구조
---

cgroup은 프로세스와 유사하게 **계층 구조**를 가진다.

#### 프로세스 계층 구조와의 공통점
- 계층 구조 (tree)
- 부모 → 자식 관계 존재
- 일부 속성 상속

#### 프로세스 계층 구조와의 차이점
|구분|프로세스|cgroup|
|---|---|---|
|구조|단일 트리|여러 트리 가능|
|목적|실행 흐름|자원 관리|
|소속|하나의 부모|여러 계층 가능|
> 프로세스 트리는 “누가 누구를 만들었는가”
> cgroup 트리는 “누가 어떤 자원 정책 아래 묶여 있는가” 를 나타낸다.

## systemd와 cgroup
---
systemd는 다음과 같은 운영 기능을 담당한다.

- 서비스 시작과 종료
- 상태 관리
- 자동 재시작
- 의존성 관리
- 로그 연계
- **자원 제어**

여기서 마지막 항목인 **자원 제어**를 가능하게 해주는 기반이 바로 cgroup이다.

> systemd는 서비스를 실행할 때 **그 서비스 전용 cgroup을 만들고, 관련 프로세스를 모두 그 안에 넣는다**.  
> 그리고 **필요한 자원 정책을 그 cgroup에 적용**한다.

정리하자면 **systemd 내부 동작** 은 다음과 같다.
```
1. 서비스 실행  
2. 해당 서비스용 cgroup 생성  
3. 관련 프로세스를 그 cgroup에 배치  
4. 자원 정책 적용
```

그래서 `systemctl status`에 CGroup 항목이 보이는 것이다. 

## cgroup 실제 구조
---


#### RHEL에서의 cgroup 버전

|버전|특징|
|---|---|
|cgroup v1|리소스별 분리된 구조 (복잡)|
|cgroup v2|단일 통합 구조 (RHEL 9 기본)|
cgroup v1은 리소스 종류마다 구조가 분리되어 있었다. 예를 들어 CPU용 계층, 메모리용 계층이 따로 존재했다.

> cgroup v2는 **모든 자원을 하나의 트리에서 관리**한다.

**v2의 장점**
- 구조 단순화
- systemd와 완전 통합
- 운영 관점에서 일관성 확보


#### 실제 cgroup 구조

cgroup v2의 기본 경로는 다음과 같다.

	/sys/fs/cgroup/

이 아래에 systemd가 관리하는 단위들이 계층적으로 나타난다.

 - slice
	 - systemd가 만든 **상위 논리 그룹**
	 - 자원 분배의 큰 틀을 나누는 단위
	- `sshd.service`가 `system.slice` 아래에 있다는 것은 **시스템 서비스 영역에 속한 unit**이라는 뜻

- service
	
	- 실제 서비스 단위
	- systemd unit과 1:1 대응

- scope

	- 외부에서 시작된 프로세스를 systemd가 관리 범위 안에 포함시킬 때 사용하는 단위


#### controller: 실제 자원 제어 지점

cgroup 디렉터리는 내부에는 실제 자원 제한과 상태 확인에 사용되는 파일들이 있다.
이 파일들이 controller 인터페이스 역할을 한다.

대표 controller

|controller|역할|
|---|---|
|cpu|CPU 사용량|
|memory|메모리 제한|
|io|디스크 I/O|
|pids|프로세스 수|

**실제 예시**
	/sys/fs/cgroup/system.slice/sshd.service/

여기 안에는:

- `cpu.max`
- `memory.max`
- `memory.current`

> “얼마까지 허용할 것인가”와 “지금 얼마나 쓰고 있는가”를 파일 단위로 확인할 수 있다.


#### systemd 설정 → cgroup으로 변환

systemd 설정은 내부적으로 cgroup 파일 값으로 변환된다.

systemd 설정(Unit File)

```
[Service]  
CPUQuota=20%  
MemoryMax=512M
```

내부적으로 다음의 파일들로 변환된다.
- `cpu.max`
- `memory.max`

> cgroup을 사람이 다루기 쉬운 형태로 감싸는 관리 계층

#### cgroup의 동적 변경

cgroup은 실행 중에도 정책 변경이 가능하다.

**systemd 방식**

	systemctl set-property nginx.service CPUQuota=30%

→ 즉시 적용 (재시작 없음)

**low-level 직접 수정**

	echo 30000 100000 > cpu.max

#### 컨테이너와 cgroup

컨테이너 기술의 핵심은 다음 커널 기능의 조합이다.
**cgroup + namespace**
- namespace → 격리
- cgroup → 자원 제한

> 컨테이너 = 격리된 환경 안에서, 제한된 자원 정책 아래 실행되는 프로세스 집합