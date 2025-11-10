---
title: Home
layout: home
nav_order: 0
description: "AI 연구와 엔지니어링 기록"
permalink: /
---

# STARTMKK Notes
{: .fs-9 }

AI 연구와 엔지니어링 실험을 기록하는 공간입니다. 모델 미세 조정, 경량화, 온디바이스 개발 경험을 중심으로 한 기술 노트를 정리합니다.
{: .fs-6 .fw-300 }

[LoRA 문서 바로가기](/model-efficiency/lora/){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[GitHub 저장소](https://github.com/starmkk/starmkk.github.io){: .btn .fs-5 .mb-4 .mb-md-0 }

---

## 최신 글

- [Prompt-Tuning: 입력 프롬프트 최적화를 통한 경량 적응](/model-efficiency/prompt-tuning/)  
  텍스트 프롬프트 또는 연속 임베딩 토큰을 학습해 언어 모델을 빠르게 태스크에 적응시키는 전략.
- [Prefix-Tuning: 연속 프롬프트를 통한 경량 미세 조정](/model-efficiency/prefix-tuning/)  
  연속 prefix 벡터로 모델의 attention을 조정해 Full Fine-Tuning에 근접한 성능을 노리는 기법.
- [Adapter: Transformer 병목 어댑터를 통한 파라미터 효율화](/model-efficiency/adapter-overview/)  
  Transformer에 병목 모듈을 삽입하는 어댑터 계열 기법의 구조, 장단점, 하이퍼파라미터 가이드.
- [PEFT: 파라미터 효율적 미세 조정 개요](/model-efficiency/peft-overview/)  
  Adapter, LoRA, Prefix-Tuning 등 대표 PEFT 기법을 비교하고 선택 가이드를 정리한 개요.
- [LoRA: 대규모 언어 모델을 위한 경량 적응 기법](/model-efficiency/lora/)  
  LoRA의 등장 배경, 저랭크 구조, 적용 효과, 실무 체크리스트를 정리한 심화 가이드.

## 다루는 주제

- 대규모 언어 모델(LLM) 미세 조정 전략
- 파라미터 효율화 기법(LoRA, QLoRA, Adapter 등)
- 머신러닝 파이프라인 자동화 및 운영
- 논문 리뷰와 코드를 통한 재현 실험

---

새 글과 실험 기록은 주기적으로 업데이트될 예정입니다. 의견이나 제안은 GitHub 이슈로 남겨 주세요!

