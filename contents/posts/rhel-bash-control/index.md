---
title: "Bash Script: 조건문, 반복문, 함수"
description: "Bash Script: 조건문, 반복문, 함수"
date: 2026-04-15
update: 2026-04-15
tags:
  - Linux
  - RHEL
series: "Red Hat Enterprise Linux 스터디"
---

이 글에서는 Bash 스크립트의 다음 내용들을 다룬다.

- [조건문](https://jiu-jung.github.io/rhel-bash-control/#조건문)
- [반복문](https://jiu-jung.github.io/rhel-bash-control/#반복문)
- [함수](https://jiu-jung.github.io/rhel-bash-control/#함수)
- [명령어 치환](https://jiu-jung.github.io/rhel-bash-control/#명령어-치환)
- [운영 자동화에서 자주 쓰이는 요소](https://jiu-jung.github.io/rhel-bash-control/#운영-자동화에서-자주-쓰이는-요소)

## 조건문

### 기본 구조

```bash
if 조건1 ; then
    명령1
elif 조건2 ; then
    명령2
else
	명령3
fi
```

조건 자리에는 다음 두 가지가 올 수 있다.

- `[ ... ]`, `[[ ... ]]` 같은 비교식/검사식
- 일반 명령어 또는 파이프라인   
	종료 코드가 `0`이면 참, 아니면 거짓

<br>

### 비교식/검사식

비교식/검사식을 을 사용할 때는 다음 두 가지를 주의해야 한다.

- 앞뒤 공백이 꼭 필요하다.
- 문자열 변수는 큰따옴표로 감싸는 것이 안전하다.
  
운영 스크립트에서 자주 사용하는 조건식은 크게 아래 세 가지다.

#### 1. 파일 존재 여부
파일이나 디렉터리가 실제로 있는지 검사할 때 사용한다.

```bash
if [ -f backup.log ]; then
    echo "file exists"
fi
```

```bash
if [ -d /var/log ]; then
    echo "log directory exists"
fi
```

자주 쓰는 파일 관련 테스트:

- `-f` : 일반 파일이 존재하는가
- `-d` : 디렉터리가 존재하는가
- `-e` : 파일 또는 디렉터리가 존재하는가

<br>

#### 2. 문자열 비교

문자열 값이 같은지, 비어 있는지 등을 검사할 때 사용한다.

```bash
if [ "$env" = "prod" ]; then
    echo "production mode"
fi
```

```bash
if [ -z "$name" ]; then
    echo "name is empty"
fi
```

자주 쓰는 문자열 테스트:

- `=` : 문자열이 같은가
- `!=` : 문자열이 다른가
- `-z` : 문자열 길이가 0인가
- `-n` : 문자열 길이가 0이 아닌가

<br>

#### 3. 숫자 비교

숫자는 문자열 비교와 다른 연산자를 사용한다.

```bash
count=10

if [ "$count" -gt 5 ]; then
    echo "count is greater than 5"
fi
```

```bash
if [ "$port" -eq 8080 ]; then
    echo "default port"
fi
```

자주 쓰는 숫자 비교 연산자:

- `-eq` : 같다
- `-ne` : 다르다
- `-gt` : 크다
- `-ge` : 크거나 같다
- `-lt` : 작다
- `-le` : 작거나 같다



<br>
<br>

## 반복문
---


### `for`

```bash
for item in 값1 값2 값3
do
    실행할 명령
done
```

예: 파일 목록 순회

```bash
for file in app.log db.log auth.log
do
    echo "checking $file"
done
```

예: 현재 디렉터리의 `.log` 파일 순회

```bash
for file in *.log
do
    echo "processing $file"
done
```

<br>

### `while`

```bash
while [ 조건식 ]
do
    실행할 명령
done
```

예시

```bash
count=1

while [ "$count" -le 5 ]
do
    echo "$count"
    count=$((count + 1))
done
```

<br>

### 운영에서 자주 쓰이는 반복문 예시

#### 여러 파일 처리

```bash
for file in *.log
do
    grep "ERROR" "$file"
done
```

각 로그 파일에서 `ERROR`를 검색한다.

<br>

#### 여러 프로세스 이름 검사

```bash
for proc in nginx sshd cron
do
    ps -ef | grep "[${proc:0:1}]${proc:1}"
done
```

여러 서비스 상태를 한 번에 점검할 수 있다.

<br>

#### 여러 로그에 대한 반복 작업

```bash
for file in /var/log/*.log
do
    echo "===== $file ====="
    grep "failed" "$file"
done
```
로그 파일이 여러 개일 때 같은 필터를 반복 적용하는 데 유용하다.


<br>
<br>

## 함수
---

### 기본 구조

함수 정의

```bash
hello() {
    echo "hello"
}
```

호출

```bash
hello
```
- 함수 이름만 쓰면 된다.

<br>

### 예시

정의:

```bash
check_file() {
    if [ -f "$1" ]; then
        echo "$1 exists"
    else
        echo "$1 missing"
    fi
}
```
- `$1` :  전달한 첫 번째 인자

호출:

```bash
check_file "/etc/passwd"
check_file "nofile.txt"
```

<br>

### 반복되는 작업을 함수로 분리하기

같은 로직이 두 번 이상 나오기 시작하면 함수로 분리하는 것이 좋다.

예를 들어 여러 파일의 존재 여부를 검사한다고 할 때, 아래처럼 함수로 분리하는 것이 좋다.

```bash
check_file() {
    if [ -f "$1" ]; then
        echo "[OK] $1 exists"
    else
        echo "[FAIL] $1 missing"
    fi
}

check_file "/etc/passwd"
check_file "/etc/hosts"
check_file "./config.env"
```

<br>
<br>

## 명령어 치환
---

Bash Script에서는 명령어 실행 결과를 변수에 넣고,  
그 값을 이후 조건문이나 반복문에서 활용하는 경우가 많다.  
이때 사용하는 문법이 **명령어 치환(command substitution)** 이다.

<br>

명령어의 실행 결과를 변수에 저장할 때는 `$(...)`를 사용한다.

```bash
name=$(whoami)  
today=$(date)  
count=$(grep -c "ERROR" app.log)
```

<br>

예시
```bash
count=$(grep -c "ERROR" app.log)
```
- `grep -c "ERROR" app.log`를 실행한 뒤,  
- 그 출력 결과를 `count` 변수에 저장한다.


<br>
<br>

## 운영 자동화에서 자주 쓰이는 요소
---

### 변수화

고정값을 직접 코드에 반복해서 쓰기보다 변수로 분리하는 것이 좋다.

```bash
LOG_DIR="/var/log"
KEYWORD="ERROR"
```

이렇게 하면 나중에 경로나 검색어를 바꾸기 쉽다.

```bash
grep "$KEYWORD" "$LOG_DIR/app.log"
```

특히 다음은 변수화하는 경우가 많다.

- 경로
- 파일명
- 포트 번호
- 환경 이름
- 반복 대상 목록

<br>


### 조건 검사 후 실행

무조건 실행하기보다, 조건을 확인하고 실행해야 예상치 못한 실패를 줄일 수 있다.

```bash
if [ -f "$LOG_DIR/app.log" ]; then
    grep "$KEYWORD" "$LOG_DIR/app.log"
else
    echo "log file not found"
fi
```

- 파일이 있을 때만 읽기
- 디렉터리가 있을 때만 이동
- 인자가 주어졌을 때만 특정 작업 수행
- 이전 명령이 성공했을 때만 다음 단계 진행


<br>

### 실패 시 종료


가장 단순한 방식은 `exit 1`이다. 아래처럼 필수 파일이 없을 때 바로 종료시킬 수 있다.

```bash
if [ ! -f config.env ]; then
    echo "config.env not found"
    exit 1
fi
```


명령 직후 종료 코드를 검사할 수도 있다.

```bash
cp source.txt backup.txt

if [ $? -ne 0 ]; then
    echo "copy failed"
    exit 1
fi
```

<br>

### 로그 남기기

자동화 스크립트는 나중에 무엇이 실행됐는지 추적할 수 있어야 한다.  
그래서 중요한 단계마다 메시지를 남기는 경우가 많다.

```bash
echo "[INFO] backup started"
echo "[INFO] checking log files"
echo "[ERROR] config file missing"
```

파일에 저장할 수도 있다.

```bash
echo "[INFO] backup started" >> backup.log
```


<br>

### 실행 결과 확인

스크립트는 실행만 하는 것이 아니라, 실행 결과를 확인하고 필요하면 다음 동작을 정해야 한다.

예를 들어 로그 검색 결과 개수를 세고 판단할 수 있다.

```bash
count=$(grep -c "ERROR" app.log)

if [ "$count" -gt 0 ]; then
    echo "errors found: $count"
else
    echo "no errors"
fi
```

서비스가 실행 중인지 검사할 수도 있다.

```bash
ps -ef | grep '[n]ginx' > /dev/null

if [ $? -eq 0 ]; then
    echo "nginx is running"
else
    echo "nginx is not running"
fi
```