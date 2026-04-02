---
title: "리눅스 작업 스케쥴링: cron, at"
description: "리눅스 작업 스케쥴링: cron, at"
date: 2026-04-01
update: 2026-04-01
tags:
  - Linux
  - RHEL
series:
---


## systemd 의 스케줄링 구조
---

현대 RHEL에서는 서비스 관리의 중심이  
systemd 이다.
- 서비스 실행 → `systemctl`
- 로그 확인 → `journalctl`
- 리소스 관리 → cgroup

스케줄링은?
→ 여전히 **cron + at**이 기본 도구로 사용된다.

**cron/at은 systemd와 별개의 도구가 아니라,  systemd 서비스로 관리되는 스케줄러이다.**  (`crond.service`, `atd.service`)

> systemd는 “스케줄러 자체”를 관리하고,  
> cron/at은 “작업 실행 정책”을 담당한다.


## cron: 주기 작업 스케줄러
---
### 개념

**cron**은 리눅스에서 가장 대표적인 **주기적 작업 스케줄러**다.

- 반복 작업에 최적화
- 시간 기반 실행
- 백그라운드 데몬(`crond`) 형태로 동작

### 핵심 구조

- 작업 등록 단위: `crontab`
- 실행 주체: `crond` (systemd가 관리)

### 중요한 운영 포인트

> cron은 “조용히 실패(silent failure)”할 수 있다.

대표적인 원인:
- 환경 변수 미설정 (`PATH`)
- 권한 문제
- 스크립트 내부 에러
- 출력이 버려짐

따라서 실무에서는 반드시 다음을 고려해야 한다.
- 로그 파일 리다이렉션
- 에러 출력 포함 (`2>&1`)
- 모니터링 연계

### crontab 기본 명령어

```bash
# 현재 등록된 작업 확인
crontab -l

# 작업 편집
crontab -e
```

### crontab 포맷

```
* * * * * command
│ │ │ │ │
│ │ │ │ └ 요일 (0-7, 일요일=0 또는 7)
│ │ │ └ 월 (1-12)
│ │ └ 일 (1-31)
│ └ 시 (0-23)
└ 분 (0-59)
```

### 예시

```bash
0 3 * * * /opt/myapp/backup.sh >> /var/log/myapp-backup.log 2>&1
```

- 매일 새벽 3시 실행
- 표준 출력 + 에러 로그를 파일에 기록


## at: 단발성 작업 스케줄러
---
### 개념

**at**은 **한 번만 실행되는 작업 을 예약할 때 사용한다.

- cron: 반복 작업
- at: 단발 작업

### 사용 예시

```bash
at 02:00
> /opt/myapp/run-once-job.sh
> Ctrl+D
```

### 주요 명령어

```bash
# 예약 목록 확인
atq

# 작업 삭제
atrm <job_id>
```

### 동작 구조

- 데몬: `atd`
- systemd 서비스로 관리됨

## systemd 관점에서의 정리

|구성 요소|역할|systemd 관계|
|---|---|---|
|cron|반복 작업 스케줄링|`crond.service`|
|at|단발 작업 스케줄링|`atd.service`|
|systemd|전체 서비스 관리|상위 제어|

구조적으로 보면:

```
systemd
 ├── crond (cron scheduler)
 └── atd   (at scheduler)
```

## systemd timer와의 관계

systemd 환경에서는 cron 대신  
**systemd timer**를 사용하는 방식도 존재한다.

- `.service` + `.timer` 조합
- journal과 통합된 로그
- 의존성 관리 가능

하지만 실무에서는:

- 기존 호환성 → cron 사용
- 단순 작업 → cron이 더 직관적


## 정리
---

- 운영 환경에서는 **반복 작업 자동화는 필수**
- cron은 여전히 가장 널리 사용되는 스케줄러
- at은 단발성 작업에 적합
- systemd는 이들을 **서비스 단위로 관리**