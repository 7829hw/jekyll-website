---
title: elec462-lecture-08-threads
author: khw
date: 2025-06-18
categories: [Exam, SystemProgramming]
tags: [2025_first_semester_final_exam]
pin: true
math: true
mermaid: true
---

# ELEC462 스레드와 동기화 핵심 정리

## 1. 스레드(Thread) 기본 개념

### 스레드란?
- **단일 실행 프로세스의 새로운 추상화**
- 멀티스레드 프로그램은 **하나 이상의 실행 지점**을 가짐
- 여러 개의 프로그램 카운터(PC)를 가지면서 **같은 주소 공간을 공유**

### 스레드의 특징
- **각 스레드마다 독립적인 스택**을 가짐 (thread-local storage)
- **프로그램 카운터와 레지스터 세트**를 각각 보유
- 스레드 간 컨텍스트 스위치 시 **주소 공간은 동일하게 유지**

### 스레드를 사용하는 이유
1. **병렬성(Parallelism)**: 멀티코어 프로세서에서 성능 향상
2. **I/O 블로킹 방지**: 한 스레드가 I/O 처리 중일 때 다른 스레드가 CPU 활용

### 스레드 API 예시
```c
#include <pthread.h>

// 스레드 생성
pthread_create(&thread_id, NULL, thread_function, argument);

// 스레드 완료 대기
pthread_join(thread_id, NULL);
```

## 2. 동기화 문제와 경쟁 상태(Race Condition)

### 문제 상황
공유 변수에 대한 동시 접근으로 인한 **예측 불가능한 결과**:

```c
// 문제가 되는 코드
counter = counter + 1;

// 어셈블리 코드로 분해하면:
mov 0x8049a1c, %eax  // 메모리에서 값 읽기
add $0x1, %eax       // 1 증가
mov %eax, 0x8049a1c  // 메모리에 값 저장
```

### 핵심 용어
- **경쟁 상태(Race Condition)**: 스레드 실행 타이밍에 따라 결과가 달라지는 상황
- **임계 영역(Critical Section)**: 공유 자원에 접근하는 코드 부분
- **상호 배제(Mutual Exclusion)**: 한 번에 하나의 스레드만 임계 영역에 진입

## 3. 락(Lock) - 동기화 기본 도구

### 락의 기본 아이디어
```c
lock_t mutex;
lock(&mutex);
balance = balance + 1;  // 임계 영역
unlock(&mutex);
```

### 락의 의미론
- **available/unlocked**: 아무도 락을 보유하지 않음
- **acquired/locked**: 정확히 하나의 스레드가 락을 보유

### Pthread 뮤텍스 사용법
```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

pthread_mutex_lock(&lock);
// 임계 영역
pthread_mutex_unlock(&lock);
```

### 락 평가 기준
1. **상호 배제**: 락이 제대로 작동하는가?
2. **공정성**: 각 스레드가 공평하게 락을 획득할 기회를 얻는가?
3. **성능**: 락 사용으로 인한 오버헤드는 얼마인가?

### 하드웨어 지원: Test-and-Set
```c
int TestAndSet(int *ptr, int new) {
    int old = *ptr;  // 기존 값 읽기
    *ptr = new;      // 새 값 설정
    return old;      // 기존 값 반환
}

// 스핀락 구현
void lock(lock_t *lock) {
    while (TestAndSet(&lock->flag, 1) == 1)
        ; // 스핀 대기
}
```

### 스핀락의 문제점과 해결책
- **문제**: CPU 사이클 낭비, 기아 현상 가능
- **해결책 1**: `yield()` 시스템 콜로 CPU 양보
- **해결책 2**: 큐를 사용한 대기 (`park()`/`unpark()`)
- **해결책 3**: 2단계 락 (짧은 스핀 후 슬립)

## 4. 락을 이용한 데이터 구조

### 단순 카운터 (Thread-Safe)
```c
typedef struct {
    int value;
    pthread_mutex_t lock;
} counter_t;

void increment(counter_t *c) {
    pthread_mutex_lock(&c->lock);
    c->value++;
    pthread_mutex_unlock(&c->lock);
}
```

### 확장성 문제와 근사 카운터
**문제**: 단일 락으로 인한 병목 현상

**해결책**: 근사 카운터 (Approximate Counter)
- **로컬 카운터** (CPU 코어별) + **글로벌 카운터**
- 임계값(S)에 도달하면 로컬 → 글로벌로 병합
- **확장성 ↑**, **정확도 ↓** (트레이드오프)

```c
// 근사 카운터 구조체
typedef struct {
    int global;                    // 글로벌 카운터
    pthread_mutex_t glock;         // 글로벌 락
    int local[NUMCPUS];           // 로컬 카운터들
    pthread_mutex_t llock[NUMCPUS]; // 로컬 락들
    int threshold;                 // 병합 임계값
} counter_t;
```

## 5. 조건 변수(Condition Variables)

### 문제 상황
스레드가 **특정 조건이 참이 될 때까지 대기**해야 하는 경우

### 기본 API
```c
pthread_cond_t cond;
pthread_mutex_t mutex;

// 조건 대기
pthread_cond_wait(&cond, &mutex);

// 조건 신호
pthread_cond_signal(&cond);
```

### 사용 패턴 (부모-자식 동기화)
```c
// 자식 스레드 완료 대기 구현
int done = 0;
pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t c = PTHREAD_COND_INITIALIZER;

void thr_exit() {
    pthread_mutex_lock(&m);
    done = 1;
    pthread_cond_signal(&c);
    pthread_mutex_unlock(&m);
}

void thr_join() {
    pthread_mutex_lock(&m);
    while (done == 0)  // while 사용 중요!
        pthread_cond_wait(&c, &m);
    pthread_mutex_unlock(&m);
}
```

### 핵심 설계 요소
1. **상태 변수** (`done`): 조건 저장
2. **락**: 상태 변수 보호
3. **조건 변수**: 대기/신호 메커니즘

### 생산자-소비자 문제 (Producer-Consumer)

#### 문제 정의
- **생산자**: 데이터를 생성하여 버퍼에 저장
- **소비자**: 버퍼에서 데이터를 가져와 처리
- **버퍼**: 제한된 크기의 공유 자원

#### 최종 해결책
```c
cond_t empty, fill;  // 두 개의 조건 변수
mutex_t mutex;

void *producer(void *arg) {
    for (int i = 0; i < loops; i++) {
        pthread_mutex_lock(&mutex);
        while (count == MAX)           // 버퍼 가득참
            pthread_cond_wait(&empty, &mutex);
        put(i);
        pthread_cond_signal(&fill);    // 소비자에게 신호
        pthread_mutex_unlock(&mutex);
    }
}

void *consumer(void *arg) {
    for (int i = 0; i < loops; i++) {
        pthread_mutex_lock(&mutex);
        while (count == 0)             // 버퍼 비어있음
            pthread_cond_wait(&fill, &mutex);
        int tmp = get();
        pthread_cond_signal(&empty);   // 생산자에게 신호
        pthread_mutex_unlock(&mutex);
        printf("%d\n", tmp);
    }
}
```

#### 중요한 설계 원칙
- **두 개의 조건 변수 사용**: `empty`(생산자 대기), `fill`(소비자 대기)
- **while 루프 사용**: Mesa 의미론에서 안전성 보장
- **방향성 있는 신호**: 생산자는 소비자에게만, 소비자는 생산자에게만 신호

## 6. 일반적인 동시성 버그

### 6.1 비교착 상태 버그 (Non-Deadlock Bugs) - 74%

#### 원자성 위반 (Atomicity Violation)
**문제**: 원자적이어야 할 메모리 접근이 중간에 끊어짐

```c
// 버그 예시 (MySQL)
Thread1:
if (thd->proc_info) {        // 검사
    fputs(thd->proc_info, ...); // 사용 (중간에 다른 스레드가 개입!)
}

Thread2:
thd->proc_info = NULL;       // NULL로 설정
```

**해결책**: 락으로 원자성 보장
```c
pthread_mutex_lock(&lock);
if (thd->proc_info) {
    fputs(thd->proc_info, ...);
}
pthread_mutex_unlock(&lock);
```

#### 순서 위반 (Order Violation)
**문제**: 특정 실행 순서가 보장되지 않음

```c
Thread1:
void init() {
    mThread = PR_CreateThread(mMain, ...);  // 초기화
}

Thread2:
void mMain(...) {
    mState = mThread->State;  // mThread가 아직 초기화되지 않을 수 있음!
}
```

**해결책**: 조건 변수로 순서 강제
```c
// Thread1에서 초기화 완료 신호
pthread_mutex_lock(&mtLock);
mtInit = 1;
pthread_cond_signal(&mtCond);
pthread_mutex_unlock(&mtLock);

// Thread2에서 초기화 대기
pthread_mutex_lock(&mtLock);
while (mtInit == 0)
    pthread_cond_wait(&mtCond, &mtLock);
pthread_mutex_unlock(&mtLock);
```

### 6.2 교착 상태 (Deadlock) - 31%

#### 교착 상태 정의
두 스레드가 서로가 가진 락을 기다리며 **무한 대기**하는 상황

```c
Thread 1:           Thread 2:
lock(L1);          lock(L2);
lock(L2);          lock(L1);  // 교착 상태!
```

#### 교착 상태 발생 조건 (4가지 모두 충족 시)
1. **상호 배제**: 자원을 독점적으로 사용
2. **점유와 대기**: 자원을 가진 채로 다른 자원 대기
3. **비선점**: 자원을 강제로 빼앗을 수 없음
4. **순환 대기**: 스레드들이 순환적으로 자원 대기

### 6.3 교착 상태 방지 기법

#### 1. 순환 대기 방지 (가장 실용적)
**전체 락 순서**: 모든 락에 대해 전역적 순서 정의
```c
// 항상 L1 → L2 순서로 획득
lock(L1);
lock(L2);
// 작업 수행
unlock(L2);
unlock(L1);
```

#### 2. 점유와 대기 방지
**모든 락을 한 번에 획득**
```c
lock(prevention);  // 메타 락
lock(L1);
lock(L2);
unlock(prevention);
// 작업 수행
```

**단점**: 캡슐화 파괴, 동시성 감소

#### 3. 비선점 방지
**trylock() 사용**: 획득 실패 시 보유 락 해제
```c
top:
lock(L1);
if (trylock(L2) == -1) {
    unlock(L1);
    // 랜덤 지연 (라이브락 방지)
    goto top;
}
```

#### 4. 교착 상태 회피
**스케줄링을 통한 방지**: 미래의 락 요청 정보를 활용하여 안전한 스케줄링

#### 5. 탐지 및 복구
**주기적 탐지**: 자원 그래프에서 순환 탐지 후 시스템 재시작

## 핵심 원칙과 모범 사례

### 락 사용 시 주의사항
1. **최소한의 락 사용**: 성능과 복잡성 고려
2. **일관된 락 순서**: 교착 상태 방지
3. **짧은 임계 영역**: 락 보유 시간 최소화
4. **적절한 락 세분화**: 동시성과 복잡성의 균형

### 조건 변수 사용 원칙
1. **항상 while 루프 사용**: Mesa 의미론 대응
2. **상태 변수 필수**: 조건 저장 및 검사
3. **방향성 있는 신호**: 관련 스레드에게만 신고
4. **락과 함께 사용**: 상태 변수 보호

### 버그 방지 전략
1. **코드 리뷰**: 동시성 문제 사전 발견
2. **단위 테스트**: 다양한 스케줄링 시나리오 테스트
3. **정적 분석 도구**: 교착 상태 가능성 분석
4. **락 프리 자료구조**: 가능한 경우 락 없는 설계 고려

## 결론

스레드와 동기화는 현대 시스템 프로그래밍의 핵심입니다. **올바른 동기화 기법의 사용**이 안전하고 효율적인 멀티스레드 프로그램 개발의 열쇠입니다. 락과 조건 변수를 적절히 활용하고, 일반적인 동시성 버그 패턴을 이해함으로써 **robust한 동시성 프로그램**을 작성할 수 있습니다.
