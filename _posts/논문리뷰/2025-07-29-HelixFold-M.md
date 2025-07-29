---
title: "Precise Antigen-Antibody Structure Predictions Enhance Antibody Development with HelixFold-Multimer"
last_modified_at: 2025-07-29
categories:
  - 논문리뷰
tags:
  - Structural Bioinformatics
  - Antibody
  - arXiv
use_math: true
classes: wide
---

> arXiv 2024. [[Paper](https://arxiv.org/abs/2412.09826)]  [[Github](https://github.com/PaddlePaddle/PaddleHelix)] [[Page](https://paddlehelix.baidu.com/)]   
> Jie Gao, Jing Hu, Lihang Liu, Yang Xue, Kunrui Zhu, Xiaonan Zhang, Xiaomin Fang  
> 13 December 2024 



# Introduction
정확한 항원-항체 구조는 면역학의 발전에 중요한 부분이다. AlphaFold, RoseTTAFold와 같은 단백질 구조 예측 모델의 발전에도 불구하고, 항원-항체 복합체 구조의 예측은 여전히 부족한 부분으로 남아있다. 또한 AFSample, AlphaFold3와 같은 시도들은 많은 구조 예측으로 이를 극복하려 했지만, 컴퓨팅 자원의 소모가 매우 크다. IgFold, DeepAb, AbodyBuilder와 같은 항원-항체 특화 모델들도 존재한다. 이들은 CDR(Complementarity-Determining Region) 구조 예측의 정확도를 향상시킨 모델들인데, 항체 구조 예측에는 강하지만 복합체 구조 예측은 여전히 과제로 남아있다. 이에 저자들은 항원-항체 구조 예측의 성능을 끌어올린 HelixFold-Multimer를 제안한다.  

# Antigen-Antibody Structure Prediction
HelixFold-Multimer는 기본적으로 AlphaFold-Multimer의 Structure Module을 약간 변형한 구조를 따른다. 학습 데이터 또한 PDB + Self distillation data로 구성되어 일반적인 단백질 구조 예측 성능을 갖추었다. 그리고 항원-항체 구조 데이터에 대한 fine tuning을 진행했다. 항체 서열들에 대해서는 Antiref 데이터베이스를 추가로 활용해 MSA를 더 풍부하게 만들었다.  


<p align="center">
  <img src="{{'/assets/img/HelixFold-M/HelixFold-M-fig1.png' | relative_url}}" width="90%">
</p>

## Overall Accuracy
항원-항체 구조 예측의 성능을 평가하기 위해, 141개의 recent PDB 구조들로 평가 데이터셋을 구성했다. 비교 모델로는 RoseTTAFold, AlphaFold-Multimer, AlphaFold3를 선정했고, 해당 모델들보다 더 나은 결과를 보여준다. 


## Accuracy across Antibody Species
항체는 종마다 isotypes, contant region sequences 그리고 glycosylation pattern에서 차이를 보이는데, 이런 구조적 다양성은 그들의 안정성, 특이성과 기능에 큰 영향을 미친다. 따라서 저자들은 종 별 항원-항체 구조 예측에 대한 성능을 평가했다. HelixFold-Multimer는 모든 종에 있어서 AlphaFold3보다 나은 성능을 보여주었고, 두 모델 모두 Homo sapiens와 Mus musculus에서 더 나은 성능을 보였다. 이는 두 종에 해당하는 항체 구조 데이터가 다른 종들에 비해 풍부했기 때문이다. 


## Precision of the Confidence Scores
저자들은 HelixFold-Multimer의 confidence score와 실제 정확도 간의 관계도 분석했다. DockQ 점수와 confidence score, ipTM, pLDDT를 비교한 결과는 다음과 같다.  

| Score | Pearson correlation |
|:-------|:---------------------|
| Confidence | 0.664 |
| ipTM | 0.658 |
| pLDDT | 0.344 |


# Epitope-Specific Antigen-Antibody Structure Prediction
항원-항체 구조 예측에서 항원의 epitope 정보가 주어지면 유의미한 정확도 향상을 보였다. 학습 시에 EvoFormer의 attention과 Structure Module을 통해 항원의 epitope 범위에 대한 정보를 추가로 입력했다. 또한 추론 시에도 epitope 범위를 모델에 입력함으로써 항원-항체 구조 예측 성능을 향상시켰다.  

<p align="center">
  <img src="{{'/assets/img/HelixFold-M/HelixFold-M-fig2.png' | relative_url}}" width="90%">
</p>


## Accuracy Improvement with Specified Epitopes
Epitope 정보가 없을 때, DockQ 점수가 낮았던 타켓들도 epitope 정보가 주어지면 눈에 띄는 DockQ 점수의 향상이 있었다. Testset은 학습 데이터에 있는 서열과의 low homology(sequence identity 40% 이하), medium homology(sequence identity 95% 이하), high homology(sequence identity 95% 이상)으로 구성했는데, low homology 데이터에 대한 성능 향상폭이 가장 컸다. 


## Effect of Epitope Completeness
저자들은 epitope 정보의 완전성이 성능에 미치는 영향도 분석했는데, 주어지는 epitope 정보가 많을수록(잔기의 범위가 넓을수록) DockQ 점수가 꾸준히 상승했다. 이 때, epitope는 항체에 가장 가까운 항원의 잔기들을 기준으로 선정되었다. 하지만 epitope 정보가 적어지면 때때로 epitope 정보를 주지 않았을 때보다 성능이 나빠졌다. 저자들은 이 현상에 대해 모델이 소수의 epitope에 지나치게 집중해(overfitting) 전체 복합체의 정보를 활용하지 못함을 원인으로 지적했다. 


# Antigen-Antibody Interaction Prediction
실제 항체 의약품 개발에서 항원-항체의 결합 유무, 결합 강도를 예측하는 것은 스크리닝 비용을 크게 줄인다. 이에 저자들은 HelixFold-Multimer의 구조와 confidence score(ipTM)의 활용 방안들을 제시한다.


<p align="center">
  <img src="{{'/assets/img/HelixFold-M/HelixFold-M-fig3.png' | relative_url}}" width="90%">
</p>


## Confidence Metrics of HelixFold-Multimer for Interaction Prediction
HelixFold-Multimer의 ipTM을 통해 항원-항체 결합을 판단할 수 있는지에 대한 분석을 진행했다. 우선 아래와 같은 4가지 항원을 선정하고, OAS(Observed Antibody Space), Thera-SAbDab과 같은 데이터베이스와 항원-항체 실험 데이터를 참고해 양성 테스트 데이터셋을 구성했다. 음성 테스트 데이터셋은 각 항원에 대해 OAS에서 무작위 항체 서열 1,000개를 수집했다. 그 결과 AlphaFold-Multimer, ESM2보다 월등히 뛰어난 AUC 점수를 보여주었다. 이는 HelixFold-Multimer의 ipTM을 기준으로 한 결합 여부 판단의 신뢰성을 보여준다. 

| 항원 이름              | 설명                                        | 항체 출처                        |
| ------------------ | ----------------------------------------- | ---------------------------- |
| **SARS-CoV-2 RBD** | 코로나바이러스 스파이크 단백질의 receptor-binding domain | OAS                          |
| **Lysozyme**       | 고전적인 모델 항원 (구조·항체 풍부)                     | 항-Lysozyme 실험 논문/DB          |
| **VEGF**           | 혈관 생성 인자, 항암 항체 타겟                        | 항-VEGF 항체 DB                 |
| **PD-1**           | 면역관문 단백질 (면역항암제 타겟)                       | **Thera-SAbDab** (치료용 항체 DB) |  


  
또한 ipTM 기준 상위 5%(1%) 타겟 내에 실제 결합하는 타겟이 얼마나 존재하는지를 평가하는 지표 EF(Enrichment Factor)도 다른 모델들에 비해 뛰어남을 알 수 있다. 

<p align="center">
  <img src="{{'/assets/img/HelixFold-M/HelixFold-M-fig4.png' | relative_url}}" width="90%">
</p>


## High-Precision Structures of HelixFold-Multimer for Superior Interaction Predcition
저자들은 HelixFold-Multimer가 예측한 구조로 FoldX(구조를 통해 자유에너지 변화를 계산)를 통해 계산한 자유에너지 변화와 실제 실험적인 친화도 Kb와의 상관관계를 평가했다. 그 결과 AlphaFold-Multimer보다 뛰어난 성능을 보여준다. 또한 ESM-IF(구조와 서열을 통해 항체 친화도 등을 예측)의 결과 또한 AlphaFold3와 유사한 수준을 보여준다. 


## Integrated Enhancement of HelixFold-Multimer and Energy-Based Method
이 절에서는 HelixFold-Multimer의 ipTM와 FoldX의 자유에너지 변화 계산의 통합이 강력한 스크리닝 방법으로 사용될 수 있는 가능성을 보여준다. 먼저 ipTM으로 결합 가능성이 있는 타겟들을 선별하고, 이 필터링된 타겟들의 false positive를 FoldX의 자유에너지 계산을 통해 한 번 더 걸러냄으로써 고품질의 항원-항체 복합체를 얻을 수 있다. 실제로 HelixFold-Multimer의 ipTM과 FoldX의 자유에너지는 음의 상관관계를 가지므로 두 점수가 서로 다른 정보를 나타내고 있음을 알 수 있다. 


<p align="center">
  <img src="{{'/assets/img/HelixFold-M/HelixFold-M-fig5.png' | relative_url}}" width="90%">
</p>


# Antibody Optimization and Design
HelixFold-Multimer의 정밀한 구조 예측이 새로운 항체 서열을 설계하거나, 기존 항체를 더 좋은 결합 친화도로 최적화할 수 있을지에 대한 분석을 진행했다.


## High-Precision Structures for Better Antibody Design
저자들은 HelixFold-Multimer의 항체 서열 설계 능력을 평가하기 위해 SARS와 PD-1이라는 항원에 결합하는 항체 구조를 예측하고, Inverse folding 모델인 ESM-IF를 통해 해당 구조를 가질법한 서열을 생성한다. 그리고 그 서열에 대한 구조 예측을 다시 해 평가함으로써 모델들의 서열 설계 능력을 평가한다. 그 결과 ipTM과 자유에너지의 분포가 crystallographic 구조들과 유사함을 알 수 있었다. 

<p align="center">
  <img src="{{'/assets/img/HelixFold-M/HelixFold-M-fig6.png' | relative_url}}" width="90%">
</p>


## Masked MSA Prediction Module for Design
또한 AlphaFold2의 구조를 사용한 HelixFold-Multimer는 Masked MSA 모듈을 가지고 있다. 이는 학습단계에서 MSA를 마스킹 한 뒤 해당 자리에 들어갈 아미노산의 확률을 예측하는 테스크로 학습이 되었기 때문이다. 이 모듈을 활용해서 항원-항체 복합체의 항체 서열 중 CDR 부분을 마스킹하고 예측해 서열을 생성한 뒤 구조 예측을 통해 ipTM과 자유에너지 변화를 계산한다. 그 결과 다른 서열 설계 모델들보다 뛰어난 성능을 보여주고, wild type에 해당하는 서열보다도 좋은 결과를 보여주는 서열들도 존재했다.

<p align="center">
  <img src="{{'/assets/img/HelixFold-M/HelixFold-M-fig7.png' | relative_url}}" width="90%">
</p>


# Method
## Datasets
먼저 pretraining 단계에서는 AlphaFold-Multimer처럼 2021-09-30 이전에 PDB에 공개된 단백질 구조들과 self-distillation 데이터셋을 구성해 학습을 진행했다. 그리고 fine-tuning 시에는 SabDab(항원-항체 구조 데이터베이스)의 2023-01-25 이전의 구조들을 사용해 학습했고, 항체의 heavy chain과 light chain의 가변영역만 사용했다. 클러스터링을 통해 유사한 구조를 지속적으로 학습할 시 발생하는 과적합 문제를 방지했으며 항체의 잔기와의 거리가 $12 \AA$ 이내인 항원의 잔기는 epitope으로 지정해 epitope 정보를 주입할 때 사용했다. 

