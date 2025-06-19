---
title: 4_1_CA_Lecture_Data Path
author: khw
date: 2025-06-16
categories: [Exam, ComputerArchitecture]
tags: [2025_first_semester_final_exam]
pin: true
math: true
mermaid: true
---

# 🚀 컴퓨터구조 Data Path 벼락치기 완벽 정리

## 📋 핵심 개념 요약 (30초 암기용)
- **데이터패스**: CPU 내에서 데이터가 흐르는 경로
- **단일 사이클**: 모든 명령어를 1클럭에 실행
- **멀티플렉서**: 여러 입력 중 하나를 선택하는 장치
- **컨트롤 신호**: 데이터패스의 동작을 제어하는 신호들

---

## 🎯 시험 출제 포인트 1: 명령어 포맷 (암기 필수!)

### RISC-V 명령어 포맷
```
비트 위치:  31:25 | 24:20 | 19:15 | 14:12 | 11:7 | 6:0
```

**R-type (레지스터-레지스터 연산)**
```
funct7 | rs2 | rs1 | funct3 | rd | opcode
  7비트   5비트  5비트   3비트   5비트  7비트
```

**I-type (즉값 연산, Load)**
```
immediate[11:0] | rs1 | funct3 | rd | opcode
     12비트       5비트   3비트   5비트  7비트
```

**S-type (Store)**
```
immed[11:5] | rs2 | rs1 | funct3 | immed[4:0] | opcode
   7비트      5비트  5비트   3비트     5비트      7비트
```

**SB-type (Branch)**
```
immed[12,10:5] | rs2 | rs1 | funct3 | immed[4:1,11] | opcode
     7비트       5비트  5비트   3비트      5비트        7비트
```

### 🔥 암기 팁
- **rs1**: 첫 번째 소스 레지스터 (19:15)
- **rs2**: 두 번째 소스 레지스터 (24:20) 
- **rd**: 목적지 레지스터 (11:7)
- **opcode**: 명령어 종류 (6:0)

---

## 🎯 시험 출제 포인트 2: 컨트롤 신호 (표로 암기!)

| 명령어 타입 | RegWrite | ALUSrc | MemRead | MemWrite | MemtoReg | Branch | ALUOp |
|------------|----------|--------|---------|----------|----------|--------|-------|
| **R-type** | 1        | 0      | 0       | 0        | 0        | 0      | 10    |
| **Load**   | 1        | 1      | 1       | 0        | 1        | 0      | 00    |
| **Store**  | 0        | 1      | 0       | 1        | X        | 0      | 00    |
| **Branch** | 0        | 0      | 0       | 0        | X        | 1      | 01    |

### 컨트롤 신호 의미 (암기 필수!)
- **RegWrite**: 레지스터에 쓰기 허용 (1=허용, 0=금지)
- **ALUSrc**: ALU 두 번째 입력 선택 (0=레지스터, 1=즉값)
- **MemRead**: 메모리 읽기 허용 (1=허용, 0=금지)
- **MemWrite**: 메모리 쓰기 허용 (1=허용, 0=금지)
- **MemtoReg**: 레지스터에 쓸 데이터 선택 (0=ALU결과, 1=메모리데이터)
- **Branch**: 분기 허용 (1=분기, 0=순차실행)
- **ALUOp**: ALU 연산 제어 (00=ADD, 01=SUB, 10=funct에따라)

---

## 🎯 시험 출제 포인트 3: ALU 제어

### ALU 제어 코드

| ALU 제어 | 연산 | 사용처 |
|----------|------|--------|
| 0000 | AND | R-type AND |
| 0001 | OR | R-type OR |
| 0010 | ADD | Load/Store, R-type ADD |
| 0110 | SUB | Branch, R-type SUB |

### ALUOp 해석

| ALUOp | 의미 | funct3/funct7 사용 |
|-------|------|-------------------|
| 00 | ADD (Load/Store) | 사용안함 |
| 01 | SUB (Branch) | 사용안함 |
| 10 | R-type | funct3/funct7로 결정 |

---

## 🎯 시험 출제 포인트 4: 데이터패스 구성요소

### 주요 구성요소 (그림에서 식별 필수!)
1. **PC (Program Counter)**: 현재 명령어 주소
2. **Instruction Memory**: 명령어 저장소
3. **Register File**: 32개 레지스터 (읽기 2개, 쓰기 1개)
4. **ALU**: 산술논리연산장치
5. **Data Memory**: 데이터 저장소
6. **Immediate Generator**: 즉값 생성기
7. **Multiplexer**: 데이터 선택기

### 멀티플렉서 위치 (빈칸 문제 단골!)
- **ALU 입력 MUX**: ALUSrc로 제어 (레지스터 vs 즉값)
- **레지스터 쓰기 MUX**: MemtoReg로 제어 (ALU결과 vs 메모리데이터)
- **PC 입력 MUX**: Branch로 제어 (PC+4 vs 분기주소)

---

## 🎯 시험 출제 포인트 5: 명령어별 데이터 흐름

### R-type 명령어 (add rd, rs1, rs2)
1. PC → Instruction Memory
2. rs1, rs2 → Register File 읽기
3. 레지스터 값들 → ALU
4. ALU 결과 → rd 레지스터
5. PC = PC + 4

### Load 명령어 (ld rd, offset(rs1))
1. PC → Instruction Memory
2. rs1 → Register File 읽기
3. rs1 + offset → ALU (주소 계산)
4. ALU 결과 → Data Memory 주소
5. 메모리 데이터 → rd 레지스터
6. PC = PC + 4

### Store 명령어 (sd rs2, offset(rs1))
1. PC → Instruction Memory
2. rs1, rs2 → Register File 읽기
3. rs1 + offset → ALU (주소 계산)
4. rs2 데이터 → Data Memory (ALU 결과 주소에)
5. PC = PC + 4

### Branch 명령어 (beq rs1, rs2, offset)
1. PC → Instruction Memory
2. rs1, rs2 → Register File 읽기
3. rs1 - rs2 → ALU (비교)
4. PC + (offset << 1) → 분기 주소 계산
5. Zero 신호에 따라 PC 갱신

---

## 🎯 시험 출제 포인트 6: 단일 사이클의 특징

### 장점
- 구현이 간단
- 명령어당 정확히 1클럭

### 단점
- 가장 느린 명령어에 맞춰 클럭 설정
- 하드웨어 자원 중복 (명령어/데이터 메모리 분리 필요)
- 성능 비효율

### 중요한 제약사항
- **하바드 아키텍처 필수**: 명령어 메모리와 데이터 메모리 분리
- **한 클럭에 한 자원만 사용**: 레지스터 파일, ALU, 메모리 등

---

## 🎯 시험 출제 포인트 7: Sign Extension (부호 확장)

### Load/Store에서 즉값 처리
- **12비트 즉값 → 64비트 확장**
- 최상위 비트(부호비트)를 복사해서 확장
- 예: 0x800 (12비트) → 0xFFFFFFFFFFFFF800 (64비트)

---

## 🔥 시험 직전 체크리스트

### 그림 문제 대비
□ 각 구성요소 이름과 위치 암기  
□ 컨트롤 신호 연결선 파악  
□ 멀티플렉서 입력/출력 확인  
□ 데이터 흐름 경로 추적  

### 빈칸 문제 대비
□ 컨트롤 신호 표 완전 암기  
□ ALU 제어 코드 암기  
□ 명령어 포맷 비트 위치 암기  

### 서술 문제 대비
□ 각 명령어타입별 실행 과정 설명 가능  
□ 단일 사이클의 장단점 설명 가능  
□ 컨트롤 신호의 역할 설명 가능  

---

## 💡 벼락치기 팁

1. **컨트롤 신호 표를 손으로 10번 써보기**
2. **명령어 포맷을 그림으로 그려가며 암기**
3. **각 멀티플렉서의 선택 조건 반복 암송**
4. **Load 명령어의 데이터 흐름을 소리내어 설명**
5. **R-type과 I-type의 차이점 명확히 구분**

**🚨 중요**: 이 내용들은 모두 시험에서 **빈칸 채우기**나 **그림 해석** 문제로 나올 가능성이 매우 높습니다!