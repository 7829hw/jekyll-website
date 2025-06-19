---
title: 3_2_CA_Lecture_Floating Point Numbe.md
author: khw
date: 2025-06-16
categories: [Exam, ComputerArchitecture]
tags: [2025_first_semester_final_exam]
pin: true
math: true
mermaid: true
---

# 🔥 컴퓨터구조 Floating Point Number 벼락치기 핵심 정리

## 📍 핵심 암기 공식 (무조건 외우기!)

### **IEEE 754 Single Precision 공식**
```
X = (-1)^S × (1 + Fraction) × 2^(Exponent-127)
```
- **S**: Sign bit (1bit)
- **Exponent**: 8bits (Bias = 127)
- **Fraction**: 23bits

### **IEEE 754 Double Precision 공식**
```
X = (-1)^S × (1 + Fraction) × 2^(Exponent-1023)
```
- **S**: Sign bit (1bit)  
- **Exponent**: 11bits (Bias = 1023)
- **Fraction**: 52bits

---

## 🎯 1. Floating Point 기본 개념

### **Scientific Notation (과학적 표기법)**
- **정규화된 표기**: 소수점 왼쪽에 0이 아닌 숫자 하나만
- **예시**: 
  - 십진수: 3155760000 = 3.15576 × 10⁹
  - 이진수: 1.101₂ × 2⁻¹

### **Floating Point vs Fixed Point**
- **Fixed Point**: 소수점 위치가 고정
- **Floating Point**: 소수점 위치가 "떠다님" → 넓은 범위 표현 가능

---

## 🏗️ 2. IEEE 754 표준 구조

### **Single Precision (32bit)**
```
|S|    Exponent    |           Fraction           |
|1|    8 bits      |          23 bits             |
31  30          23  22                           0
```

### **Double Precision (64bit)**
```
|S|     Exponent     |              Fraction              |
|1|     11 bits      |             52 bits                |
63  62            52  51                                  0
```

### **왜 이런 구조?**
- **Sign bit가 맨 앞**: 비교 연산 시 편리
- **Exponent가 Fraction 앞**: 크기 비교 시 효율적
- **Implicit leading 1**: 23bit로 24bit 정밀도 확보

---

## ⚖️ 3. Bias 개념 (★★★ 시험 필수)

### **Bias 사용 이유**
- 지수가 음수일 수 있어서 부호 있는 정수 표현 필요
- 하지만 비교 연산을 위해 **양수로만** 표현하고 싶음
- **해결책**: Bias 값을 더해서 모두 양수로 만들기

### **Bias 값**
- **Single Precision**: Bias = 127 (2⁷-1)
- **Double Precision**: Bias = 1023 (2¹⁰-1)

### **실제 지수 계산**
```
실제 지수 = 저장된 지수 - Bias
```

### **예시**
```
Single Precision에서:
- 저장된 지수 = 126 (01111110₂) → 실제 지수 = 126-127 = -1
- 저장된 지수 = 128 (10000000₂) → 실제 지수 = 128-127 = +1
```

---

## 🔢 4. 특수한 값들 (★★★ 기말고사 출제)

### **Single Precision 기준**

| Exponent | Fraction | 의미 |
|----------|----------|------|
| 00000000 | 00000000 | **0** |
| 00000000 | non-zero | **Denormalized Number** |
| 00000001~11111110 | anything | **Normal FP Number** |
| 11111111 | 00000000 | **Infinity (∞)** |
| 11111111 | non-zero | **NaN (Not a Number)** |

### **Double Precision 기준**

| Exponent | Fraction | 의미 |
|----------|----------|------|
| 00000000000 | 000...000 | **0** |
| 00000000000 | non-zero | **Denormalized Number** |
| 00000000001~11111111110 | anything | **Normal FP Number** |
| 11111111111 | 000...000 | **Infinity (∞)** |
| 11111111111 | non-zero | **NaN** |

### **특수 값 발생 경우**
- **Infinity**: 0으로 나누기, overflow 등
- **NaN**: 0/0, √(-1), ∞-∞ 등 무효한 연산
- **Denormalized**: 너무 작은 수 (underflow 근처)

---

## 📊 5. 범위와 정밀도

### **Single Precision (32bit)**
- **최소값**: ±1.2 × 10⁻³⁸
- **최대값**: ±3.4 × 10⁺³⁸
- **정밀도**: 약 7자리 십진수

### **Double Precision (64bit)**
- **최소값**: ±2.2 × 10⁻³⁰⁸
- **최대값**: ±1.8 × 10⁺³⁰⁸
- **정밀도**: 약 15~16자리 십진수

---

## ➕ 6. Floating Point 덧셈 알고리즘

### **덧셈 과정 (5단계)**
1. **Align**: 지수를 맞춤 (작은 지수를 큰 지수에 맞춤)
2. **Add**: significand 덧셈
3. **Normalize**: 결과를 정규화
4. **Round**: 반올림
5. **Check**: overflow/underflow 확인

### **예시: 9.999×10¹ + 1.610×10⁻¹**
```
1단계 (Align): 9.999×10¹ + 0.016×10¹
2단계 (Add):   9.999 + 0.016 = 10.015
3단계 (Normalize): 1.0015×10²
4단계 (Round): 1.002×10² (4자리로 반올림)
```

### **Binary 덧셈 예시: 0.5 + (-0.4375)**
```
0.5 = 1.000×2⁻¹
0.4375 = 1.110×2⁻²

1단계: 1.000×2⁻¹ - 0.111×2⁻¹
2단계: 1.000 - 0.111 = 0.001
3단계: 1.0×2⁻⁴ (정규화)
```

---

## ⚠️ 7. Floating Point의 한계와 오차

### **Round Error (반올림 오차)**
- **0.1을 정확히 표현 불가**: 이진수로 무한소수
- **0.1을 100번 더해도 10이 안됨**
- **실제 사례**: 패트리어트 미사일 시스템 오류 (0.1초×8시간 = 20% 오차)

### **Density 문제**
- **작은 수**: 조밀하게 분포
- **큰 수**: 성기게 분포
- **32bit로 2³² 개의 서로 다른 값만 표현 가능**

### **Overflow & Underflow**
- **Overflow**: 지수가 너무 커서 표현 불가 → Infinity
- **Underflow**: 지수가 너무 작아서 표현 불가 → 0 또는 Denormalized

---

## 🧮 8. 실전 계산 문제 유형

### **유형 1: 십진수 → IEEE 754 변환**
**예시: -0.75를 Single Precision으로**
```
1단계: -0.75 = -3/4 = -11₂/100₂ = -0.11₂
2단계: -0.11₂ × 2⁰ = -1.1₂ × 2⁻¹
3단계: Sign = 1, Exponent = -1+127 = 126, Fraction = 100...000

결과: 1 01111110 10000000000000000000000
```

### **유형 2: IEEE 754 → 십진수 변환**
**공식 적용**: (-1)ˢ × (1 + Fraction) × 2^(Exponent-127)

### **유형 3: 특수값 판별**
- Exponent와 Fraction 값 확인
- 위 표 참조하여 0, ∞, NaN, 정상값 구분

---

## 🎯 9. 시험 출제 포인트 정리

### **객관식 단골 문제**
1. IEEE 754 구조 (bit 수)
2. Bias 값 (127, 1023)
3. 특수값 판별 (0, ∞, NaN)
4. 범위와 정밀도 비교

### **서술형/계산 문제**
1. 십진수 ↔ IEEE 754 변환
2. Floating Point 덧셈 과정
3. 오차 발생 원인 설명
4. Overflow/Underflow 상황

### **암기 필수 숫자**
- **Single Precision**: 1+8+23 bits, Bias=127
- **Double Precision**: 1+11+52 bits, Bias=1023
- **Single 범위**: ±1.2×10⁻³⁸ ~ ±3.4×10⁺³⁸
- **Double 범위**: ±2.2×10⁻³⁰⁸ ~ ±1.8×10⁺³⁰⁸

---

## 📝 최종 점검 체크리스트

- [ ] IEEE 754 공식 암기 완료
- [ ] Single/Double Precision 구조 이해
- [ ] Bias 개념과 계산법 숙지
- [ ] 특수값 표 완전 암기
- [ ] Floating Point 덧셈 5단계 암기
- [ ] 오차 발생 원인 3가지 이상 설명 가능
- [ ] 실전 변환 문제 3개 이상 연습 완료

**🔥 벼락치기 성공 비법: 위 체크리스트를 시험 전날 3번 이상 반복!**