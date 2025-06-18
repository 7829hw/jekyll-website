---
title: elec462-lecture-07-chap9
author: khw
date: 2025-06-18
categories: [Exam, SystemProgramming]
tags: [2025_first_semester_final_exam]
pin: true
math: true
mermaid: true
---

# 프로그래밍 가능한 셸: 셸 변수와 환경

## 📚 개요

이 강의에서는 Unix/Linux 셸이 단순한 명령어 실행기를 넘어서 **완전한 프로그래밍 언어**로 기능하는 방법을 학습합니다. 셸 스크립트, 변수 관리, 환경 설정, 그리고 입출력 리다이렉션의 핵심 개념들을 다룹니다.

---

## 🎯 학습 목표

- **셸 프로그래밍의 기본 원리** 이해
- **명령줄 파싱**과 토큰화 과정 학습
- **제어 구조**(if-then-else) 구현 방법
- **지역 변수와 전역 변수** 차이점과 활용
- **환경변수**의 역할과 관리 방법
- **입출력 리다이렉션** 메커니즘 이해

---

## 🔧 1. 셸 프로그래밍 기초

### 셸의 이중적 역할
- **프로그램 실행기**: 다른 프로그램들을 실행
- **프로그래밍 언어**: 스크립트를 통한 복잡한 작업 자동화

### 셸 스크립트란?
```bash
# script0.sh 예제
# this is called script0
# it runs some commands
ls
echo the current date/time is
date
echo my name is
whoami
```

**셸 스크립트**는 명령어들의 배치(batch)로, 순차적으로 실행되는 명령어 모음입니다.

### 스크립트 실행 방법
1. **셸에 인자로 전달**: `bash script0.sh`
2. **실행 권한 설정**: `chmod +x script0.sh && ./script0.sh`

---

## 🧩 2. 명령줄 파싱 (Command-Line Parsing)

### 파싱의 개념
- **파싱(Parsing)**: 문자열을 토큰으로 나누는 과정
- **토큰(Token)**: 하나의 정보 단위, "단어"
- **구분자(Delimiter)**: 토큰을 분리하는 문자

### C언어 예제: strtok 함수
```c
char phrase[] = "the music made it hard to concentrate";
char *token = strtok(phrase, " ");
while (token != NULL) {
    printf("%s\n", token);
    token = strtok(NULL, " ");
}
```

### 파서의 구조
1. **어휘 분석(Lexical Analysis)**: 입력을 토큰으로 분해
2. **구문 분석(Syntactic Analysis)**: 토큰 구조를 문법에 따라 분석

---

## 🎮 3. 제어 구조 (Control Flow)

### if-then-else 구문
```bash
if date | grep Fri
then
    echo time for backup. Insert tape and press enter
    read x
    tar cvf /dev/tape /home
fi
```

### 동작 원리
1. **조건 명령어 실행**: `if` 다음의 명령어 실행
2. **종료 상태 확인**: 
   - `exit(0)` = 성공 → `then` 블록 실행
   - `exit(≠0)` = 실패 → `else` 블록 실행
3. **블록 종료**: `fi`로 if 구문 종료

### 구현 핵심 포인트
- 셸은 키워드(`if`, `then`, `fi`)를 감지
- 조건에 따라 명령어 실행을 제어
- **영역(Region) 기반 해석**:
  - `neutral`: 일반 영역
  - `want_then`: then 대기 상태
  - `then_block`: then 실행 영역
  - `else_block`: else 실행 영역

---

## 📦 4. 셸 변수 시스템

### 변수 사용법
```bash
age=7                    # 변수 할당 (공백 없음!)
echo $age               # 변수 참조 ($ 필수)
echo $age+$age          # 문자열 연산: "7+7"
echo $((age+age))       # 산술 연산: "14"
read name               # 사용자 입력
echo hello, $name       # 변수 보간
```

### 두 가지 변수 타입

#### 🏠 지역 변수 (Local Variables)
- **범위**: 현재 셸 세션에만 제한
- **상속**: 자식 프로세스에 전달되지 않음
- **용도**: 스크립트 내부 계산, 임시 저장

#### 🌍 환경 변수 (Environment Variables)
- **범위**: 모든 자식 프로세스에 전달
- **상속**: `fork()`로 생성된 자식이 자동 상속
- **용도**: 시스템 설정, 프로그램 설정 전달

### 변수 연산 명령어

| 연산 | 구문 | 설명 |
|------|------|------|
| 할당 | `var=value` | 공백 없음 주의 |
| 참조 | `$var` | $ 기호 필수 |
| 삭제 | `unset var` | 변수 제거 |
| 입력 | `read var` | 표준입력에서 값 읽기 |
| 목록 | `set` | 모든 변수 표시 |
| 전역화 | `export var` | 환경변수로 변환 |

---

## 🌐 5. 환경변수 (Environment Variables)

### 환경변수의 목적
- **사용자 선호도 저장**: 홈 디렉토리, 사용자명
- **프로그램 설정**: 터미널 타입, 에디터 선택
- **시스템 구성**: PATH, LANG 등

### 주요 환경변수들
- **HOME**: 사용자 홈 디렉토리 경로
- **PATH**: 실행파일 검색 경로 목록
- **USER/USERNAME**: 현재 사용자명
- **TERM**: 터미널 타입
- **LANG**: 언어 설정

### 환경변수 관리 명령어
```bash
# 환경변수 보기
env                     # 모든 환경변수 출력
echo $HOME             # 특정 변수 값 출력

# 환경변수 설정
export VAR=value       # 변수 생성과 동시에 환경변수화
VAR=value              # 지역 변수 생성
export VAR             # 기존 변수를 환경변수로 전환
```

### C언어에서 환경변수 접근
```c
// 방법 1: getenv() 함수 사용
char *home = getenv("HOME");
if (home) printf("Home: %s\n", home);

// 방법 2: environ 전역 변수 사용
extern char **environ;
for (char **env = environ; *env; env++) {
    printf("%s\n", *env);
}
```

### 환경변수 상속 메커니즘
1. **부모 프로세스**: 환경변수 설정
2. **fork()**: 자식 프로세스가 환경변수 복사본 상속
3. **exec()**: 새 프로그램이 상속받은 환경변수 유지

---

## 🔄 6. 입출력 리다이렉션 (I/O Redirection)

### 표준 스트림 개념
모든 Unix/Linux 프로그램은 **3개의 표준 스트림**을 사용합니다:

| 스트림 | 파일 디스크립터 | 용도 | 방향 |
|--------|----------------|------|------|
| **stdin** | 0 | 표준 입력 | 읽기 |
| **stdout** | 1 | 표준 출력 | 쓰기 |
| **stderr** | 2 | 표준 오류 | 쓰기 |

### 리다이렉션 연산자
```bash
# 입력 리다이렉션
sort < data.txt         # data.txt에서 입력 받기

# 출력 리다이렉션
who > userlist.txt      # 출력을 파일로 저장
echo "hello" >> log.txt # 파일에 추가

# 조합 사용
cat < input.txt > output.txt  # 입출력 모두 리다이렉션
```

### stdin을 파일에 연결하는 방법

#### 방법 1: Close-then-Open
```c
close(0);                    // stdin 닫기
open("file", O_RDONLY);      // 파일 열기 → fd 0 할당
```

#### 방법 2: Open-Dup2-Close
```c
int fd = open("file", O_RDONLY);  // 파일 열기
dup2(fd, 0);                     // fd를 stdin(0)에 복사
close(fd);                       // 원본 fd 닫기
```

### 리다이렉션 구현 과정
1. **fork()**: 자식 프로세스 생성
2. **리다이렉션 설정**: 자식에서 파일 디스크립터 조작
3. **exec()**: 새 프로그램 실행 (파일 디스크립터 유지)

---

## 🏗️ 7. 미니 셸 구현 핵심

### 주요 구성 요소

#### 1. 명령어 파싱
```c
void parse_command(char *line, char **args) {
    char *token = strtok(line, " \t\n");
    int i = 0;
    while (token != NULL && i < MAX_ARGS - 1) {
        args[i++] = token;
        token = strtok(NULL, " \t\n");
    }
    args[i] = NULL;
}
```

#### 2. 내장 명령어 처리
```c
// 변수 할당: VAR=value
if (strchr(args[0], '=') && args[0][0] != '=') {
    char *eq = strchr(args[0], '=');
    *eq = '\0';
    set_local_var(args[0], eq + 1);
    return;
}

// export 명령어
if (strcmp(args[0], "export") == 0) {
    // export 로직
}
```

#### 3. 변수 확장
```c
void expand_variables(char* line) {
    // $VAR 형태의 변수를 실제 값으로 치환
    // 예: "echo $HOME" → "echo /home/user"
}
```

#### 4. 제어 구조 처리
```c
if (strncmp(line, "if ", 3) == 0) {
    handle_if_block(line, input);
}
```

### 변수 저장 시스템
```c
typedef struct {
    char name[64];
    char value[256];
} Variable;

Variable local_vars[MAX_VARS];      // 지역 변수
Variable global_vars[MAX_VARS];     // 전역 변수
```

---

## 🎯 8. 핵심 개념 정리

### 프로그래밍 패러다임
- **해석형 언어**: 셸은 스크립트를 실시간으로 해석하고 실행
- **명령어 기반**: 각 줄이 하나의 명령어 또는 제어 구조
- **프로세스 중심**: fork/exec 모델을 기반으로 한 프로그램 실행

### 변수 관리 철학
- **지역성 원칙**: 기본적으로 지역 변수, 필요시에만 전역화
- **명시적 전역화**: `export` 명령어로 의도적 환경변수 생성
- **상속 모델**: 부모→자식으로 환경변수 자동 전달

### I/O 추상화
- **파일 디스크립터 기반**: 모든 I/O가 숫자 기반 추상화
- **유닉스 철학**: "모든 것은 파일이다"
- **투명한 리다이렉션**: 프로그램은 리다이렉션을 모름

---

## 🔍 9. 고급 주제들

### 시스템 호출 활용
- **getenv()**: 환경변수 읽기
- **setenv()**: 환경변수 설정 
- **dup2()**: 파일 디스크립터 복제
- **waitpid()**: 자식 프로세스 상태 확인

### 오류 처리 전략
- **종료 상태 코드**: 0=성공, 비영=실패
- **표준 오류 스트림**: 오류 메시지 분리
- **견고한 파싱**: 잘못된 입력에 대한 방어

### 성능 고려사항
- **메모리 관리**: 동적 할당보다 정적 배열 선호
- **프로세스 비용**: fork/exec의 오버헤드 인식
- **문자열 처리**: 효율적인 토큰화와 변수 확장

---

## 📝 실습 포인트

1. **간단한 셸 구현**: 기본 명령어 실행부터 시작
2. **변수 시스템 추가**: 지역/전역 변수 구분 구현
3. **제어 구조 구현**: if-then-else 파싱과 실행
4. **리다이렉션 지원**: 입출력 스트림 조작
5. **오류 처리 강화**: 예외 상황에 대한 견고한 대응

이러한 구현을 통해 운영체제의 프로세스 관리, 파일 시스템, 그리고 시스템 프로그래밍의 핵심 개념들을 실제로 경험할 수 있습니다.
