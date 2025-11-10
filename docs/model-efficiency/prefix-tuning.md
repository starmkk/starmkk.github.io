---
title: "Prefix-Tuning: 연속 프롬프트를 통한 경량 미세 조정"
parent: "PEFT: 파라미터 효율적 미세 조정 개요"
grand_parent: 모델 경량화
nav_order: 4
layout: default
permalink: /model-efficiency/prefix-tuning/
mathjax: true
---

# Prefix-Tuning 개요

Prefix-Tuning은 사전학습된 언어 모델의 입력에 **학습 가능한 연속 벡터(prefix)**를 붙여서 특정 태스크에 맞게 모델을 조정하는 파라미터 효율적 미세 조정 기법이다. 원본 모델의 가중치를 업데이트하지 않고 prefix 파라미터만 학습하므로, Full Fine-Tuning 대비 훨씬 적은 리소스로 태스크 적응이 가능하다.
- 모델의 parameter를 freeze하고, 입력의 task-specific한 벡터 sequence인 prefix에 해당하는 블록의 파라미터만 update
- 전체 파라미터 중 0.1%가량만을 훈련하였지만, 효과적으로 동작, 또한 다양한 작업에 대해 다른 prefix가 활용됨으로 domain 침범 하지 않음

## 왜 Prefix-Tuning을 사용하는가

- **파라미터 효율성**: 수백억 개 파라미터를 가진 모델에서도 prefix 수천 개만 학습하면 태스크별 성능을 확보할 수 있다.
- **구현 간결성**: 입력 앞에 붙이는 벡터를 학습하면 되므로 구조 변경이 최소화된다.
- **다중 태스크 관리 용이**: 태스크별 prefix만 교체하면 동일 모델을 여러 도메인에 빠르게 적용 가능하다.
- **언어 생성 태스크에 특화**: 자연어 생성, 요약, 번역 등에서 높은 성능을 보이며 GPT-3 수준의 모델에서도 활용된다.

## 구조와 동작 원리

| Prefix 구조 개념도 | 수식 표현 |
| --- | --- |
| ![Prefix-Tuning 구조 개념도](/assets/images/prefix-tuning-diagram.png){: width="280" } | $$\tilde{X} = [P; X], \quad h_t = \text{Transformer}(\tilde{X})$$ |

- **Prefix 벡터 $P$**: 학습 가능한 연속 벡터들의 집합으로, 길이 $k$의 가상 토큰을 구성한다.
- **입력 $X$**: 실제 텍스트 토큰 시퀀스.
- **결합 입력 $\tilde{X}$**: prefix와 입력 시퀀스를 연결(concatenate)한 벡터.
- **Transformer 연산**: 기본 모델의 Self-Attention이 prefix를 attention key/value로 인식하면서, 태스크 특화 정보를 주입한다.

이 과정에서 prefix가 모델의 attention 경로를 조정하므로, 별도의 구조 변경 없이 태스크별 행동을 유도할 수 있다.

## Prefix vs Prompt-Tuning

- **Prefix-Tuning**: key/value 벡터를 포함한 연속 벡터 전체를 학습하며, Transformer 내부에 영향을 준다.
- **Prompt-Tuning**: 입력 토큰 앞에 문자열 형태나 임베딩 벡터를 추가하되 key/value는 원본을 사용한다.
- **P-Tuning v2**: Prefix-Tuning을 Ladder 구조로 확장해 더 깊은 레이어에 prefix를 삽입하고, Stability를 향상시킨 변형이다.

## 장점과 한계

**장점**
- 특정 태스크에서 Full Fine-Tuning에 버금가는 성능.
- 태스크마다 다른 prefix를 로드해 핫스왑이 가능.
- 파이프라인이 단순하므로 구현이 빠름.

**한계**
- prefix 길이가 길수록 추론 지연이 증가할 수 있다.
- 프롬프트 길이 제한이 있는 모델에서는 사용에 제약이 있다.
- 일부 추론형 태스크에서는 Adapter나 LoRA 대비 성능이 떨어질 수 있다.

## 하이퍼파라미터 가이드

1. **Prefix 길이 $k$**: 5~20 사이에서 시작해 태스크 난이도에 따라 조정.
2. **학습률**: prefix 파라미터는 보통 1e-3~5e-3 범위에서 학습.
3. **초기화**: 랜덤 초기화 대신, 기존 임베딩 평균이나 관련 키워드 임베딩으로 초기화하면 수렴이 빠를 수 있다.
4. **층별 배치**: Transformer의 여러 레이어에 prefix를 삽입하면 성능을 높일 수 있지만 메모리 사용량이 증가한다.
5. **정규화**: LayerNorm/Dropout을 prefix 경로에 추가해 과적합을 방지한다.

## 실무 적용 체크리스트

1. 데이터 전처리 단계에서 prefix 길이를 고려해 입력 길이를 조정한다.
2. 학습 중 prefix만 업데이트되도록 optimizer 파라미터 그룹을 명시한다.
3. 태스크별 prefix를 별도 파일로 저장해 MLOps 파이프라인에 통합한다.
4. 평가 시 prefix 없이도 기본 모델이 동작하는지 sanity check를 수행한다.
5. Prefix 길이와 학습률 조합을 다르게 한 실험을 기록해 최적 지점을 찾는다.

---

### 참고 자료

- Li, Xiang Lisa, and Percy Liang. "Prefix-Tuning: Optimizing Continuous Prompts for Generation." (2021)
- Lester, Brian et al. "The Power of Scale for Parameter-Efficient Prompt Tuning." (2021)
- Liu, Peng et al. "P-Tuning v2: Prompt Tuning Can Be Comparable to Fine-tuning Universally Across Scales and Tasks." (2022)

