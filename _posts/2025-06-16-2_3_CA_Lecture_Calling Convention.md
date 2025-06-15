---
title: 2_3_CA_Lecture_Calling Convention
author: khw
date: 2025-06-16
categories: [Exam, ComputerOrganization]
tags: [2025_first_semester_final_exam]
pin: true
math: true
mermaid: true
---

# 🎯 RISC-V Calling Convention 벼락치기 핵심 정리

## 🔥 **시험 핵심 포인트 (반드시 암기!)**

### **📋 RISC-V 33개 레지스터 완전 정복**

| 레지스터 | ABI 이름 | 역할 | **Saver** | **🎯 시험포인트** |
|---------|---------|------|-----------|------------------|
| x0 | zero | 하드웨어 고정 0 | - | **항상 0, 변경 불가** |
| x1 | ra | 리턴 주소 | **Caller** | **JAL이 PC+4 저장** |
| x2 | sp | 스택 포인터 | **Callee** | **스택 탑 가리킴** |
| x3 | gp | 전역 포인터 | - | 시험에 안나옴 |
| x4 | tp | 스레드 포인터 | - | 시험에 안나옴 |
| x5-x7 | t0-t2 | 임시 레지스터 | **Caller** | **함수 호출시 파괴됨** |
| x8 | s0/fp | 저장 레지스터/프레임포인터 | **Callee** | **스택 프레임 시작점** |
| x9 | s1 | 저장 레지스터 | **Callee** | **보존됨** |
| x10-x11 | a0-a1 | 함수 인자/리턴값 | **Caller** | **첫번째 인자 & 리턴값** |
| x12-x17 | a2-a7 | 함수 인자 | **Caller** | **8개 인자까지 레지스터로** |
| x18-x27 | s2-s11 | 저장 레지스터 | **Callee** | **함수 호출에도 보존** |
| x28-x31 | t3-t6 | 임시 레지스터 | **Caller** | **함수 호출시 파괴됨** |

---

## 🚨 **Register Saving Convention (초중요!)**

### **🔴 Caller Saved 레지스터**
- **누가**: Caller(호출자)가 저장
- **언제**: 함수 호출 **전에** 필요한 값을 스택에 저장
- **어떤 레지스터**: `a0-a7, t0-t6, ra` (x1, x5-x7, x10-x17, x28-x31)
- **특징**: 함수 호출 후 **값이 파괴될 수 있음**
- **🎯 암기**: "호출하기 전에 caller가 살려야 할 것들"

### **🔵 Callee Saved 레지스터**
- **누가**: Callee(피호출자)가 저장
- **언제**: 함수 시작 시 사용할 레지스터를 스택에 저장
- **어떤 레지스터**: `s0-s11, sp` (x2, x8-x9, x18-x27)
- **특징**: 함수 호출 후에도 **값이 보존됨**
- **🎯 암기**: "함수가 끝나도 saved(저장된) 상태"

---

## 📚 **Calling Convention 핵심 과정**

### **1단계: 함수 호출 준비 (Caller)**
```assembly
# 1. Caller Saved 레지스터 백업 (필요시)
addi sp, sp, -8
sd t0, 0(sp)      # t0을 스택에 저장

# 2. 인자 준비 (최대 8개까지 레지스터로)
addi a0, x20, 0   # 첫 번째 인자
addi a1, x21, 0   # 두 번째 인자

# 3. 함수 호출
jal ra, function  # PC+4를 ra에 저장하고 function으로 점프
```

### **2단계: 함수 실행 시작 (Callee)**
```assembly
function:
# 1. 스택 공간 할당
addi sp, sp, -16

# 2. Callee Saved 레지스터 백업
sd ra, 8(sp)      # 리턴 주소 저장
sd s0, 0(sp)      # s0 저장

# 3. 프레임 포인터 설정 (선택사항)
addi s0, sp, 16   # fp를 현재 프레임 시작점으로
```

### **3단계: 함수 실행 종료 (Callee)**
```assembly
# 1. 리턴값 설정
addi a0, result, 0  # 결과를 a0에

# 2. Callee Saved 레지스터 복원
ld s0, 0(sp)
ld ra, 8(sp)

# 3. 스택 포인터 복원
addi sp, sp, 16

# 4. 리턴
jalr x0, 0(ra)    # ra의 주소로 점프 (caller로 돌아감)
```

### **4단계: 함수 호출 완료 (Caller)**
```assembly
# 1. 리턴값 사용
# a0에 결과값이 들어있음

# 2. Caller Saved 레지스터 복원
ld t0, 0(sp)
addi sp, sp, 8
```

---

## 🏗️ **Stack Frame 구조 (위에서 아래로)**

```
높은 주소
┌─────────────────┐
│   Caller Frame  │
├─────────────────┤ ← Previous sp
│  Argument Build │ (9번째 인자부터)
├─────────────────┤
│  Return Address │ (ra 백업)
├─────────────────┤
│ Saved Registers │ (Callee saved)
├─────────────────┤
│ Local Variables │
├─────────────────┤ ← Current sp
│   (할당 공간)    │
└─────────────────┘
낮은 주소
```

**🎯 Frame Pointer (fp/s0)**
- 현재 함수의 스택 프레임 시작점을 가리킴
- sp는 계속 변하지만 fp는 함수 실행 중 고정
- 지역변수 접근할 때 편리: `ld x5, -8(fp)`

---

## ⚡ **핵심 명령어**

### **JAL (Jump And Link)**
```assembly
jal ra, function
```
- **동작**: `ra = PC + 4`, `PC = function의 주소`
- **의미**: 다음 명령어 주소를 ra에 저장하고 function으로 점프
- **🎯 암기**: "call function"와 동일

### **JALR (Jump And Link Register)**
```assembly
jalr x0, 0(ra)
```
- **동작**: `PC = ra + 0`, `x0 = PC + 4` (하지만 x0는 변경 안됨)
- **의미**: ra의 주소로 점프 (리턴)
- **🎯 암기**: "ret"와 동일

---

## 📝 **Leaf vs Non-Leaf Procedure**

### **🍃 Leaf Procedure (단말 함수)**
- **특징**: 다른 함수를 호출하지 않음
- **최적화**: ra를 스택에 저장할 필요 없음 (덮어쓰이지 않으므로)

```assembly
leaf:
    addi sp, sp, -24    # 스택 공간 할당
    sd x5, 16(sp)       # 사용할 레지스터들 백업
    sd x6, 8(sp)
    sd x20, 0(sp)
    
    # 함수 로직
    add x5, a0, a1      # g + h
    add x6, a2, a3      # i + j  
    sub x20, x5, x6     # (g+h) - (i+j)
    addi a0, x20, 0     # 결과를 a0에
    
    ld x20, 0(sp)       # 레지스터 복원
    ld x6, 8(sp)
    ld x5, 16(sp)
    addi sp, sp, 24     # 스택 복원
    jalr x0, 0(ra)      # 리턴
```

### **🌳 Non-Leaf Procedure (비단말 함수)**
- **특징**: 다른 함수를 호출함
- **필수**: ra를 반드시 스택에 저장 (덮어쓰이므로)

```assembly
fact:
    addi sp, sp, -16    # 스택 공간 할당
    sd ra, 8(sp)        # 🔥 ra 저장 필수!
    sd a0, 0(sp)        # n 저장
    
    # Base case 체크
    addi t0, a0, -1
    bge t0, zero, L1
    addi a0, zero, 1    # return 1
    addi sp, sp, 16
    jalr x0, 0(ra)
    
L1: addi a0, a0, -1     # n-1
    jal ra, fact        # 🔥 재귀 호출 (ra가 덮어쓰임!)
    
    addi t1, a0, 0      # fact(n-1) 결과
    ld a0, 0(sp)        # n 복원
    ld ra, 8(sp)        # 🔥 ra 복원
    addi sp, sp, 16
    mul a0, a0, t1      # n * fact(n-1)
    jalr x0, 0(ra)
```

---

## 🎯 **시험 출제 예상 포인트**

### **1. 레지스터 분류 문제**
- Q: a0는 Caller saved인가 Callee saved인가?
- A: **Caller saved** (함수 호출 후 값이 변할 수 있음)

### **2. 스택 프레임 그리기**
- 함수 호출 과정에서 스택이 어떻게 변하는지
- sp, fp의 위치 변화

### **3. 코드 빈칸 채우기**
```assembly
leaf:
    addi sp, sp, ____    # -24 (스택 할당)
    sd x5, ____(sp)      # 16(sp) (레지스터 저장)
    # ... 함수 로직 ...
    ld x5, ____(sp)      # 16(sp) (레지스터 복원)
    addi sp, sp, ____    # 24 (스택 복원)
    jalr ____, ____(____)  # x0, 0(ra) (리턴)
```

### **4. 호출 과정 설명**
- JAL 명령어의 동작 과정
- 왜 Non-leaf에서는 ra를 저장해야 하는가?

### **5. Call Chain 분석**
- 재귀 함수 호출시 스택의 변화
- 각 단계에서 sp, fp의 위치

---

## 💡 **핵심 암기 팁**

### **🔥 절대 틀리면 안되는 것들**
1. **a0-a1**: 함수 인자 + 리턴값
2. **ra**: 리턴 주소 (JAL이 PC+4 저장)
3. **sp**: 스택 포인터 (스택 탑)
4. **s0-s11**: Callee saved (함수 호출해도 보존)
5. **t0-t6**: Caller saved (함수 호출시 파괴)

### **🎯 헷갈리기 쉬운 부분**
- **sp vs fp**: sp는 변하지만 fp는 고정
- **JAL vs JALR**: JAL은 호출, JALR은 리턴
- **Leaf vs Non-leaf**: Non-leaf는 ra 저장 필수
- **Caller vs Callee saved**: 누가 언제 저장하는가?

### **📝 문제 접근법**
1. **함수 타입 확인**: Leaf인가 Non-leaf인가?
2. **사용 레지스터 파악**: Caller saved인가 Callee saved인가?
3. **스택 크기 계산**: 저장할 레지스터 개수 × 8바이트
4. **저장/복원 순서**: 저장의 역순으로 복원

---

## 🚀 **최종 점검 체크리스트**

- [ ] 33개 레지스터 역할과 Saver 구분
- [ ] JAL/JALR 명령어 동작 원리
- [ ] Caller saved vs Callee saved 차이
- [ ] Stack Frame 구조와 sp/fp 역할
- [ ] Leaf vs Non-leaf 함수 차이점
- [ ] 함수 호출 4단계 과정
- [ ] 스택 할당/해제 방법
- [ ] 재귀 호출시 주의사항 (ra 저장)

**🎯 이것만 완벽히 알면 Calling Convention은 끝!**