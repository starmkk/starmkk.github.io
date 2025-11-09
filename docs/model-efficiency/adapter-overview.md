---
title: "Adapter: Transformer 병목 어댑터를 통한 파라미터 효율화"
parent: 모델 경량화
nav_order: 2
layout: default
permalink: /model-efficiency/adapter-overview/
mathjax: false
---

# Adapter / Bottleneck Adapter

Adapter는 Transformer 블록 사이에 **작은 병목(bottleneck) 구조**를 삽입하여 태스크별 표현을 학습하는 대표적인 파라미터 효율적 미세 조정(PEFT) 기법이다. 기존 모델의 가중치는 동결하고, 삽입된 어댑터 모듈만 학습하기 때문에 전체 파라미터 대비 매우 적은 수의 파라미터만 업데이트하면 된다.

## 구조와 동작 원리

| Adapter 구조 개념도 | Forward Pass |
| --- | --- |
| ![Adapter 구조 개념도](/assets/images/adapter-diagram.png){: width="280" } | $$h_{\text{out}} = h + W_u \sigma(W_d h)$$ |

- **Step 0. 입력 확보**: 각 Transformer 블록에서 Residual 연결 전의 은닉 상태 $h \in \mathbb{R}^d$를 입력으로 사용한다.
- **Down Projection ($W_d$)**: 원래 차원 $d$에서 작은 병목 차원 $m$($m \ll d$)으로 축소한다.
- **Nonlinearity**: ReLU, GELU 등 활성화 함수를 적용한다.
- **Up Projection ($W_u$)**: 다시 원래 차원 $d$로 확장하여 Residual 연결과 더한다.
- **Step 4. Residual 결합**: 어댑터 출력 $W_u \sigma(W_d h)$를 원 본 출력 $h$와 더해 $h_{\text{out}}$을 만든다.

이 방식은 `Adapter = Down → Activation → Up + Residual` 순서를 따르며, GPT/BERT 같은 Transformer의 Self-Attention/FFN 블록 사이에 삽입된다. 입력 은닉 상태 $h$가 어댑터를 통해 축소·확장되면서 태스크 특화 표현을 학습하며, 최종 출력은 $h_{\text{out}}$으로 Residual 연결을 통해 안정성을 확보한다.

### 단계별로 살펴보기

1. **은닉 상태 추출**  
   - Self-Attention 또는 Feed-Forward 블록의 입력/출력 위치에서 은닉 벡터 $h$를 얻는다.
2. **Down Projection**  
   - 어댑터의 첫 번째 선형 변환 $W_d \in \mathbb{R}^{m \times d}$가 $h$를 낮은 차원 $m$으로 투사한다.  
   - 작은 차원에서만 파라미터를 학습하므로 메모리와 연산을 절약한다.
3. **활성화(Nonlinearity)**  
   - $h_d = W_d h$에 대해 $\sigma(h_d)$를 적용해 비선형 표현을 학습한다.
   - ReLU, GELU 등이 사용되며, 도메인에 따라 Swish 등 다른 함수도 시도된다.
4. **Up Projection**  
   - $W_u \in \mathbb{R}^{d \times m}$이 다시 원래 차원으로 확장한다: $h_u = W_u \sigma(h_d)$.
   - 확장된 벡터는 태스크별 특성을 담은 보정값(Residual) 역할을 한다.
5. **Residual 결합**  
   - 원본 은닉 상태와 어댑터 출력을 더함으로써 $h_{\text{out}} = h + h_u$를 얻는다.  
   - 원래 표현은 유지하면서, 필요한 정보만 미세 조정한다.

### 어댑터 삽입 위치 예시

- **Self-Attention 후**  
  1. Multi-Head Attention → Residual & LayerNorm  
  2. **Adapter (Down→Activation→Up)**  
  3. Residual 결합 후 LayerNorm  
- **Feed-Forward 네트워크 후**  
  1. MLP (두 개의 선형 + 활성화)  
  2. Residual & LayerNorm  
  3. **Adapter 모듈**  
  4. Residual 결합 후 LayerNorm  
- 필요한 위치마다 어댑터를 삽입해도 되며, Houlsby Adapter처럼 어텐션 블록과 FFN 블록에 각각 배치하는 변형도 있다.

### 학습 흐름 요약

1. 사전학습된 Transformer 파라미터는 모두 동결한다.
2. Adapter 모듈의 파라미터(주로 $W_d$, $W_u$)만 학습한다.
3. 역전파 시 Adapter를 통과한 Gradient만 업데이트되므로 학습 속도가 빠르다.
4. 학습 완료 후에는 태스크별 Adapter만 저장하면 되므로 배포/교체가 쉽다.

## 주요 변형

- **Bottleneck Adapter**: 가장 일반적인 형태로, $d \rightarrow m \rightarrow d$ 구조를 사용한다.
- **Parallel Adapter**: Residual에 병렬로 어댑터 출력을 더하는 방식으로, 원본 출력과 병렬 경로를 구성한다.
- **Compactor, Houlsby Adapter**: 어텐션과 FFN에 각각 별도 어댑터를 넣거나, 레이어마다 다른 구성을 쓰는 등 위치/구조를 다양화한 변형.
- **AdapterFusion / AdapterMix**: 여러 도메인의 어댑터를 동시에 활성화해, 가중치를 학습하거나 앙상블처럼 활용한다.

## 장점

- **안정성**: Residual 연결을 활용하며 전체 모델 구조를 유지하므로 학습이 안정적이다.
- **모듈성**: 태스크마다 서로 다른 어댑터 가중치를 쉽게 저장·교체할 수 있다.
- **확장성**: 멀티태스크 환경에서 어댑터만 스왑하면 다양한 도메인을 커버할 수 있다.
- **조합 가능성**: LoRA나 Prompt-Tuning과 함께 사용하는 하이브리드 설계도 가능하다.

## 한계

- **추론 지연 증가**: 각 레이어마다 추가 연산이 늘어나 추론 속도가 다소 느려질 수 있다.
- **메모리 사용량**: LoRA처럼 저랭크 행렬에 비해 메모리 절감 폭이 작을 수 있다.
- **세밀한 튜닝 필요**: 병목 차원 $m$, 배치 정규화 여부 등 하이퍼파라미터 조정이 성능에 영향을 준다.

## 하이퍼파라미터 가이드

- **병목 차원 $m$ 선택**: 일반적으로 $m = 4\dots64$ 사이에서 모델 크기와 태스크 복잡도에 맞춰 조절한다.
- **삽입 위치**: Self-Attention 블록 후, Feed-Forward 블록 후에 각각 삽입하는 것이 일반적이다.
- **학습률**: 어댑터 파라미터만 학습하므로 1e-4~5e-4 범위의 상대적으로 큰 학습률을 적용해도 안정적이다.
- **정규화**: LayerNorm이나 Dropout을 어댑터 내부에 추가해 과적합을 방지할 수 있다.

## 적용 사례

- **AdapterHub**: 다양한 사전학습 모델과 어댑터 체크포인트를 공유하는 생태계.
- **MAD-X**: 다국어 모델에서 Adapter를 활용해 언어 간 전이 학습을 수행.
- **멀티모달 어댑터**: CLIP, Flamingo 등에서 이미지/텍스트를 연결하기 위해 어댑터를 활용한 사례가 보고되고 있다.

---

### 참고 자료

- Houlsby, Neil et al. "Parameter-Efficient Transfer Learning for NLP." (2019)
- Pfeiffer, Jonas et al. "AdapterFusion: Non-Destructive Task Composition for Transfer Learning." (2020)
- Pfeiffer, Jonas et al. "AdapterHub: A Framework for Adapting Transformers." (2020)

