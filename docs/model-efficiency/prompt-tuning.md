---
title: "Prompt-Tuning: 입력 프롬프트 최적화를 통한 경량 적응"
parent: "PEFT: 파라미터 효율적 미세 조정 개요"
grand_parent: 모델 경량화
nav_order: 5
layout: default
permalink: /model-efficiency/prompt-tuning/
mathjax: false
---

# Prompt-Tuning 개요

Prompt-Tuning은 사전학습된 언어 모델에 **태스크 특화 텍스트 프롬프트 또는 임베딩 토큰을 추가**하여 원하는 출력을 이끌어내는 파라미터 효율적 미세 조정 기법이다. 모델의 본체 파라미터는 동결하고, 입력에 붙이는 프롬프트만 학습하므로 학습 비용이 매우 낮다.

- 원본 모델 파라미터 동결, 프롬프트 토큰 임베딩만 업데이트
- 전체 파라미터의 0.1% 이하만 훈련하면서도 다양한 다운스트림 태스크에 적용 가능
- 자연어 프롬프트(Discrete)나 연속 임베딩 벡터(Continuous) 방식 모두 활용 가능

## 왜 Prompt-Tuning을 사용하는가

- **저비용 실험**: 프롬프트만 조정하면 되므로 빠른 프로토타이핑이 가능하다.
- **언어 모델 활용성 극대화**: LLM이 사전학습된 지식을 보다 효과적으로 끌어낼 수 있다.
- **멀티태스크/도메인 확장**: 도메인별 프롬프트를 교체하는 것만으로 모델 재학습 없이 다양한 태스크를 지원한다.
- **해석력**: 사람이 읽을 수 있는 프롬프트로 설계하면 모델 행동을 직관적으로 이해하기 쉽다.

## 구조와 동작 방식

Prompt-Tuning은 새로운 작업에 적응시키기 위해 **가장 적합한 특정 Prompt(입력 텍스트) 매개변수를 찾아 모델을 튜닝하는 과정**이다. 이는 LLM이 특정 작업을 수행하도록 유도하는 텍스트나 명령을 의미한다.

Prompt-Tuning은 주어진 작업의 요구사항과 목적에 따라 모델의 입력을 조정하여 성능을 최적화한다. 이를 통해 특정 작업에 대한 성능을 개선하거나 모델이 원하는 방식으로 동작하도록 할 수 있다.

| Prompt-Tuning 구조 개념도 | 입력 예시 |
| --- | --- |
| ![Prompt-Tuning 구조 개념도](/assets/images/prompt-tuning-diagram.png){: width="280" } | `"Summarize: " ⊕ Input Text` |

1. **프롬프트 준비**  
   - 문자열 프롬프트: `"Classify sentiment: "` 처럼 명시적으로 지시하는 문장을 앞에 붙인다.  
   - 임베딩 프롬프트: 학습 가능한 연속 벡터를 입력 앞에 추가한다.
2. **입력 결합**  
   - 프롬프트 토큰과 실제 입력 텍스트를 연결(concatenate)한다.
3. **모델 추론**  
   - 사전학습된 Transformer가 결합된 입력을 처리한다.
4. **프롬프트 업데이트**  
   - 역전파 시 프롬프트 임베딩만 업데이트하고, 다른 파라미터는 동결한다.

## Prompt-Tuning vs Prefix-Tuning

| 항목 | Prompt-Tuning | Prefix-Tuning |
| --- | --- | --- |
| 입력 처리 | 프롬프트 토큰 앞에 붙임 | Key/Value까지 포함한 연속 prefix 삽입 |
| 적용 범위 | 주로 입력 토큰 레벨 | Transformer 각 레이어의 attention에 영향 |
| 장점 | 구현 간단, 해석 쉬움 | 더 깊은 레이어까지 제어 가능 |
| 한계 | 안정적 수렴 어려울 수 있음 | prefix 길이에 따라 지연 증가 |

## 장점과 한계

**장점**
- 파라미터 효율이 높아 대규모 모델에도 부담 없이 적용 가능.
- 도메인 전환이 빠르고, 프롬프트만 교체하면 핫스왑 가능.
- 사람이 설계한 지시문과 결합해 직관적인 제어가 가능.

**한계**
- 프롬프트 설계에 따라 성능 변동이 크다.
- 최적 프롬프트를 찾기 위한 탐색이 필요하며, 자동화 도구나 전략이 요구된다.
- 복잡한 추론이나 구조적 이해가 필요한 태스크에서는 성능이 제한될 수 있다.

## 하이퍼파라미터/설계 팁

1. **프롬프트 길이**: 5~20 토큰 범위에서 실험하며, 모델과 태스크 특성에 맞게 조정.
2. **프롬프트 형태**: 자연어 명령, 예시(Few-shot), 학습 가능한 연속 토큰 등 다양한 형태를 혼합 가능.
3. **학습률**: 임베딩 프롬프트는 1e-3~5e-3 수준의 학습률이 자주 사용된다.
4. **정규화**: Dropout 또는 LayerNorm을 프롬프트 경로에 적용해 과적합 방지.
5. **자동 프롬프트 탐색**: AutoPrompt, RL 기반 최적화 등 자동화된 프롬프트 설계 기법 활용.

## 실무 적용 체크리스트

1. 태스크마다 기본 프롬프트 템플릿을 정의하고 실험한다.
2. 프롬프트 임베딩만 업데이트하도록 optimizer 파라미터 그룹을 구성한다.
3. 프롬프트와 입력 길이를 고려해 최대 시퀀스 길이를 설정한다.
4. 성능이 떨어질 경우 프롬프트 길이, 초기화 방법, Few-shot 예시 등을 조정한다.
5. Prompt-Tuning과 Prefix/Adapter/LoRA를 조합해 하이브리드 전략을 시도해 본다.

---

### 참고 자료

- Li, Xiang Lisa, and Percy Liang. "Prefix-Tuning: Optimizing Continuous Prompts for Generation." (2021)
- Lester, Brian et al. "The Power of Scale for Parameter-Efficient Prompt Tuning." (2021)
- Shin, Taylor et al. "Autoprompt: Eliciting Knowledge from Language Models with Automatically Generated Prompts." (2020)

