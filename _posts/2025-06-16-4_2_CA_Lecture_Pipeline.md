---
title: 4_2_CA_Lecture_Pipeline
author: khw
date: 2025-06-16
categories: [Exam, ComputerOrganization]
tags: [2025_first_semester_final_exam]
pin: true
math: true
mermaid: true
---

# 🚀 컴퓨터구조 파이프라인 벼락치기 완전정복

## 📝 **시험 핵심 암기사항** ⭐⭐⭐
- **파이프라인 5단계**: IF → ID → EX → MEM → WB
- **3가지 해저드**: Structural, Data, Control
- **CPI 공식**: CPI = 1 + Pipeline stall cycles per instruction
- **Speedup 공식**: Speedup = Pipeline depth / (1 + Stall cycles per instruction)

---

## 🔥 **1. 파이프라인 기본 개념** ⭐⭐⭐

### **파이프라인이란?**
- 여러 명령어를 **동시에 처리**하는 CPU 구현 기법
- **Instruction-Level Parallelism (ILP)** 활용
- 이상적인 경우 **CPI = 1** 달성 가능

### **5단계 파이프라인** (RISC-V 기준)
1. **IF (Instruction Fetch)**: 메모리에서 명령어 가져오기
2. **ID (Instruction Decode)**: 명령어 해독 + 레지스터 읽기
3. **EX (Execute)**: 연산 수행 or 주소 계산
4. **MEM (Memory)**: 메모리 접근 (load/store)
5. **WB (Write Back)**: 결과를 레지스터에 쓰기

### **파이프라인 다이어그램** (시험 단골!)
```
Clock:     1  2  3  4  5  6  7  8  9
Inst i:   IF ID EX MEM WB
Inst i+1:    IF ID EX  MEM WB
Inst i+2:       IF ID  EX  MEM WB
Inst i+3:          IF  ID  EX  MEM WB
Inst i+4:             IF  ID  EX  MEM WB
```

---

## ⚠️ **2. 파이프라인 해저드 (Hazards)** ⭐⭐⭐

### **해저드 정의**
- 다음 명령어를 다음 사이클에 시작할 수 없게 하는 상황
- **CPI > 1**이 되어 성능 저하 발생

### **3가지 해저드 유형**

#### **🔧 Structural Hazard (구조적 해저드)**
- **원인**: 하드웨어 자원 충돌
- **예시**: 명령어 메모리와 데이터 메모리가 같은 메모리 사용
- **해결**: 자원 복제 (I-cache, D-cache 분리)

#### **📊 Data Hazard (데이터 해저드)**
- **원인**: 명령어가 이전 명령어 결과에 의존
- **예시**:
  ```assembly
  sub x2, x19, x3    # x2 값 계산
  add x19, x2, x1    # x2 값 필요! (의존성)
  ```

#### **🎯 Control Hazard (제어 해저드)**
- **원인**: 분기 명령어로 인한 PC 변경
- **문제**: EX 단계까지 가야 분기 여부 확인 가능
- **지연**: 3 사이클 stall 발생

---

## 🛠️ **3. 데이터 해저드 해결방법** ⭐⭐⭐

### **💨 Forwarding (포워딩/바이패싱)**
- **핵심**: 결과가 계산되자마자 바로 전달
- **장점**: 대부분의 데이터 해저드 해결
- **구현**: 추가 하드웨어 연결 필요

#### **Load-Use 데이터 해저드** (중요!)
```assembly
ld x1, 0(x2)      # x1 값을 메모리에서 로드
sub x4, x1, x5    # x1 값 바로 사용
```
- **문제**: 포워딩으로도 해결 불가 ("시간을 되돌릴 수 없음!")
- **해결**: **1 사이클 stall** 필수

### **🧊 Pipeline Stall (파이프라인 정지)**
- **방법**: 의존성 해결될 때까지 파이프라인 정지
- **단점**: 성능 저하 (CPI 증가)

### **📋 Compiler Scheduling**
- **방법**: 컴파일러가 명령어 순서 재배치
- **목표**: 의존성 있는 명령어들 사이에 간격 생성

---

## 🎲 **4. 제어 해저드와 분기 예측** ⭐⭐⭐

### **기본 해결 방법들**

#### **📈 Static Prediction (정적 예측)**
- **Predict Not-Taken**: 항상 분기 안 함으로 예측
- **문제점**: taken 비율이 70%인데 최적화가 잘못됨

#### **🔮 Dynamic Prediction (동적 예측)**
- **Branch Target Buffer (BTB)**: 분기 이력 저장
- **예측 요소 3가지**:
  1. 분기 명령어인지 여부
  2. 분기 방향 (taken/not-taken)
  3. 분기 목표 주소

#### **🎯 Branch Prediction 방식**

**1-bit Prediction**:
- 마지막 실행 결과 저장
- **문제**: 루프 첫/마지막에서 오예측

**2-bit Prediction** (중요!):
```
Strongly    Weakly     Weakly      Strongly
Taken  →   Taken  →  Not-taken →  Not-taken
 (11)       (10)        (01)        (00)
```
- **장점**: 더 안정적인 예측

---

## 📊 **5. 성능 분석** ⭐⭐⭐

### **핵심 공식들** (암기 필수!)

#### **CPI 계산**
```
CPI_pipelined = 1 + Pipeline stall cycles per instruction
```

#### **Speedup 계산**
```
Speedup = CPI_unpipelined / CPI_pipelined
        = Pipeline depth / (1 + Stall cycles per instruction)
```

#### **분기 해저드의 CPI 기여도**
```
CPI 증가 = Branch frequency × Branch penalty
```
- **예시**: 분기 빈도 20%, 패널티 3 사이클
- **결과**: 3 × 0.20 = 0.6 (CPI가 1.6이 됨)

---

## 🧮 **6. 계산 문제 유형** ⭐⭐

### **문제 유형 1: CPI 계산**
- 주어진 stall cycle로 전체 CPI 계산
- 각 해저드별 기여도 분석

### **문제 유형 2: Speedup 계산**
- 파이프라인 도입 전후 성능 비교
- 이상적 케이스 vs 실제 케이스

### **문제 유형 3: 해저드 분석**
- 주어진 어셈블리 코드에서 해저드 찾기
- Forwarding 적용 후 stall 계산

---

## 🎯 **7. 시험 출제 포인트** ⭐⭐⭐

### **객관식 단골 문제**
- 파이프라인 5단계 순서
- 해저드 유형 분류
- Forwarding 가능/불가능 판단
- Branch prediction 방식 차이

### **서술형 예상 문제**
- 파이프라인 다이어그램 그리기
- 해저드 해결 방법 설명
- 성능 향상 방안 제시
- Load-Use 해저드 분석

### **계산 문제 패턴**
- CPI와 Speedup 공식 적용
- 분기 예측 miss rate 계산
- 전체 실행 시간 분석

---

## 💡 **8. 암기 tip & 혼동 주의사항**

### **🔥 반드시 암기**
- **IF-ID-EX-MEM-WB** 순서
- **3가지 해저드**: Structural, Data, Control
- **Load-Use는 1 stall 필수**
- **2-bit predictor 상태 전이**

### **⚠️ 헷갈리지 말자**
- Multi-cycle ≠ Pipeline (완전히 다른 개념!)
- Forwarding ≠ 모든 해저드 해결 (Load-Use는 예외)
- Static prediction ≠ Dynamic prediction
- Branch penalty는 파이프라인 깊이에 따라 달라짐

### **📝 서술 시 주의사항**
- "시간을 되돌릴 수 없다" (Load-Use 설명 시)
- "하드웨어 자원 추가 필요" (Forwarding 설명 시)
- "통계적 성능 향상" (Branch prediction 설명 시)

---

## 🚀 **마지막 점검 체크리스트**

✅ 파이프라인 5단계 완벽 암기  
✅ 3가지 해저드 구분 가능  
✅ Forwarding vs Stall 차이점 이해  
✅ Load-Use 해저드 특별한 경우 숙지  
✅ CPI, Speedup 공식 암기  
✅ Branch prediction 방식 구분  
✅ 성능 계산 문제 연습 완료  

**7일 남았다! 하루에 1-2개 섹션씩 복습하면서 완벽하게 마스터하자! 💪**