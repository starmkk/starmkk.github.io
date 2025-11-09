---
title: "LoRA: 대규모 언어 모델을 위한 경량 적응 기법"
parent: 모델 경량화
nav_order: 1
layout: default
permalink: /model-efficiency/lora/
mathjax: true
---

# LoRA란 무엇인가

LoRA(Low-Rank Adaptation)는 대규모 언어 모델(LLM)에서 적은 리소스를 활용해 모델을 미세 조정할 수 있도록 고안된 기법입니다. 이 기술은 기존 모델의 성능을 유지하며, 학습 가능한 저랭크 행렬만을 조정해 효율성을 극대화합니다.

---

## 암기를 위한 핵심 포인트

### 1. 원리와 구조
- **모델 파라미터의 동결**: 원본 가중치 $W$는 변경하지 않고 유지.
- **저랭크(Low-Rank) 행렬 학습**: $W x + B A x$ 형태로 가중치를 분해해 두 행렬 $A$, $B$만 학습.
  - 학습 가능한 파라미터 수를 $d \times k$에서 $r \times (d + k)$로 감소.
  - 기존 모델의 무결성을 보장하며, 메모리 및 계산 효율성 향상.

### 2. 적용 위치
- **Self-Attention 계층**: Query, Value 프로젝션에서 동작 최적화.
- **MLP 계층**: 첫 번째 선형 변환에 적용해 표현력 증대.
- **다중 도메인 확장**: LoRA 가중치 로딩만으로 새로운 도메인에 빠르게 전환 가능.

### 3. 효과
- **파라미터 효율성**: 파라미터 크기를 줄여 학습 및 저장 비용 절감. 
  - 예: GPT-3 7B 모델 기준으로 $r = 8$ 사용 시 전체 추가 파라미터는 약 0.05%.
- **학습 안정성**: 적은 에폭만으로도 기존 전체 미세 조정과 유사한 성능 확보.

### 4. 실무 적용 체크리스트
- **랭크 및 스케일 선택**: $r = 8$, $\alpha = 16$ 기본값 추천.
- **스케일링 초기화**: $\alpha/r$ 값으로 학습 초반 안정성을 개선.
- **혼합 정밀도(FP16/BF16)**: 추가 효율성 극대화를 위한 전략.
- **추론 배포 전략**: 합쳐진 형식(merged) 또는 동적 주입(On-the-Fly) 방식 선택.

---

## 연구 배경과 동기

### 모델의 제한 요소 해결
1. **대규모 모델의 활용 비용**: GPT-3 등 초대형 모델의 등장으로 인해 전통적인 학습 기법은 높은 비용 문제를 초래.
2. **다중 작업(Multi-Task) 요구**: 유연하게 다중 도메인에 적응할 수 있는 새로운 미세 조정 방법 필요.
3. **기존 Adapter의 한계 극복**: 추론 지연(latency)과 메모리 사용량 증가 문제를 해결.

---

## 시험 대비를 위한 핵심 키워드 정리

### LoRA 한눈에 보기
- **저랭크(Low-Rank) 학습**: $W x + B A x$ 형태로 경량화된 미세 조정 방식.
- **효율성**: 업데이트 파라미터 수 0.05% 수준, 메모리 절감.
- **적용 위치**: Self-Attention, MLP, 다중 도메인 확장.
- **기술적 특징**: $r$, $\alpha$, 스케일링, 초기화 전략.
- **적용 효과**: 빠른 전환(Hot-swapping), 안정적 학습.
- **배경**: 대규모 LLM 확산, Multi-task/Domain 요구 대응, Adapter 한계 극복.

---

## 참고 문헌
- Hu, Edward J., et al. "LoRA: Low-Rank Adaptation of Large Language Models." *arXiv preprint arXiv:2106.09685* (2021).
- Dettmers, Tim, et al. "QLoRA: Efficient Finetuning of Quantized LLMs." *arXiv preprint arXiv:2305.14314* (2023).
- Xu, Canwen, et al. "Parameter-Efficient Fine-Tuning for Speech and Vision Models." *Proceedings of the IEEE/CVF* (2023).