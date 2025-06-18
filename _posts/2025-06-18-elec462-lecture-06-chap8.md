---
title: elec462-lecture-06-chap8
author: khw
date: 2025-06-18
categories: [Exam, SystemProgramming]
tags: [2025_first_semester_final_exam]
pin: true
math: true
mermaid: true
---

# ELEC462 Chapter 8: 프로세스와 프로그램 - Shell 구현하기

## 📚 학습 목표

### 핵심 개념과 기술
- **Linux Shell의 동작 원리** 이해
- **프로세스 모델** 개념 습득
- **프로그램 실행** 메커니즘 학습
- **프로세스 생성** 방법 이해
- **부모-자식 프로세스 간 통신** 방법 학습

### 중요 시스템 콜
- `fork()` - 프로세스 생성
- `exec()` - 프로그램 실행
- `wait()` - 자식 프로세스 대기
- `exit()` - 프로세스 종료

---

## 🏗️ 프로세스 vs 프로그램

### 프로그램 (Program/Executable)
```
프로그램 = 파일에 저장된 기계어 명령어들의 집합
```
- 디스크에 저장된 정적인 코드
- 소스코드를 컴파일하여 생성된 바이너리

### 프로세스 (Process)
```
프로세스 = 실행 중인 프로그램의 인스턴스
```
- **메모리 공간** + **동적 설정**으로 구성
- 각 프로세스는 고유한 **PID(Process ID)** 보유
- 독립적인 메모리 주소 공간 사용

### 메모리 레이아웃
```
고주소  ┌─────────────┐
       │  argc, argv │  명령행 인수
       ├─────────────┤
       │    Stack    │  임시 데이터 저장
       ├─────────────┤
       │     ↕️      │  성장 방향
       ├─────────────┤
       │    Heap     │  동적 메모리 할당
       ├─────────────┤
       │Uninitialized│  초기화되지 않은 데이터
       │    Data     │
       ├─────────────┤
       │ Initialized │  초기화된 전역 변수
       │    Data     │
       ├─────────────┤
       │    Text     │  실행 가능한 코드
저주소  └─────────────┘
```

---

## 🔍 프로세스 탐색: ps 명령어

### 기본 사용법
```bash
ps          # 현재 터미널의 프로세스들
ps -a       # 모든 사용자의 프로세스들
ps -l       # 자세한 정보 표시
```

### ps 출력 정보
- **PID**: 프로세스 ID
- **PPID**: 부모 프로세스 ID
- **TTY**: 연결된 터미널
- **TIME**: CPU 사용 시간
- **CMD**: 실행 중인 명령어
- **S**: 프로세스 상태 (S=sleeping, R=running, T=stopped)

---

## 🐚 Shell의 역할과 기능

### Shell이란?
```
Shell = 운영체제 서비스에 접근하기 위한 사용자 인터페이스
```

### Shell의 3가지 핵심 기능

#### 1. 프로그램 실행 (Program Execution)
- 사용자 명령어 해석
- 프로그램을 메모리에 로드하여 실행
- 프로그램 런처 역할

#### 2. 입출력 관리 (I/O Management)
```bash
ls > output.txt     # 출력 리다이렉션
cat < input.txt     # 입력 리다이렉션
ls | grep ".c"      # 파이프를 통한 프로세스 연결
```

#### 3. 프로그래밍 인터페이스 (Programmable Interface)
```bash
# 변수와 제어 구조 사용 가능
for i in {1..5}; do
    echo "Count: $i"
done
```

---

## ⚙️ Shell의 프로그램 실행 과정

### Shell의 메인 루프
```c
while (!end_of_input) {
    get_command();      // 명령어 입력 받기
    execute_command();  // 명령어 실행
    wait_for_finish();  // 완료까지 대기
}
```

### 실행 과정 상세
```
1. 사용자가 명령어 입력 (예: ls -l)
2. Shell이 새 프로세스 생성 (fork)
3. 새 프로세스에 프로그램 로드 (exec)
4. Shell이 프로세스 완료까지 대기 (wait)
5. 다음 명령어 대기
```

---

## 🔧 핵심 시스템 콜들

### 1. execvp() - 프로그램 실행

#### 기능
- 현재 프로세스 이미지를 새 프로그램으로 **완전 교체**
- 성공하면 원래 프로그램으로 돌아오지 않음

#### 사용법
```c
char *arglist[] = {"ls", "-l", NULL};
execvp("ls", arglist);
// 이 라인은 실행되지 않음 (성공시)
printf("execvp failed\n");
```

#### 명령행 인수 전달
```c
// "necho hello world" 명령어의 경우
argc = 3
argv[0] = "necho"
argv[1] = "hello" 
argv[2] = "world"
argv[3] = NULL     // 반드시 NULL로 종료
```

### 2. fork() - 프로세스 생성

#### 기능
- 호출 프로세스의 **거의 완전한 복사본** 생성
- 부모와 자식 프로세스가 **동일한 코드** 실행
- **분리된 메모리 공간** 사용

#### 반환값의 특징
```c
pid_t fork_result = fork();

if (fork_result == -1) {
    // 에러 발생
    perror("fork failed");
} else if (fork_result == 0) {
    // 자식 프로세스에서 실행
    printf("I am child, PID: %d\n", getpid());
} else {
    // 부모 프로세스에서 실행
    printf("I am parent, child PID: %d\n", fork_result);
}
```

#### fork() 후 프로세스 상태
```
실행 전:
┌─────────────┐
│   Parent    │
│    PID: A   │
└─────────────┘

실행 후:
┌─────────────┐    ┌─────────────┐
│   Parent    │    │    Child    │
│    PID: A   │    │    PID: B   │
│   return: B │    │   return: 0 │
└─────────────┘    └─────────────┘
```

### 3. wait() - 자식 프로세스 대기

#### 두 가지 주요 기능
1. **동기화**: 자식 프로세스가 끝날 때까지 부모 프로세스 일시 정지
2. **통신**: 자식의 종료 상태 정보 수집

#### 사용법
```c
int status;
pid_t child_pid = wait(&status);

printf("Child %d finished\n", child_pid);
printf("Exit status: %d\n", WEXITSTATUS(status));
```

#### 자식 상태 정보 해석
```
16비트 상태 정보:
┌─────────┬─────────┬─────────┐
│Exit Code│Signal #│Core Dump│
│ (8비트) │ (7비트) │ (1비트) │
└─────────┴─────────┴─────────┘
```

### 4. exit() vs _exit() - 프로세스 종료

#### exit() 라이브러리 함수
```c
exit(status);  // 정리 작업 후 종료
```
1. 모든 스트림 버퍼 flush
2. atexit() 등록 함수들 실행
3. 기타 정리 작업 수행
4. _exit() 시스템 콜 호출

#### _exit() 시스템 콜
```c
_exit(status);  // 즉시 종료
```
- 모든 파일 디스크립터 닫기
- 메모리 해제
- 커널 데이터 구조 정리
- 부모에게 SIGCHLD 신호 전송

---

## 💻 Shell 구현 실습

### 첫 번째 Shell: psh1.c
```c
// 간단한 단일 명령 실행 쉘
int main() {
    char *arglist[MAXARGS+1];
    int numargs = 0;
    char argbuf[ARGLEN];
    char *makestring();
    
    numargs = 0;
    while (numargs < MAXARGS) {
        printf("Arg[%d]? ", numargs);
        if (fgets(argbuf, ARGLEN, stdin) && *argbuf != '\n') {
            arglist[numargs++] = makestring(argbuf);
        } else {
            if (numargs > 0) {
                arglist[numargs] = NULL;
                execute(arglist);
                numargs = 0;
            }
        }
    }
    return 0;
}
```

### 두 번째 Shell: psh2.c (개선된 버전)
```c
void execute(char *arglist[]) {
    int pid, exitstatus;
    
    pid = fork();  // 새 프로세스 생성
    
    switch(pid) {
        case -1:
            perror("fork failed");
            exit(1);
            
        case 0:  // 자식 프로세스
            execvp(arglist[0], arglist);
            perror("execvp failed");
            exit(1);
            
        default:  // 부모 프로세스
            while(wait(&exitstatus) != pid)
                ;  // 자식 완료까지 대기
            printf("child exited with status %d\n", 
                   exitstatus>>8);
    }
}
```

### Shell 동작 흐름도
```
Shell Process              Child Process
     │                           │
     ├──── fork() ──────────────→ │ (새 프로세스 생성)
     │                           │
     ├──── wait() ──────────┐     │
     │ (대기 상태)           │     ├──── execvp() ────→ Program
     │                      │     │ (프로그램 실행)
     │                      │     │
     │                      │     ├──── main() ───────→ Logic
     │                      │     │ (프로그램 로직)
     │                      │     │
     │                      │     └──── exit() ────────→ Terminate
     │                      │           (프로세스 종료)
     │ ←─────── 완료 ────────┘
     │ (대기 해제)
     └──── 다음 명령 대기
```

---

## 🚨 특수한 상황들

### 좀비 프로세스 (Zombie Process)
```
좀비 프로세스 = exit()는 호출했지만 부모가 wait()하지 않은 프로세스
```
- **defunct** 프로세스라고도 불림
- 커널에 종료 상태 정보가 남아있음
- 부모가 wait() 호출시 완전히 제거됨
- 메모리는 해제되었지만 프로세스 테이블 엔트리는 유지

### 신호 처리 (Signal Handling)
- **Ctrl+C**: SIGINT 신호 → 터미널의 모든 프로세스에 전송
- **SIGCHLD**: 자식 프로세스 종료시 부모에게 전송 (기본적으로 무시됨)

### 고아 프로세스 (Orphan Process)
- 부모가 자식보다 먼저 종료된 경우
- **init 프로세스**(PID 1)가 새로운 부모가 됨
- 계속 실행 가능

---

## 📋 중요 개념 정리

### 프로세스 관련 개념

| 개념 | 설명 | 예시 |
|------|------|------|
| **PID** | Process ID, 각 프로세스의 고유 식별자 | 1234 |
| **PPID** | Parent Process ID, 부모 프로세스의 PID | 1000 |
| **프로세스 상태** | 현재 프로세스의 실행 상태 | R(실행중), S(대기), T(정지) |
| **사용자 공간** | 프로그램과 데이터를 위한 메모리 영역 | vs 커널 공간 |

### 시스템 콜 비교

| 시스템 콜 | 목적 | 반환값 | 특징 |
|-----------|------|--------|------|
| **fork()** | 프로세스 복제 | 부모: 자식PID, 자식: 0 | 메모리 공간 복사 |
| **execvp()** | 프로그램 실행 | 실패시 -1 | 프로세스 이미지 교체 |
| **wait()** | 자식 대기 | 자식 PID | 동기화 + 상태 수집 |
| **exit()** | 프로세스 종료 | 없음 | 정리 후 종료 |

### Shell vs 일반 프로그램

| 특징 | Shell | 일반 프로그램 |
|------|-------|---------------|
| **실행 방식** | 연속적 루프 | 단일 실행 후 종료 |
| **사용자 입력** | 명령어 대기 | 프로그램 내 처리 |
| **프로세스 관리** | 자식 프로세스 생성/관리 | 단일 프로세스 |
| **I/O 처리** | 리다이렉션, 파이프 | 표준 I/O |

---

## 🎯 핵심 포인트

### 1. Shell의 본질
> **Shell = 프로세스와 프로그램을 관리하는 특별한 프로그램**
- 사용자와 운영체제 간의 중개자 역할
- fork → exec → wait 패턴의 반복

### 2. 프로세스 생성의 Unix 철학
> **"작은 프로그램들의 조합으로 복잡한 작업 수행"**
- 각 프로그램은 단일 작업에 특화
- 파이프와 리다이렉션을 통한 연결

### 3. 메모리 보호
> **각 프로세스는 독립된 메모리 공간 사용**
- 프로세스 간 간섭 방지
- 시스템 안정성 보장

### 4. 동기화의 중요성
> **부모-자식 프로세스 간 적절한 동기화 필요**
- wait() 없이는 좀비 프로세스 발생
- 리소스 누수 방지

---

## 🔬 실습 예제

### 간단한 명령어 실행기
```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork();
    
    if (pid == 0) {
        // 자식: ls -l 실행
        execlp("ls", "ls", "-l", NULL);
        perror("exec failed");
        return 1;
    } else {
        // 부모: 자식 완료 대기
        int status;
        wait(&status);
        printf("Child finished with status: %d\n", 
               WEXITSTATUS(status));
    }
    
    return 0;
}
```

### 다중 자식 프로세스 관리
```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    int num_children = 3;
    
    // 여러 자식 프로세스 생성
    for (int i = 0; i < num_children; i++) {
        if (fork() == 0) {
            printf("Child %d (PID: %d) working...\n", i, getpid());
            sleep(i + 1);  // 다른 시간만큼 작업
            printf("Child %d finished\n", i);
            return i;  // 각기 다른 종료 코드
        }
    }
    
    // 모든 자식 프로세스 완료 대기
    for (int i = 0; i < num_children; i++) {
        int status;
        pid_t child_pid = wait(&status);
        printf("Child PID %d exited with status %d\n", 
               child_pid, WEXITSTATUS(status));
    }
    
    return 0;
}
```

---

이 내용들은 Unix/Linux 시스템의 **프로세스 관리**와 **Shell 구현**의 핵심 원리를 다루며, 현대 운영체제의 기본 철학을 이해하는 데 매우 중요한 개념들입니다.
