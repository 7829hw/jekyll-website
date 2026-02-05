---
title: 병렬 MCU 처리를 통한 PIM 아키텍처 기반의 향상된 JPEG 디코딩
author: khw
date: 2026-02-06
categories: [Study, PIM]
tags: [LAB]
pin: true
math: true
mermaid: true
---

## 병렬 MCU 처리를 통한 PIM 아키텍처 기반의 향상된 JPEG 디코딩

김지은, 경북대학교 컴퓨터학부, 대구, 대한민국, kimjea1999@knu.ac.kr

남덕윤, 경북대학교 컴퓨터학부, 대구, 대한민국, dynam@knu.ac.kr

초록 - JPEG 형식은 시각적 품질을 유지하면서 파일 크기를 줄여 디지털 이미지를 압축할 수 있습니다. 그러나 대규모 이미지 처리가 필수적인 경우 JPEG 디코딩이 병목 현상이 될 수 있습니다. JPEG 디코딩을 가속화하기 위한 수많은 연구가 진행되었지만, 연산 유닛을 메모리에 통합하여 데이터 병목 현상을 완화하는 메모리 내 처리(PIM, Processing-in-Memory) 아키텍처를 탐구한 연구는 단 하나뿐입니다. UPMEM PIM 솔루션은 DRAM 처리 장치(DPU)를 통해 대규모 병렬 처리를 가능하게 합니다. 기존의 PIM 기반 JPEG 디코더는 각 DPU가 최소 하나의 이미지를 처리해야 하는 UPMEM PIM의 구조적 제약으로 인해 DPU 수준의 병렬성을 완전히 활용하지 못합니다. 따라서 지원되는 이미지 해상도가 제한적이며 디코딩 속도가 감소합니다. 또한 하드웨어 스레드 동기화 구현의 어려움으로 인해 이미지 블록 부정합(misalignment)이 발생하여 원본 이미지와 불일치가 생길 수 있습니다. 본 논문에서는 다중 DPU의 병렬성을 활용하여 호스트 CPU에서 허프만 디코딩을 수행하고, 이미지 블록 간의 의존성을 제거하며, DPU 전반에 걸친 분산 처리를 가능하게 하는 새로운 PIM 디코더(NPD, Novel PIM Decoder)를 제안합니다. 이 접근 방식은 지원 가능한 이미지 해상도를 높이고, 디코딩 속도를 개선하며, 동기화 없이 여러 블록을 단일 이미지로 재조립하여 디코딩 품질을 향상시킵니다. 저해상도 및 고해상도 이미지 데이터셋을 사용한 UPMEM PIM 서버에서의 성능 평가는 디코더들의 디코딩 가능한 이미지 해상도, 실행 시간 및 디코딩 품질을 비교합니다. NPD는 기존 디코더보다 최대 3.57배 더 높은 해상도의 이미지를 처리하며, 실행 시간을 4%에서 85%까지 단축합니다. 최대 신호 대 잡음비(PSNR)와 구조적 유사성 지수(SSIM)를 사용하여 디코딩된 결과와 원본 이미지를 비교한 결과, NPD가 더 높고 안정적인 디코딩 품질을 제공함을 확인했습니다.

색인 용어 - JPEG 디코딩, 메모리 내 처리(PIM), 병렬 컴퓨팅, DRAM 처리 장치(DPU)

## I. 서론

JPEG(Joint Photographic Expert Group) 표준은 1992년 ISO/IEC 10918-1에서 디지털 이미지의 손실 압축을 위해 개발된 이래 웹 그래픽, 디지털 카메라, 이미지 저장 및 전송 등 다양한 분야에서 널리 채택되었습니다 [1]. JPEG의 주파수 변환 및 양자화 기술은 시각적 이미지 품질을 유지하면서 파일 크기를 줄여주어 디지털 이미지 처리의 필수 기술이 되었습니다. 또한, 압축된 이미지를 픽셀 수준 데이터로 복원하여 장치에 표시하거나 후처리를 하는 JPEG 디코딩은 실시간 애플리케이션이나 수많은 이미지를 효율적으로 처리해야 하는 딥러닝과 같은 분야에서 중요한 역할을 합니다. JPEG 비트스트림은 효율적인 이미지 데이터 저장을 위한 가변 길이 코딩 기술인 허프만 코딩(Huffman coding)을 사용하여 압축됩니다.

그러나 허프만 코딩은 차분 부호화(differential coding)를 사용하므로 데이터 의존성이 높고 병렬화를 위한 구조적 한계가 발생합니다. 이러한 특성으로 인해 JPEG 디코딩은 실시간 처리가 필요하거나 대규모 데이터셋을 다루는 환경에서 병목 현상이 될 수 있습니다. 이러한 문제를 해결하기 위해 SIMD 명령어를 사용하여 여러 데이터 스트림을 동시에 처리하거나 그래픽 처리 장치(GPU), 현장 프로그래밍 가능 게이트 어레이(FPGA), 주문형 반도체(ASIC)와 같은 하드웨어 가속기를 적용하는 등 병렬 처리 기술을 탐구한 여러 연구가 있습니다 [2]-[7]. 그럼에도 불구하고 이러한 접근 방식은 하드웨어 아키텍처에 따라 달라질 수 있는 병렬 처리의 복잡성으로 인해 여전히 어려움을 겪고 있습니다.

PIM(Processing-in-Memory) 아키텍처에서의 JPEG 디코딩 관련 연구로, Nider 등 [2]은 유일하게 상용화된 제품인 UPMEM의 PIM 하드웨어 [8]를 사용하여 JPEG 디코딩의 병렬화를 조사했습니다. 그들의 접근 방식은 태스크릿(하드웨어 스레드)과 Arai, Agui, Nakajima (AAN) 알고리즘 [9]을 적용하여 PIM의 연산 한계를 완화하고 성능을 향상시키는 것을 목표로 했습니다. 그러나 UPMEM PIM 서버는 DRAM 처리 장치(DPU) 당 64MB의 제한된 DRAM 용량과 DPU 간의 직접 통신 부재와 같은 구조적 한계가 있습니다. 병렬 허프만 디코딩 기술은 허프만 코드의 자기 동기화(self-synchronization) 속성을 적용하며 병렬 처리 장치 간의 통신을 필요로 합니다. 차분 부호화는 각 최소 코딩 단위(MCU) 간에 적용되어 데이터 의존성을 초래합니다. 따라서 JPEG 디코딩의 모든 단계를 병렬화하려면 이미지당 하나의 DPU만 할당해야 합니다. 그러나 이는 처리할 수 있는 이미지 해상도를 제약하며, 병렬 처리에도 불구하고 DPU의 제한된 연산 능력으로 인해 libjpeg-turbo [10]보다 디코딩 시간이 더 오래 걸립니다.

본 논문에서는 이전 연구의 한계를 극복하기 위해 설계된 UPMEM PIM을 사용하는 효율적인 병렬 JPEG 디코더인 새로운 PIM 디코더(NPD)를 제안합니다. NPD는 호스트 CPU에서 허프만 디코딩을 수행하는 한편, 다중 DPU를 사용하여 MCU를 병렬로 처리함으로써 JPEG 디코딩 중 병렬화 문제를 해결합니다. 이러한 디코딩 파이프라인 수정은 기존 PIM 기반 디코더의 기능적 한계를 극복하고 다중 DPU를 유연하게 사용하여 실행 시간을 단축합니다.

성능 평가는 두 가지 데이터셋 유형을 사용하여 UPMEM PIM 서버에서 수행되었으며, 기존 PIM 기반 JPEG 디코더와 비교하여 최대 디코딩 가능 해상도, 디코딩 시간, 디코딩 품질의 세 가지 측면을 비교했습니다. 실험 결과에 따르면 NPD는 세 가지 측면 모두에서 기존 디코더보다 우수한 성능을 보였습니다. 3.57배 더 높은 해상도의 이미지를 처리하고, 모든 경우에서 더 빠른 실행 시간을 달성하며, 일관되게 더 높고 안정적인 디코딩 품질을 제공합니다.

본 논문의 기여는 다음과 같습니다:

- 본 연구는 PIM에서의 허프만 디코딩 병렬화가 메모리를 제약하고 성능을 저하시키는 이유를 설명하고, JPEG 디코딩의 부분적 병렬화를 통해 이를 해결합니다.
- 본 논문은 기존 PIM 기반 JPEG 디코더의 한계를 줄이는 NPD를 제안합니다. NPD는 호스트 CPU에서 순차적 허프만 디코딩을 수행하고 멀티스레드 구조를 사용하여 성능 오버헤드를 완화합니다.
- 블록 기반 데이터 분배를 통해 크로마 서브샘플링(chroma-subsampled)된 이미지를 여러 DPU에 걸쳐 디코딩할 수 있습니다.
- 이미지 수와 해상도에 따른 실행 시간 변화를 분석하여 NPD와 PIM-JPEG를 비교합니다. 또한, 다중 데이터 전송이 데이터 전송 시간을 어떻게 상당히 증가시킬 수 있는지에 대한 통찰력을 제공합니다.

## II. 관련 연구

JPEG 디코딩을 개선하기 위해 가속기를 사용한 병렬 처리 기술에 대한 연구가 수행되었습니다. Sodsong 등은 CPU를 사용하여 비트스트림에서 계수 블록(coefficient blocks)의 시작 위치를 빠르게 스캔하고 GPU에서 각 블록을 병렬로 디코딩하는 방법을 제안했습니다 [3]. Wang 등은 두 가지 사전 스캔(prescan) 방법을 사용하여 임의의 블록 단위로 JPEG를 디코딩하고 GPU에서 병렬로 처리하는 방법을 제시했습니다 [4]. Cheng 등은 효율적인 FPGA 기반 JPEG 디코더를 설계하고 딥러닝 워크로드를 위한 데이터 전처리를 FPGA 디코더로 오프로딩(offloading)하면 데이터 지연(data stalls)을 개선할 수 있음을 입증했습니다 [5]. 이러한 연구들의 공통점은 허프만 데이터의 병렬 디코딩 방법을 탐구했다는 것입니다. Klein과 Wiseman은 허프만 코드의 특성을 적용하여 허프만 인코딩된 비트스트림을 멀티코어 프로세서에서 병렬로 디코딩할 수 있음을 증명했습니다 [11].

허프만 코드의 자기 동기화 특성은 허프만 인코딩된 비트스트림의 시작 위치에 관계없이 디코딩이 결국 올바르게 진행되는 동기화 지점에 도달하도록 보장합니다. 이 연구는 이 속성을 다중 프로세서를 사용하는 JPEG 이미지 디코딩에 적용하여 병렬 허프만 디코딩이 가능함을 입증했습니다.

Nider 등 [2]은 UPMEM PIM(이하 'PIM-JPEG')을 사용하여 대규모 병렬 JPEG 디코딩의 타당성을 평가했습니다. PIM-JPEG 방식은 JPEG 이미지를 DPU로 전송하고 다중 태스크릿을 사용하여 하나의 이미지를 병렬로 디코딩합니다. 이 방법은 허프만 디코딩에 병렬화된 접근 방식을 채택하여 단일 이미지가 여러 DPU에 분산되는 것을 방지합니다.

이 작업은 디코딩 오프로딩 방식의 제약으로 인해 세 가지 중요한 한계를 가집니다. 첫째, 고해상도 이미지를 지원하지 않습니다. 각 DPU에는 대부분의 JPEG 파일보다 큰 64MB의 DRAM이 연결되어 있지만, 디코딩된 MCU는 약 380바이트의 고정 크기를 차지합니다. JPEG 파일 크기가 메인 RAM(MRAM) 용량보다 작더라도 이미지가 174,762개 이상의 MCU를 포함하면 메모리 제약으로 인해 처리할 수 없습니다.

둘째, 디코딩 속도가 저하될 수 있습니다. 이미지당 하나의 DPU만 할당되므로 병렬성이 완전히 활용되지 않아 여러 이미지를 처리할 때 비효율적입니다. 단일 DPU의 연산 능력은 CPU보다 현저히 낮기 때문에 PIM에서 효율적인 JPEG 디코딩을 위해서는 여러 DPU에 워크로드를 분산하여 병렬성을 극대화하는 것이 중요합니다. 그러나 현재 접근 방식은 이 기능을 제한합니다.

셋째, 디코딩 품질이 저하됩니다. 처리 장치 간의 올바른 동기화 구현의 복잡성으로 인해 디코딩 오류 및 출력 손상이 발생할 수 있습니다. Kaggle에 공개된 Clothing 데이터셋 [12]을 사용한 실험에서 PIM-JPEG 디코더는 이미지의 약 20%를 올바르게 디코딩하지 못했습니다.

## III. 배경 지식

## A. JPEG 디코딩 과정

JPEG 디코딩 과정은 압축된 이미지 데이터를 재구성하여 픽셀 수준의 RGB 값으로 변환합니다. 이 과정은 (1) 메타데이터 스캔, (2) 허프만 디코딩, (3) 역양자화(dequantization), (4) 역 이산 코사인 변환(IDCT), (5) 색 공간 변환(colorspace conversion)의 다섯 가지 주요 단계로 구성됩니다. 첫 번째 단계인 메타데이터 스캔에서는 JPEG 마커, 허프만 테이블, 양자화 테이블을 포함한 메타데이터를 파일에서 추출합니다. JPEG 마커는 이미지 시작 및 종료 마커와 같은 파일 구조를 정의합니다. 각 마커는 0xFF로 시작하며 JPEG 파일 내 데이터 블록의 의미와 위치를 식별합니다.

두 번째 단계는 허프만 코드를 사용하여 압축된 비트스트림을 MCU로 변환하는 허프만 디코딩입니다. JPEG 인코딩에서 각 MCU는 빈번한 값을 더 짧은 코드로 표현하는 허프만 인코딩을 사용하여 압축됩니다. MCU는 DC 및 AC 계수로 구성됩니다. DC 계수는 블록의 평균 색상 값을 나타내고, AC 계수는 미세한 세부 사항(예: 질감 또는 가장자리)을 캡처합니다. 이러한 계수는 별도의 허프만 테이블을 사용하여 인코딩됩니다. DC 계수는 MCU를 형성하는 픽셀의 평균값을 나타내며, 유사한 특성으로 인해 인접한 MCU 간에 약간의 변형이 발생합니다. 압축 효율을 높이기 위해 차분 부호화를 사용하여 DC 계수와 이전 값 간의 차이를 기록합니다. 따라서 DC 계수를 디코딩할 때 이전 MCU의 값을 더해야 합니다.

![Image](/assets/img/posts/2026-02-06-Enhanced_JPEG_Decoding_Using_PIM_Architectures_with_Parallel_MCU_Processing_한국어\figure1.png)
그림 1: DC 계수 디코딩 예시.

![Image](/assets/img/posts/2026-02-06-Enhanced_JPEG_Decoding_Using_PIM_Architectures_with_Parallel_MCU_Processing_한국어\figure2.png)
그림 2: 지그재그 인코딩

그림 1은 재시작 간격(restart interval)을 기반으로 DC 계수를 디코딩하는 방법을 보여줍니다. 첫 번째 MCU의 경우 0 값이 추가됩니다. 재시작 마커는 지정된 수의 MCU 후에 누적된 DC 계수 값을 0으로 자주 재설정하여 오류 수정을 용이하게 합니다. 재시작 간격이 없는 경우(그림 1(a)), DC 계수는 이전 MCU의 DC 계수에 지속적으로 의존하는 반면, 재시작 간격이 4인 경우(그림 1(b)), 다섯 번째 MCU는 이전 MCU의 DC 계수에 의존하지 않습니다.

AC 계수는 각 픽셀 값과 MCU 내 평균값의 차이이며 MCU의 상세한 변화를 나타냅니다. 일반적으로 많은 0 값이 존재하며, 이러한 희소성(sparseness)을 적용하기 위해 지그재그 및 런 렝스(run-length) 인코딩이 적용됩니다. 지그재그 인코딩은 AC 계수 크기가 MCU의 오른쪽 하단으로 갈수록 감소하는 특성을 사용하여 그림 2의 순서대로 인코딩하여 연속된 0의 길이를 최대화합니다. 런 렝스 인코딩은 동일한 값이 연속적으로 나타나는 구간인 런(run)의 길이를 저장하는 압축 기술입니다. 이 인코딩은 0의 런이 긴 AC 계수를 저장하는 데에도 효율적입니다.

허프만 코드는 가변 길이를 가지므로 순차적으로 디코딩될 때까지 비트스트림에서 각 코드를 식별할 수 없습니다. 따라서 데이터를 블록으로 나누어 각 블록을 병렬로 디코딩하는 방법을 적용하기 어렵습니다. 허프만 디코딩 병렬화 연구는 허프만 코드의 자기 동기화 특성을 활용합니다. 이 특성은 허프만 인코딩된 비트스트림의 시작 위치에 관계없이 디코딩이 올바르게 진행되는 동기화 지점에 도달하도록 보장합니다. 동기화 지점은 서로 다른 위치에서 디코딩을 시작한 두 처리 장치가 동일한 디코딩 결과를 생성하기 시작하는 곳입니다.

또한 각 MCU의 DC 계수에 차분 부호화가 적용되므로 이전 MCU의 DC 계수에 의존합니다. 병렬 처리 장치(예: UPMEM PIM의 DPU) 간의 직접 통신이 지원되지 않는 경우 동기화 및 DC 계수 조정 모두 병목 현상이 되어 여러 DPU에 걸쳐 이 단계를 병렬화하는 것이 비효율적입니다.

세 번째 단계는 MCU를 역양자화하는 과정입니다. 양자화는 JPEG 압축 과정에서 손실 압축을 수행하고 주파수 성분의 정밀도를 낮추어 픽셀 값을 줄이는 단계입니다. 역양자화는 양자화의 반대 과정으로 양자화 테이블을 사용하여 양자화된 값을 원래 주파수 성분에 가깝게 복원합니다. 양자화 테이블은 MCU와 동일한 8x8 크기를 가지며 역양자화는 다음과 같이 수행됩니다.

$$
D[i][j] = MCU[i][j] \times QT[i][j]
\quad (0 \leq i, j < 8)
$$

일반적인 컬러 JPEG 이미지는 밝기 정보를 나타내는 Y 채널과 색차(chrominance) 정보를 나타내는 Cb 및 Cr 채널로 구성됩니다. 인간의 시각은 밝기 변화에 더 민감하고 색차 변화에 덜 민감합니다.

![Image](/assets/img/posts/2026-02-06-Enhanced_JPEG_Decoding_Using_PIM_Architectures_with_Parallel_MCU_Processing_한국어\figure3.png)
그림 3: 크로마 서브샘플링 예시.

이러한 민감도는 일반적으로 Y 및 CbCr 채널이 서로 다른 양자화 테이블을 사용하도록 적용됩니다. 색차 채널의 양자화 계수를 큰 값으로 설정하면 색차 채널의 압축률이 증가하여 데이터 크기가 줄어들지만, 이로 인한 손실은 육안으로 식별하기 어렵습니다. 따라서 이미지 용량을 효율적으로 줄일 수 있습니다.

네 번째 단계는 IDCT로, 주파수 영역으로 변환된 주파수 성분을 다시 공간 영역(즉, 픽셀 데이터)으로 복원합니다. JPEG 압축의 중요한 요소 중 하나는 이미지 블록(일반적으로 8x8 픽셀)의 주파수 성분이 분리되는 DCT입니다. 저주파 성분은 전체적인 이미지 모양과 패턴을 나타내고, 고주파 성분은 상세한 변화(예: 가장자리 또는 노이즈)를 나타냅니다. JPEG 압축에서 고주파 성분은 시각적 품질에 미치는 영향이 미미하므로 양자화를 통한 손실이 허용됩니다.

마지막 단계는 색 공간 변환으로, YCbCr 색 공간에서 디코딩된 데이터를 일반적으로 사용되는 RGB 색 공간으로 변환합니다. JPEG 이미지는 저장 공간을 줄이기 위해 YCbCr 색 공간을 사용하여 압축되며, YCbCr에서 RGB로의 변환은 다음 수학적 변환 공식을 사용하여 수행되고, MCU를 구성하는 각 픽셀의 R, G, B 값은 Y, Cb, Cr 채널의 MCU 값을 사용하여 계산할 수 있습니다.

$$
\begin{align*}
R[i][j] &= Y[i][j] + 1.402 \times (Cr[i][j] - 128) \\
G[i][j] &= Y[i][j] - 0.344 \times (Cb[i][j] - 128) - 0.714 \times (Cr[i][j] - 128) \\
B[i][j] &= Y[i][j] + 1.772 \times (Cb[i][j] - 128) \\
        &\quad (0 \leq i, j < 8)
\end{align*}
$$

RGB 변환 시 고려해야 할 사항 중 하나는 이미지가 크로마 서브샘플링되었는지 여부입니다. 크로마 서브샘플링은 크로마 채널 값의 절반 또는 1/4만 저장하여 이미지 크기를 줄입니다. 값 압축 방향에 따라 이미지는 수평 및 수직 서브샘플링으로 나뉘며 두 유형 모두 적용될 수 있습니다. 그림 3은 두 가지 서브샘플링 유형을 보여줍니다. 그림 3(a)는 휘도 채널의 Y 0 및 Y 1 MCU가 색차 채널의 Cb 0 및 Cr 0 MCU를 공유하는 수평 서브샘플링을 보여줍니다. 그림 3(b)는 Y 0, Y 1, Y 2, Y 3가 색차 채널의 Cb 0 및 Cr 0 MCU를 공유하는 두 가지 서브샘플링 유형을 모두 묘사합니다.

## B. UPMEM PIM 아키텍처

이 섹션에서는 그림 4의 UPMEM PIM 아키텍처의 핵심 사항을 요약합니다. UPMEM PIM은 하나의 랭크(rank)로 구성되며, 랭크는 여러 개의 PIM 지원 칩으로 구성되고, 각 칩은 8개의 DPU를 포함합니다. DPU는 순차 실행 방식으로 작동하는 32비트 RISC 프로세서로, 태스크릿(tasklet)이라고 불리는 24개의 하드웨어 스레드를 통해 멀티스레딩을 지원하며 동기화 메커니즘(예: 뮤텍스, 배리어, 세마포어)을 제공합니다. DPU 파이프라인은 14단계로 구성되지만 마지막 3단계만 병렬로 실행될 수 있습니다. 따라서 파이프라인을 완전히 적용하려면 최소 11개의 스레드를 사용해야 합니다 [13].

DPU 내의 전용 메모리: 각 DPU에는 명령어 RAM, 작업 RAM(WRAM), MRAM이 포함되어 있습니다. 명령어 RAM은 DPU가 실행하는 프로그램이 저장되는 메모리 영역이며, WRAM은 스택 및 힙 데이터를 저장하기 위한 64KB의 전용 SRAM으로 모든 태스크릿이 공유합니다. MRAM은 호스트가 전송한 데이터가 저장되는 64MB DRAM 슬라이스입니다. DPU 코어가 MRAM에 액세스하거나 MRAM에서 WRAM으로 데이터를 로드할 때 블로킹 직접 메모리 접근(blocking DMA) 명령어를 사용합니다. 직접 메모리 접근으로 인한 대기 시간은 여러 태스크릿을 사용하여 숨길 수 있습니다.

통신: UPMEM SDK의 통신 메커니즘은 CPU-to-DPU 데이터 전송 및 브로드캐스팅, DPU-to-CPU 데이터 전송을 지원합니다. DPU는 지정된 메모리 영역에만 액세스할 수 있으며 현재 DPU 간 직접 통신은 지원되지 않습니다. DPU 간 데이터 전송은 호스트 CPU를 거쳐야 하므로 상당한 오버헤드가 발생하며 데이터 의존도가 높은 워크로드는 PIM에 부적합합니다. 예를 들어, 병렬 허프만 디코딩에서 블록에 할당된 스레드나 프로세서는 다음 블록의 비트를 처리해야 할 수 있습니다. 여러 DPU를 사용할 때 DPU 간 잦은 데이터 이동은 비효율성을 초래할 수 있습니다.

산술 연산의 한계: DPU 코어는 기본적으로 32비트 및 64비트 정수 덧셈과 뺄셈을 지원합니다. 그러나 32비트 및 64비트 정수 곱셈과 나눗셈 및 부동 소수점 연산은 하드웨어 수준에서 지원되지 않으며 소프트웨어에서 에뮬레이션해야 하므로 계산 속도가 크게 감소합니다. UPMEM PIM에 대한 연구는 이러한 한계로 인해 부동 소수점 연산을 정수 기반의 'add and shift' 기술로 대체하는 경우가 많습니다. JPEG 디코딩에서 IDCT 및 색 공간 변환 과정은 많은 부동 소수점 연산을 포함하므로 정수 산술을 사용하여 이러한 연산을 최적화하는 것이 필수적입니다.

![Image](/assets/img/posts/2026-02-06-Enhanced_JPEG_Decoding_Using_PIM_Architectures_with_Parallel_MCU_Processing_한국어\figure4.png)
그림 4: UPMEM PIM 아키텍처.

## IV. 다중 DPU를 이용한 JPEG 디코딩

## A. 아키텍처 설계 및 작업 할당 전략

UPMEM PIM 플랫폼은 DPU당 64MB의 제한된 DRAM 용량과 직접적인 DPU 간 통신 부재를 포함한 구조적 한계를 가지고 있습니다. 이러한 제약으로 인해 기존 디코더 [2]는 이미지당 하나의 DPU만 할당할 수 있어 메모리 용량 제한으로 인해 고해상도 이미지 처리가 제한되었습니다. 우리는 호스트 CPU와 다중 DPU 간의 전략적 작업 분배를 통해 해상도 제한을 완화하여 JPEG 디코딩을 지원하는 NPD를 제안함으로써 이러한 문제를 해결합니다. 제안된 NPD 아키텍처에서는 초기 두 단계(메타데이터 스캔 및 허프만 디코딩)는 호스트 CPU에서 실행되는 반면, 후속 단계(역양자화, IDCT, 색 공간 변환)는 여러 DPU에 걸쳐 병렬화됩니다.

메타데이터 스캔 및 허프만 디코딩을 DPU가 아닌 호스트 CPU에서 수행하기로 한 결정은 이러한 단계의 병렬화와 관련된 문제에 기반합니다. 메타데이터 스캔에서는 각 데이터 요소의 크기와 위치를 미리 결정할 수 없으며 데이터 유형에 따라 형식과 처리 방법이 달라져 SIMD와 같은 병렬화 기술 적용이 복잡합니다. 스캔된 데이터는 모든 프로세서 간에 공유되어야 하므로 빈번한 통신이 필요합니다. 병렬화된 허프만 디코딩 또한 처리 장치 간의 통신이 필요하며, 다음 단계인 DC 계수 조정은 순차적으로 수행되어야 합니다. 그러나 DPU 간의 통신은 CPU를 통해 라우팅되어야 하므로 비효율적입니다. 메타데이터 스캔 및 허프만 디코딩은 전체 실행 시간의 작은 부분을 차지하며 병렬 처리에 구조적 문제를 제기하므로, 본 연구에서는 이러한 단계를 호스트 CPU에서 관리합니다. 비트스트림이 스캔되면 DPU에 의해 병렬로 디코딩됩니다.

JPEG 디코딩의 다음 단계(역양자화, IDCT, 색 공간 변환)는 데이터 병렬 처리에 더 적합하지만 높은 복잡성으로 인해 계산 집약적입니다. NPD는 DPU에서 여러 태스크릿을 사용하여 이러한 단계를 병렬로 실행함으로써 처리 속도를 향상시킵니다. 그러나 DPU 코어는 CPU보다 낮은 클럭 주파수에서 작동하며 JPEG 디코딩에서 부동 소수점 연산을 기본적으로 지원하지 않으므로 성능을 향상시키고 실행 시간을 최소화하기 위해 최적화 기술이 적용되었습니다.

![Image](/assets/img/posts/2026-02-06-Enhanced_JPEG_Decoding_Using_PIM_Architectures_with_Parallel_MCU_Processing_한국어\figure5.png)
그림 5: 데이터 분배 방법 비교.

## B. 크로마 서브샘플링 지원을 위한 블록 기반 MCU 분배

여러 DPU에 데이터를 분배할 때는 데이터 간의 의존성을 고려해야 합니다. 압축 방식에 따라 허프만 디코딩이 완료된 MCU 간에 데이터 의존성이 존재할 수 있습니다. 색 공간 변환 중에는 세 채널의 동일한 위치에 있는 MCU가 참조됩니다. 그러나 크로마 서브샘플링된 이미지에서는 휘도 및 색차 채널의 MCU가 1:1 비율로 매핑되지 않습니다. 예를 들어, 4:2:0 서브샘플링된 이미지에서 휘도 채널의 2x2 MCU 블록은 색차 채널의 단일 MCU를 공유합니다.

NPD는 크로마 서브샘플링을 지원하기 위해 블록 기반 MCU 분배 방식을 채택합니다. 2x2 MCU로 구성된 블록은 일반적으로 사용되는 서브샘플링 방법(예: 수평, 수직 및 4:2:0)에 적용할 수 있는 가장 작은 단위입니다. 그림 5는 두 가지 다른 데이터 분배 방식에서 다양한 DPU로 전송되는 MCU의 예를 보여줍니다. 4:2:0 크로마 서브샘플링 이미지를 가정할 때 각 DPU는 8개의 MCU를 수신합니다. MCU A와 B는 동일한 색차 채널 MCU를 공유하며 동일한 DPU에 할당되어야 합니다. 데이터가 MCU 단위로 분배되는 그림 5(a)에서는 A와 B가 다른 DPU에 할당됩니다. 반면 MCU가 블록 단위로 분배되는 그림 5(b)에서는 동일한 DPU에 할당됩니다. 전자의 경우 DPU 간 통신이 불가능하므로 B에 할당된 DPU는 색 공간 변환을 수행할 수 없습니다. 그러나 후자의 경우 DPU는 서브샘플링된 색차 채널 데이터에 액세스할 수 있습니다.

![Image](/assets/img/posts/2026-02-06-Enhanced_JPEG_Decoding_Using_PIM_Architectures_with_Parallel_MCU_Processing_한국어\figure6.png)
그림 6: 호스트 측 아키텍처.

![Image](/assets/img/posts/2026-02-06-Enhanced_JPEG_Decoding_Using_PIM_Architectures_with_Parallel_MCU_Processing_한국어\algorithm1.png)
알고리즘 1.

## C. 호스트와 메모리 처리 장치 간의 작업 중첩

메타데이터 스캔 및 허프만 디코딩의 높은 데이터 의존성으로 인해 이러한 단계는 DPU가 아닌 CPU에서 실행됩니다. 여러 JPEG 파일을 동시에 처리할 때 CPU와 DPU 간의 작업을 중첩하여 성능을 최적화할 수 있습니다. 시스템은 CPU가 초기 순차 단계를 처리하는 동안 DPU가 전송된 데이터에 대해 다른 단계(역양자화, IDCT, 색 공간 변환)를 동시에 실행하도록 디코딩 파이프라인을 분할하여 작업 중첩을 적용할 수 있습니다.

그림 6은 호스트 CPU의 두 스레드가 JPEG 디코딩의 중첩 단계를 관리하기 위해 조정하는 모습을 보여줍니다. 첫 번째 스레드인 **MCU 준비자(MCU preparer)**는 각 JPEG 이미지 파일을 열고 메타데이터를 파싱하며 허프만 디코딩을 수행합니다. 이러한 작업이 완료되면 MCU 준비자는 파싱된 메타데이터와 디코딩된 MCU를 추가 처리를 위해 큐에 넣습니다. 두 번째 스레드인 **MCU 오프로더(MCU offloader)**는 큐에서 MCU를 검색하여 시스템의 모든 사용 가능한 DPU에 분배합니다. 그런 다음 오프로더는 DPU 실행을 시작하고 디코딩된 결과를 수집하여 데이터를 호스트 CPU로 전송하여 최종 비트맵 파일을 생성합니다.

MCU 준비자는 메타데이터와 MCU 버퍼라는 2차원(2D) 벡터로 구성된 배치를 생성합니다. 이 벡터는 이미지의 메타데이터와 MCU를 각각 저장하여 MCU 오프로더가 관리하는 체계적이고 효율적인 분배 프로세스를 가능하게 합니다.

알고리즘 1은 여러 DPU에 JPEG 이미지를 분배하여 배치를 생성하는 과정을 보여줍니다. 2D 벡터 구조의 배치(batching)는 시스템에서 사용 가능한 DPU 수에 해당하는 행과 각 DPU가 동시에 처리할 수 있는 최대 MCU 수(MAX MCU PER DPU)를 나타내는 열을 가집니다. 이 값은 DPU의 MRAM 용량을 기준으로 결정됩니다. UPMEM PIM 시스템에서 각 DPU는 64MB의 MRAM을 가지며, 각 MCU가 디코딩 후 384바이트를 차지한다고 고려하면 MAX MCU PER DPU의 최대값은 약 174,762입니다. 그러나 성능을 최적화하고 배치 생성 시간을 최소화하기 위해 MAX MCU PER DPU 값을 더 작게 선택하여 더 많은 DPU를 동시에 사용할 수 있게 함으로써 전체 디코딩 시간을 줄이는 경우가 많습니다.

알고리즘은 JPEG 이미지를 열고 처리에 필요한 DPU 수를 계산한 다음 메타데이터를 파싱하여 메타데이터 버퍼를 채우는 것으로 시작합니다. 예를 들어, 이미지 A를 처리하는 데 DPU 0, 1, 2가 필요한 경우 이미지 A의 메타데이터는 메타데이터 버퍼의 0, 1, 2행에 저장됩니다. 알고리즘은 허프만 디코딩을 수행하여 MCU 버퍼에 필요한 데이터를 채웁니다. 이미지를 처리하는 데 필요한 DPU 수가 시스템에서 사용 가능한 DPU 수를 초과하면 부분적으로 생성된 배치가 큐에 추가되고 새 배치가 시작됩니다.

## 알고리즘 2: 다중 태스크릿을 이용한 병렬 MCU 디코딩

- 1 if get tasklet id() = 0 then
- 2 load metadata();
- 3 mcu per tasklet ← mcu num / NR TASKLETS;
- 4 end
- 5 barrier wait();
- 6 start mcu index ← get tasklet id() × mcu per tasklet;
- 7 end mcu index ← start mcu index + mcu per tasklet;
- 8 dequantization(start mcu index, end mcu index);
- 9 IDCT(start mcu index, end mcu index);
- 10 color space conversion(start mcu index, end mcu index);

## D. 다중 태스크릿을 이용한 병렬 실행

NPD의 DPU 측 구성 요소는 각 DPU 내의 다중 태스크릿을 사용하여 PIM의 병렬 처리 기능을 극대화합니다. 알고리즘 2는 DPU에서 실행되는 상세 절차를 개략적으로 설명하며 각 태스크릿의 역할과 워크플로를 보여줍니다. 처음에 각 DPU의 첫 번째 태스크릿은 메타데이터를 MRAM에서 WRAM으로 전송합니다. 메타데이터에는 디코딩 작업에 필요한 필수 정보(즉, 양자화 테이블 및 이미지 구성 요소 수)가 포함되어 있습니다. 이 데이터를 로드함으로써 첫 번째 태스크릿은 모든 태스크릿이 할당된 작업을 수행하기 위한 기초 정보를 제공합니다.

메타데이터가 WRAM에 로드되면 각 태스크릿에 할당된 MCU 수(mcu per tasklet)가 계산됩니다. 이 단계 동안 나머지 태스크릿은 MCU 분배가 완료될 때까지 배리어 동기화를 통해 대기 상태로 들어갑니다. mcu per tasklet은 전역 변수이며 병렬 디코딩 중에 각 태스크릿은 이 값과 태스크릿 번호(get tasklet id())를 기반으로 처리할 MCU의 인덱스 범위를 계산합니다.

MCU 할당 후 각 태스크릿은 할당된 범위에 대해 역양자화, IDCT, 색 공간 변환의 세 가지 주요 디코딩 작업을 진행합니다. 역양자화는 원래 주파수 성분을 재구성하고, IDCT는 데이터를 공간 영역으로 변환하여 픽셀 값을 복원하며, 색 공간 변환은 이 값을 RGB와 같은 필요한 색 공간으로 변환하여 최종 이미지를 생성합니다.

WRAM은 64KB로 제한되어 있으므로 태스크릿은 처리 중인 MCU만 WRAM에 로드하여 이 제약을 관리합니다. 처리된 결과는 MRAM에 저장됩니다. 이 전략은 제한된 WRAM 용량을 효율적으로 사용하고 각 DPU의 병렬성을 극대화하여 디코딩 전반에 걸쳐 최적의 성능과 데이터 일관성을 보장합니다.

## E. 연산 최적화

IDCT 연산은 주파수 영역으로 표현된 데이터를 픽셀 값으로 변환하고, 색 공간 변환은 각 픽셀의 YCbCr 색 공간을 RGB 값으로 변환합니다. 두 단계 모두 부동 소수점 연산을 포함하며 각 픽셀의 세 채널(Y, Cb, Cr)을 모두 처리해야 하므로 많은 부동 소수점 곱셈 연산이 필요합니다. 유일한 현재 상용 제품인 UPMEM PIM은 부동 소수점 연산과 32비트 곱셈을 에뮬레이션하므로 이러한 연산을 DPU에서 빠르게 처리되도록 최적화해야 합니다.

UPMEM PIM 하드웨어에서 부동 소수점 연산을 최적화하기 위해 계산 전에 부동 소수점 값을 정수로 스케일링했습니다. 부동 소수점 값이 상수로 표현되는 경우 적절한 2의 거듭제곱을 곱하고 소수점을 버린 정수 값을 사용하여 대체할 수 있습니다. 이 접근 방식은 부동 소수점 연산을 정수 연산으로 변환하며, 결과는 부동 소수점 상수를 대체한 값에 특정 비트 수만큼 오른쪽 시프트를 수행하여 얻습니다. 예를 들어, 부동 소수점 값을 16을 곱하여 대체한 경우 결과는 값을 4비트 오른쪽 시프트하여 얻습니다. 이 방법은 계산의 부동 소수점 값이 고정되어 있고 결과를 정수로 표현해야 할 때 유용합니다.

또한 NPD는 IDCT의 효율성을 향상시키기 위해 AAN 알고리즘 [9] (연산 최적화에 널리 적용됨)을 적용합니다. AAN 알고리즘은 8x8 DCT의 구조적 대칭성을 활용하고 스케일링 팩터를 양자화 테이블에 포함시켜 곱셈 횟수를 줄입니다. 우리의 구현은 [2]에 기술된 방법론과 일치하는 유사한 최적화 전략을 통합합니다.

## V. 평가

## A. 실험 환경

우리는 UPMEM SDK 2021.3.0을 사용하여 NPD를 구현하고 성능을 평가했습니다. 실험 환경은 40개의 랭크와 Intel Xeon Silver 4215R CPU(3.20GHz)를 갖춘 UPMEM PIM 하드웨어(랭크당 64개 DPU, 총 2,560개 DPU)로 구성됩니다. 기준선(baseline)은 Nider 등 [2]이 제안한 PIM 기반 디코더(PIM-JPEG)입니다. 데이터셋은 이미지 해상도에 따라 저해상도(LR) 및 고해상도(HR) 데이터셋으로 분류됩니다. LR 데이터셋(약 484x424 픽셀)은 ImageNet 2012 검증 세트 [14]의 하위 집합이며, HR 데이터셋(약 1146x1355 픽셀)은 Kaggle에서 제공되는 Clothing 데이터셋 [12]의 하위 집합입니다. 두 데이터셋 모두 PIM-JPEG 실행 오류 없이 BMP 파일로 변환된 원본 데이터셋의 이미지로 구성됩니다. 실행 시간 측정을 위해 clock gettime 함수가 실행의 각 단계에서 소요된 시간을 기록했습니다. 디코딩 품질은 Python의 scikit-image 라이브러리를 사용하여 평가되었습니다.

디코딩 품질을 평가하는 데 사용된 지표는 최대 신호 대 잡음비(PSNR)와 구조적 유사성 지수(SSIM) [15]입니다. PSNR은 원본 이미지와 압축된 이미지 간의 차이를 측정하여 SNR을 dB로 표현합니다. PSNR이 높을수록 압축된 이미지 품질이 좋음을 나타내며, 일반적으로 30dB가 허용 가능한 품질의 임계값으로 간주됩니다 [16]. SSIM은 두 이미지 간의 유사성을 측정하여 밝기, 대비 및 구조 측면에서 유사성을 평가합니다. PSNR은 두 이미지 간의 픽셀 단위 차이를 기반으로 계산되는 반면, SSIM은 이미지 간의 구조적 유사성을 평가하여 인간의 시각적 인식에 더 부합합니다. 두 지표를 계산하려면 비교를 위한 원본 이미지가 필요하므로 이미지 편집 소프트웨어 ImageMagick [17]을 사용하여 데이터셋에서 올바르게 디코딩된 BMP 이미지를 생성했습니다.

![Image](/assets/img/posts/2026-02-06-Enhanced_JPEG_Decoding_Using_PIM_Architectures_with_Parallel_MCU_Processing_한국어\figure7.png)
그림 7: 최대 디코딩 해상도 비교.

## B. 실험 결과 및 분석

1\) 디코딩 가능한 해상도: NPD와 PIM-JPEG 방법을 사용하여 Kaggle에서 제공되는 Clothing 데이터셋 [12]의 모든 이미지를 디코딩하려고 시도했습니다. 그림 7은 두 디코더가 생성한 최고 해상도 BMP 이미지를 비교합니다. PIM-JPEG가 디코딩할 수 있는 최대 해상도는 1796x3070인 반면, NPD가 디코딩한 최고 해상도 이미지는 5120x3840이었습니다. MCU 수로 환산했을 때 NPD는 PIM-JPEG보다 3.57배 더 큰 이미지를 디코딩하여 해상도 제약이 줄어들었음을 입증했습니다.

2\) 실행 시간: LR 및 HR 데이터셋을 사용하여 디코딩된 이미지의 수와 해상도를 기반으로 실행 시간을 비교했습니다. 디코딩 과정을 구현한 코드 중 21.79%가 PIM으로 오프로드되었습니다. 그림 8(a)는 LR 데이터셋의 이미지 수에 따른 두 디코더의 실행 시간을 보여줍니다. 모든 경우에서 NPD의 실행 시간이 PIM-JPEG보다 우수했습니다. 단일 이미지를 디코딩할 때 MCU 준비자와 MCU 오프로더의 실행이 중첩되지 않았음에도 불구하고, 이미지 처리에 여러 DPU를 사용하여 디코딩 시간을 줄였기 때문에 NPD의 실행 시간은 PIM-JPEG의 15%에 불과했습니다. 2,500개의 이미지를 디코딩할 때 NPD의 실행 시간은 22.36초(PIM-JPEG의 63.5%)였으며, PIM-JPEG는 35.23초였습니다.

그림 8(b)와 8(c)는 디코더가 각 단계에서 소비한 시간을 나타냅니다. 큐 대기 시간은 단일 이미지를 디코딩할 때 전체 실행 시간의 62.03%를 차지했지만, 여러 이미지를 처리할 때 MCU 준비자와 MCU 오프로더의 중첩 실행으로 인해 이미지 수가 증가함에 따라 감소했습니다. 데이터 전송 시간은 NPD에서 더 길었으며, 이미지 수에 따른 데이터 전송 시간의 증가폭 또한 PIM-JPEG보다 NPD에서 더 컸습니다. 2,500개의 이미지를 디코딩할 때 데이터 전송은 NPD 실행 시간의 약 14.65%를 차지한 반면, PIM-JPEG는 1.26%에 불과했습니다. DPU 실행에 소요된 시간 비율도 두 디코더 간에 달랐는데, DPU 실행은 PIM-JPEG의 전체 실행 시간 중 44.21%에서 99.58%를 차지했지만 NPD의 경우 5% 미만이었습니다. BMP 쓰기는 두 디코더 모두 호스트 CPU에서 순차적으로 수행되었으며 이미지 수에 비례하여 증가하는 경향을 보였습니다.

그림 9(a)는 HR 데이터셋을 사용한 실행 시간을 보여줍니다. 이전 실험과 마찬가지로 NPD는 모든 경우에서 PIM-JPEG보다 짧은 실행 시간을 보였습니다. 1,000개의 이미지를 디코딩할 때 NPD는 72.6초가 걸린 반면 PIM-JPEG는 75.9초가 걸려 NPD의 실행 시간은 PIM-JPEG의 95.6%였습니다.

그림 9(b)와 9(c)는 HR 데이터셋 실험의 실행 시간을 요약합니다. 각 단계에 소요된 시간 비율은 작은 데이터셋 실험 결과와 유사하지만 BMP 쓰기에 소요된 시간 비율이 증가한 반면 NPD의 큐 대기 시간 비율은 감소했습니다. 이는 MCU 오프로더의 실행 시간이 증가하여 큐 대기 시간과 중첩되었기 때문입니다. NPD의 데이터 전송 시간은 LR 데이터셋을 사용한 실험에 비해 최대 4.6배 증가했습니다.

이전 연구 [18]에서는 UPMEM PIM을 통합한 시스템에서 메모리 간 데이터 이동으로 인해 추가 지연이 발생할 수 있으며, 이는 NPD의 데이터 전송 오버헤드 증가에 기여한다고 언급했습니다. NPD는 PIM-JPEG보다 DPU 실행 시간이 빠르다는 장점이 있습니다. 그러나 DPU는 병렬 처리 장치이므로 이미지 수나 해상도에 따라 실행 시간이 크게 증가하지 않습니다. 이 두 단계의 차이는 여러 DPU를 사용할 때 NPD의 상대적인 성능 저하에 기여합니다. 따라서 수많은 이미지를 디코딩할 때 NPD의 종단 간(end-to-end) 실행 시간이 더 증가할 수 있습니다.

![Image](/assets/img/posts/2026-02-06-Enhanced_JPEG_Decoding_Using_PIM_Architectures_with_Parallel_MCU_Processing_한국어\figure8.png)
그림 8: LR 데이터셋에 대한 실행 시간.

![Image](/assets/img/posts/2026-02-06-Enhanced_JPEG_Decoding_Using_PIM_Architectures_with_Parallel_MCU_Processing_한국어\figure9.png)
그림 9: HR 데이터셋에 대한 실행 시간.

![Image](/assets/img/posts/2026-02-06-Enhanced_JPEG_Decoding_Using_PIM_Architectures_with_Parallel_MCU_Processing_한국어\figure10.png)
그림 10: 기존 디코더와의 비교.

3) 기존 디코더와의 비교: 그림 10은 CPU를 사용하는 이미지 처리 프로그램 ImageMagick과 GPU 기반 JPEG 디코딩 라이브러리 nvJPEG [19]의 실행 시간과 NPD를 비교합니다. ImageMagick 프로그램은 NPD와 동일한 환경에서 평가되었으며, nvJPEG는 Nvidia RTX A4000 GPU(하드웨어 디코더 지원 없음)와 Intel Xeon Gold 6326 CPU가 장착된 워크스테이션에서 실행되었습니다.

실험 결과에 따르면 NPD는 nvJPEG보다 빠른 디코딩 속도를 달성한 반면, ImageMagick과의 속도 차이는 데이터셋 유형에 따라 다릅니다. 이미지당 계산 요구 사항이 상대적으로 낮은 이미지가 많은 경우(LR), NPD와 nvJPEG 모두 CPU보다 빠른 속도를 보여주었습니다. 반대로 이미지당 계산 요구 사항이 높은 이미지가 적은 경우(HR), CPU 디코딩 속도가 더 빠릅니다. 요약하면 수많은 이미지를 처리할 때 PIM이나 GPU와 같은 가속기를 사용한 병렬 처리는 디코딩 속도를 향상시킬 수 있으며, 이러한 경우 NPD가 nvJPEG보다 더 나은 성능을 발휘합니다.

4) 디코딩 품질: NPD, PIM-JPEG, ImageMagick을 사용하여 데이터셋의 모든 이미지를 디코딩한 후 ImageMagick의 결과를 기준으로 NPD와 PIM-JPEG의 디코딩 품질을 계산했습니다. 그림 11은 PSNR과 SSIM 측면에서 LR 데이터셋의 디코딩 품질을 보여줍니다. 두 디코더의 평균 PSNR 및 SSIM 값은 NPD가 PSNR 39.53dB, SSIM 0.98을 달성한 반면 PIM-JPEG는 PSNR 37.77dB, SSIM 0.96을 달성했음을 보여주었습니다. 그림 12는 HR 데이터셋에 대한 디코딩 품질을 보여주며, 여기서 NPD는 PSNR 43.69dB, SSIM 0.99를 달성한 반면 PIM-JPEG는 PSNR 38.57dB, SSIM 0.96을 달성했습니다. 이러한 결과는 두 디코더의 디코딩 품질이 ImageMagick과 유사하지만 NPD가 모든 경우에서 PIM-JPEG보다 일관되게 우수한 성능을 보였음을 나타냅니다. NPD의 이미지 품질 편차가 적어 더 안정적인 디코딩 품질을 제공함을 시사합니다.

## VI. 결론

본 논문에서는 UPMEM PIM의 제약 내에서 JPEG 디코딩을 효율적으로 병렬화하여 제한된 MRAM 용량 및 DPU 간 통신 부재와 같은 다양한 한계를 극복하는 최적화된 JPEG 디코딩 시스템인 NPD를 제안했습니다. 실험 결과, NPD는 병렬 처리를 위해 여러 DPU를 사용하여 기존 PIM 기반 디코더에 비해 디코딩 가능한 이미지 해상도를 최대 3.57배 증가시켰으며 실행 시간을 4%에서 85%까지 단축했습니다. 또한 NPD는 기존 PIM-JPEG 디코더에 비해 디코딩 품질(PSNR 및 SSIM)에서 일관된 이점을 보여주었으며 다양한 이미지 해상도와 데이터셋에서 안정적인 출력을 유지했습니다. 그러나 대규모 데이터셋을 처리할 때 증가된 데이터 전송 오버헤드는 과제였습니다. 향후 연구에서는 데이터 전송 메커니즘을 최적화하고 시스템 아키텍처를 개선하여 고해상도 대규모 이미지 디코딩에 PIM 기능을 적용할 것입니다.

![Image](/assets/img/posts/2026-02-06-Enhanced_JPEG_Decoding_Using_PIM_Architectures_with_Parallel_MCU_Processing_한국어\figure11.png)
그림 11: LR 데이터셋의 디코딩 품질.

![Image](/assets/img/posts/2026-02-06-Enhanced_JPEG_Decoding_Using_PIM_Architectures_with_Parallel_MCU_Processing_한국어\figure12.png)
그림 12: HR 데이터셋의 디코딩 품질.

## 감사의 글

이 연구는 부분적으로 한국 정부(과학기술정보통신부)가 자금을 지원한 한국연구재단(NRF) 보조금(RS-2023-00211606)과 정보통신기획평가원(IITP) 보조금(RS-2022-00156389)의 지원을 받았습니다. 건설적인 의견을 주신 익명의 검토자분들께 감사드립니다. 이 연구는 한국과학기술정보연구원(KISTI)의 자원을 사용했습니다. 교신 저자는 남덕윤입니다.

## 참고문헌

- [1] G. K. Wallace, 'The JPEG still picture compression standard,' IEEE Transactions on Consumer Electronics , vol. 38, no. 1, pages xviii-xxxiv, 1992.
- [2] J. Nider, J. Dagger, N. Gharavi, D. Ng, and A. Fedorova, 'Bulk JPEG decoding on in-memory processors,' in Proceedings of the 15th ACM International Conference on Systems and Storage (SYSTOR) , pp. 51- 57, June 2022. doi: 10.1145/3534056.3534946.
- [3] W. Sodsong, M. Jung, J. Park, and B. Burgstaller, 'JParEnt: Parallel Entropy Decoding for JPEG Decompression on Heterogeneous Multicore Architectures,' in Proceedings of the 7th International Workshop on Programming Models and Applications for Multicores and Manycores (PMAM) , pp. 104-113, March 2016. doi: 10.1145/2883404.2883423.
- [4] L. Wang, Q. Luo, and S. Yan, 'Accelerating Deep Learning Tasks with Optimized GPU-assisted Image Decoding,' in Proceedings of the IEEE 26th International Conference on Parallel and Distributed Systems (ICPADS) , pp. 274-281, December 2020. doi: 10.1109/ICPADS51040. 2020.00045.
- [5] Y. Cheng et al., 'Dlbooster: Boosting end-to-end deep learning workflows with offloading data preprocessing pipelines,' In Proceedings of the 48th International Conference on Parallel Processing (ICPP) , pp. 1-11, August 2019. doi: 10.1145/3337821.3337892.
- [6] K. Yan, J. Shan, and E. Yang, 'CUDA-based acceleration of the JPEG decoder,' in Proceedings of the Ninth International Conference on Natural Computation (ICNC) , pp. 1319-1323, July 2013. doi: 10.1109/ ICNC.2013.6818183.
- [7] A. Weißenberger and B. Schmidt, 'Accelerating JPEG Decompression on GPUs,' in Proceedings of the IEEE 28th International Conference on High Performance Computing, Data, and Analytics (HiPC) , pp. 121130, December 2021. doi: 10.1109/HiPC53243.2021.00026.
- [8] UPMEM SDK, 2024. [Online]. Available: https://sdk.upmem.com/
- [9] Y. Arai, T. Agui, and M. Nakajima, 'A fast DCT-SQ scheme for images,' IEICE TRANSACTIONS (1976-1990) , vol. E71, no. 11, pages 10951097, 1988.
- [10] libjpeg-turbo. [Online]. Available: https://libjpeg-turbo.org/
- [11] S.T. Klein and Y. Wiseman, 'Parallel Huffman Decoding with Applications to JPEG Files,' The Computer Journal , vol. 46, no. 5, pages 487-497, January 2003, doi: 10.1093/comjnl/46.5.48.
- [12] Kaggle. Clothing dataset. [Online]. Available: https://www.kaggle.com/ datasets/agrigorev/clothing-dataset-full
- [13] J. G´ omez-Luna, I. El Hajj, I. Fernandez, C. Giannoula, G. F. Oliveira, O. Mutlu, 'Benchmarking a new paradigm: Experimental analysis and characterization of a real processing-in-memory system,' IEEE Access , vol. 10, pp. 52565-52608, 2022, doi:10.1109/ACCESS.2022.3174101.
- [14] ImageNet. ILSVRC 2012. [Online]. Available: https://www.image-net. org/challenges/LSVRC/2012
- [15] A. Hor´ e and D. Ziou, 'Image Quality Metrics: PSNR vs. SSIM,' in Proceedings of the 2010 20th International Conference on Pattern Recognition (ICPR) , pp. 2366-2369, August 2010. doi: 10.1109/ICPR.2010.579
- [16] Z. Wang, A.C. Bovik, H.R. Sheikh, and E.P. Simoncelli, 'Image quality assessment: from error visibility to structural similarity,' IEEE Transactions on Image Processing , vol. 13, no. 4, pages 600-612, April 2004.
- [17] ImageMagick. [Online]. Available: https://imagemagick.org/index.php
- [18] D. Lee, B. Hyun, T. Kim, and M. Rhu, 'PIM-MMU: A memory management unit for accelerating data transfers in commercial pim systems,' in Proceedings of the 57th IEEE/ACM International Symposium on Microarchitecture (MICRO) , pp. 627-642, November 2024.
- [19] nvJPEG. [Online]. Avaliable: https://developer.nvidia.com/nvjpeg