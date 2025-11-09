---
title: "PEFT: 파라미터 효율적 미세 조정 개요"
parent: 모델 경량화
nav_order: 1
layout: default
permalink: /model-efficiency/peft-overview/
mathjax: true
---

# PEFT(Parameter-Efficient Fine-Tuning) 개요

Parameter-Efficient Fine-Tuning(PEFT)은 대규모 사전학습 모델의 파라미터 대부분을 동결한 채, **작은 추가 모듈이나 저랭크 파라미터**만 학습하여 특정 작업에 적응시키는 접근법이다. 전체 파라미터를 업데이트하는 **Full Fine-Tuning** 대비 학습 비용이 낮고, 여러 태스크를 하나의 기반 모델(Base Model) 위에서 빠르게 전환할 수 있다는 장점이 있다.

## 왜 PEFT가 필요한가

- **연산 및 메모리 비용 절감**: 수억~수천억 개의 파라미터를 가진 LLM·멀티모달 모델은 GPU 메모리와 연산량이 막대하다.
- **모델 재활용성 증가**: 다수의 태스크마다 별도의 전체 모델을 유지하는 대신, 공통 기반 모델에 태스크별로 작은 어댑터만 추가하면 관리가 쉬워진다.
- **빠른 실험과 배포**: 학습해야 할 파라미터 수가 적으므로 학습 속도가 빠르고, 응용 도메인별로 핫스왑(Hot-swap) 가능하다.
- **데이터 제한 상황 대응**: 제한된 도메인 데이터로도 안정적인 미세 조정이 가능하다.

## 대표적인 PEFT 기법

| 기법 | 핵심 아이디어 | 장점 | 고려 사항 |
| --- | --- | --- | --- |
| **Adapter / Bottleneck Adapter** | Transformer 블록 사이에 작은 MLP 어댑터 모듈 추가 | 안정성 높음, 기존 구조 보존 | 추가 레이어로 추론 지연 증가 |
| **LoRA (Low-Rank Adaptation)** | 선형 계층 가중치를 저랭크 행렬 $A, B$로 분해해 업데이트 | 파라미터 수 극소, 추론 시 병합 가능 | 랭크 $r$ 선택에 따라 성능 좌우 |
| **Prefix-Tuning / Prompt-Tuning** | 입력 토큰 앞에 학습 가능한 prefix/prompt 벡터 추가 | 원본 파라미터 완전 동결, 텍스트 생성에 적합 | 프롬프트 길이에 제약, 일부 태스크에 한정 |
| **P-Tuning / P-Tuning v2** | 연속 벡터 프롬프트(prefix)를 Ladder 구조로 확장 | GPT-3 등에서 높은 효율 | 구현 복잡, 구조 커스터마이징 필요 |
| **BitFit** | Bias 파라미터만 학습 | 구현 매우 간단 | 성능 향상 폭이 제한적 |
| **IA³ (Infused Adapter by Inhibiting and Amplifying Inner Activations)** | 어텐션과 FFN의 key/value에 스칼라 게이트 삽입 | LoRA와 유사한 파라미터 효율 | 구현과 하이퍼파라미터 튜닝 난이도 |
| **QLoRA** | 4비트 양자화 + LoRA | 48GB 이하 GPU에서도 65B 모델 학습 가능 | 양자화로 인한 정밀도 손실 가능성 |
| **LLaMA-Adapter / LLaMA-Adapter V2** | LLaMA 모델에 비지도 레벨 Adapter 삽입, 비전-언어 확장 | 멀티모달에도 적용 쉬움 | 오픈소스 생태계 의존 |

## LoRA와의 비교

| LoRA 구조 개념도 | 수식 표현 |
| --- | --- |
| ![LoRA 구조 개념도](/assets/images/lora-diagram.png){: width="280" } | $$W x + B A x$$ |

LoRA는 선형 계층을 저랭크 행렬로 분해해 업데이트하는 방식으로, PEFT 가운데서도 **추론 시 오버헤드가 거의 없고 병합이 용이**하다는 장점이 있다. 반면, **Adapter**는 다양한 구조를 주입해야 하고, **Prompt 계열**은 입력 토큰 길이 제약이 있으므로 태스크 특성에 따라 적합한 기법을 선택해야 한다.

## 기법 선택 가이드

1. **데이터 크기**: 데이터가 적으면 Prompt-Tuning·BitFit처럼 경량 기법을, 데이터가 충분하면 LoRA나 IA³처럼 구조화된 기법을 선택한다.
2. **추론 지연 허용치**: 프로덕션 환경에서 지연에 민감하다면 LoRA/QLoRA처럼 병합 가능한 방법이 낫다.
3. **멀티 태스크 관리**: Adapter 계열은 도메인별 모듈을 독립적으로 관리하기 쉬우므로 MLOps 환경에서 사용한다.
4. **하드웨어 여건**: VRAM이 매우 제한적이라면 QLoRA(양자화+LoRA) 또는 Prompt-Tuning 계열을 고려한다.
5. **엔지니어링 난이도**: BitFit, Prompt-Tuning은 구현이 간단하고, IA³나 P-Tuning은 커스텀 구현 지식이 필요하다.

## 향후 트렌드

- **복합 기법**: Prefix-Tuning + LoRA 같이 여러 PEFT 기법을 조합하여 더 높은 성능을 노리는 사례가 증가한다.
- **멀티모달 확장**: 이미지·음성·비디오 모델에서도 PEFT 기법이 활발히 연구 중이다.
- **온디바이스(Inference at Edge)**: 모바일·엣지 환경에서도 경량화된 PEFT 모듈만 교체하여 빠르게 서비스할 수 있다.
- **자동화 파이프라인**: AutoML/AutoPEFT 툴이 등장하며, 태스크별로 최적의 PEFT 설정을 자동 탐색하는 흐름이 나타난다.

---

### 참고 자료

- Pfeiffer, Jonas et al. "AdapterHub: A Framework for Adapting Transformers." (2020)
- Hu, Edward J., et al. "LoRA: Low-Rank Adaptation of Large Language Models." (2021)
- Lester, Brian et al. "The Power of Scale for Parameter-Efficient Prompt Tuning." (2021)
- Dettmers, Tim et al. "QLoRA: Efficient Finetuning of Quantized LLMs." (2023)
- Lialin, Vladislav et al. "BitFit: Simple Parameter-efficient Fine-tuning for Transformers." (2023)

