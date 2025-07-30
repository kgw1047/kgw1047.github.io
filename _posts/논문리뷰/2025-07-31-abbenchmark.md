---
title: "Evaluating Deep Learning Based Structure Prediction Methods on Antibody-Antigen Complexes"
last_modified_at: 2025-07-31
categories:
  - 논문리뷰
tags:
  - Structural Bioinformatics
  - Antibody
  - Benchmark
  - arXiv
use_math: true
classes: wide
---

> Bioinformatics 2025. [[Paper](https://www.biorxiv.org/content/10.1101/2025.07.11.662141v1)]  [[Github](https://github.com/samuelfromm/abag-benchmark-set)] [[Zenodo](https://zenodo.org/records/15764903)]  
> Samuel Fromm, Marko Ludaic, Arne Elofsson  
> 11 July 2025  


# Introduction
항원-항체 상호작용은 면역 반응과 항체 치료제 개발에 핵심적이다. 치료용 항체 시장이 급성장하면서 항원-항체 복합체 구조를 정확히 예측할 수 있을 경우 후보 물질 선별 및 최적화 속도를 크게 높이고 표적 치료 설계에 도움을 준다. AlphaFold2 및 AlphaFold-Multimer는 단백질 복합체 예측 정확도를 크게 향상시켰지만 co-evolutionary 신호가 부족한 경우, 대표적으로 항원-항체 복합체 예측에서는 정확도가 매우 떨어진다.  
이에 이를 개선하기 위한 두 가지 방법이 등장했는데, 먼저 Massive Sampling이다. CASP15에서 확인된 방법으로 수백~수천 개의 구조를 예측해서 올바른 구조가 나올 확률을 높이는 것이다. Drop out, MSA masking, MSA subsampling 등의 변형된 기법 또한 제안되었다.  
다음은 Pairformer의 도입이다. AlphaFold3에서는 기존 Evoformer를 대체한 Pairformer를 사용해 MSA에 의존하지 않고 상호작용을 추론할 수 있도록 설계해 co-evolutionary 신호에 대한 의존성을 낮추었다.  
또한 예측 구조의 신뢰도를 평가하는 지표들도 꾸준히 발전해왔는데, pDockQ 이후에 AlphaFold-Multimer의 ipTM과 ranking confidence(0.8 * ipTM + 0.2 * pTM)가 도입되었다. 이후에 pDockQ2, actifpTM, ipSAE 등의 개선된 지표가 제안되었으나 항원-항체 복합체에서 예측 구조의 순위 결정 능력을 크게 끌어올리진 못했다.  
이에 저자들은 foundation model들의 학습데이터에 없는 항원-항체 벤치마크 데이터셋을 구축하고, 이들을 평가했다. 또한 신뢰도 지표들이 최적의 구조를 선별하는데 얼마나 효과적인지 조사했다.  

# Results and Discussion
저자들은 foundation model로 AlphaFold2.3, AlphaFold3, Boltz-1, Chai-1을 선정했다. 벤치마크 데이터셋으로는 학습데이터에 사용되지 않고 서로 유사한 구조를 제외한 110개의 항원-항체 복합체 구조들을 사용했고, 모델 당 200개의 구조를 생성하고 DockQ를 측정했다. 

## AlphaFold3 makes better predictions than the other methods.

<p align="center">
  <img src="{{'/assets/img/abbenchmark/abbenchmark-fig1.png' | relative_url}}" width="90%">
</p>

Fig A를 보면 AlphaFold3가 다른 모델들을 큰 폭으로 앞서는 모습을 볼 수 있다. Chai-1과 Boltz-1의 생각보다 저조한 성능에 웹서버를 이용해서 다시 평가해보았지만 눈에 띄는 성능 향상은 없었다. 또한 AlphaFold3와 Chai-1, Boltz-1은 샘플 갯수가 200개까지 올라갈수록 DockQ의 선형적인 증가를 보였다(샘플 갯수를 로그 스케일로 나타냈을 때).
AlphaFold3는 기본적으로 하나의 초기 시드에 대해 구조 예측을 하면 구조 5개를 생성해주는데, 이렇게 생성된 구조들은 Diffusion 단계에서의 노이즈의 차이로 인해 약간의 차이가 발생한다. Fig C에서는 이렇게 Diffusion 단계에서의 노이즈 차이로 인한 구조적 다양성보다 초기 시드를 다르게 한 경우에 항원-항체 구조의 결합 다양성이 높아진다는 것을 보여준다. 


## AF2.3-ColMask is the best strategy to increase sampling.
CASP15에서 AlphaFold2.3으로 다양한 구조를 샘플링하기 위한 여러 가지 방법들이 소개되었다. MSA의 column의 15%를 masking하거나, 모델에 drop out을 적용하거나(AFsample), MSA subsampling이 그 방법들이다. 해당 방법들은 하나의 샘플만 평가했을 때에는 기본적인 AlphaFold2.3의 구조 예측보다 성능이 떨어졌지만 샘플 갯수가 증가함에따라 더 나은 성능을 보였으며, AF2.3-ColMask로 표기된 MSA의 column을 masking하는 방식이 가장 성능 향상폭이 컸다.  
*[개인적인 의견] 아마 이 부분은 MSA의 column을 masking 했을 때, co-evolutionary 신호가 가려져 모델이 대안적 결합 구조를 탐색하는 것이 아닐까라는 추측을 해볼 수 있다.*


## A large gap between the best and top-ranked models.
Fig B를 보면 Fig A와 비교했을 때, 샘플 갯수 증가에 따른 DockQ 점수의 향상이 더딘 것을 볼 수 있다. 그러니까 가장 높은 DockQ의 구조를 골랐을때보다 모델 자체의 신뢰도 점수로만 베스트 구조를 선정했을 때 그 효과가 현저히 떨어진다는 것이다. 실제로 AlphaFold3의 경우, DockQ 점수를 0.29에서 0.52까지 끌어올릴 수 있는 반면, 0.37까지밖에 끌어올리지 못했다. 이를 통해 모델의 신뢰도 지표가 여전히 개선이 필요함을 알 수 있다.  
Boltz-1과 Chai-1의 경우는 문제가 더 심각해져 샘플 갯수를 늘려도 성능 향상을 거의 기대할 수 없는 것을 볼 수 있다.

<p align="center">
  <img src="{{'/assets/img/abbenchmark/abbenchmark-fig2.png' | relative_url}}" width="90%">
</p>

위 박스플롯은 110개의 타겟을 평균 DockQ 순서대로(높음 -> 낮음) 배열한 것이고 검은 점은 top-ranked(신뢰도 지표가 가장 높은) 구조이다. 박스플롯의 넓이가 좁고 위쪽에 분포해있다면 **쉬운 타겟**, 넓게 분포해있다면 **중간**, 좁고 아래쪽에 위치해 있다면 **어려운 타겟**인데, AlphaFold3가 다른 모델들에 비해 **쉬운 타겟**이 많은 것을 볼 수 있다. 또한 **어려운 타겟**의 경우에도 top-ranked 구조를 뽑았을 때, DockQ가 높은 구조가 뽑히는 경우를 종종 볼 수 있었다.  

<p align="center">
  <img src="{{'/assets/img/abbenchmark/abbenchmark-fig3.png' | relative_url}}" width="90%">
</p>

또한 위 산점도를 보면 어떤 타겟들에 대해서는 AlphaFold2.3이 더 나은 성능을 보인다. 만약 단순 난이도로 모델의 성능을 평가할 수 없음을 나타내는데, AlphaFold3가 **특정 특징**을 가진 타겟들에 대해서 뛰어난 성능을 보인다는 사실을 시사한다.  

## Has AlphaFold3 learned to predict structure beyond its training set?
저자들은 벤치마크 데이터셋에 있는 타겟들과 2021-09-30 이전의 구조들과의 유사도 측정을 통해 모델들의 성능이 학습데이터와 어떠한 연관성이 있는지를 평가했다.  
항원-항체 복합체의 경우 co-evolutionary 신호가 없기 때문에 AlphaFold3가 물리적 법칙을 학습한 것인지를 알아내고자 벤치마크를 진행했다. 

<p align="center">
  <img src="{{'/assets/img/abbenchmark/abbenchmark-fig4.png' | relative_url}}" width="90%">
</p>

위 박스플롯을 보면 학습데이터와의 TM-score가 상승함에 따라 DockQ도 같이 상승하는 모습을 볼 수가 있다. 반면 유사도가 낮을 때에는 그 성능이 급격히 떨어지는 경향 또한 볼 수가 있는데, 이는 학습데이터의 구조에 의존성을 가진다는 점을 보여준다. 학습데이터와의 유사도와 DockQ의 상관관계는 complex search는 0.32, interface는 0.17로 없진 않지만 그렇다고 절대적으로 의존하는 것은 아니라는 사실을 알려준다. Boltz-1의 경우 학습데이터와의 유사도가 높아져도 성능 향상을 거의 기대할 수 없었다.  

## Ranking of AlphaFold3 models
첫 번째 Fig에서 여러 샘플들의 랭킹을 통해 성능 향상을 기대할 수 있었다. DockQ로 랭킹을 하는 경우 0.29에서 0.52까지, 모델 자체의 신뢰도 지표의 경우 0.37까지의 향상이 있었는데 이는 무려 40% 정도의 성능 차이이다. 

<p align="center">
  <img src="{{'/assets/img/abbenchmark/abbenchmark-fig5.png' | relative_url}}" width="50%">
</p>

위 그래프에서 **8jg5** 타겟을 보면 AlphaFold3의 신뢰도 지표 상으론 샘플 간 퀄리티 차이가 거의 없지만 DockQ 점수는 매우 다양하다. 이는 AlphaFold3의 신뢰도 지표의 취약점을 보여주는 예이다.  

<p align="center">
  <img src="{{'/assets/img/abbenchmark/abbenchmark-fig6.png' | relative_url}}" width="50%">
</p>

위 그래프에서 **7wvm** 타겟을 보면 AlphaFold3의 신뢰도 지표 상으로는 샘플 간 유의미한 차이가 있지만 실제 DockQ 점수는 거의 차이가 없음을 확인할 수 있다.  

## Ranking analysis
Best vs top-ranking에서 현저한 성능 차이를 보이기 때문에, 이 현상에 대한 분석 또한 진행했다. 결론부터 말하자면 여러 신뢰도 지표가 존재하지만 결국 모델이 예측한 PAE 및 pLDDT에 의존하고, PAE의 부정확성이 좋은 구조를 선정하는데에 있어서 병목이라고 한다. 
PAE가 문제인지, 신뢰도 지표의 수식이 문제인지를 테스트하기 위해 저자들은 실제 AE(예측 구조와 정답 구조를 통해 계산한)를 사용해 분석을 진행했다. 

<p align="center">
  <img src="{{'/assets/img/abbenchmark/abbenchmark-fig7.png' | relative_url}}" width="90%">
</p>

위 그림에서는 DockQ가 0.690에서 0.040까지 떨어지는 예측 결과에도 불구하고 PAE 및 ipTM은 거의 변하지 않는 예시를 보여준다. 

<p align="center">
  <img src="{{'/assets/img/abbenchmark/abbenchmark-fig8.png' | relative_url}}" width="90%">
</p>

위 그림은 실제 AE를 사용해서 계산한 aeiTM을 비교한 결과이다. AE 자체의 차이도 크지만 aeiTM 또한 유의미한 차이가 있는 것을 보아, PAE의 문제라는 판단을 내려볼 수 있다.  

<p align="center">
  <img src="{{'/assets/img/abbenchmark/abbenchmark-fig9.png' | relative_url}}" width="90%">
</p>

Fig E는 실제 계산된 AE를 사용한 신뢰도 지표들이 샘플 갯수가 늘어남에 따라 더 나은 예측 구조를 기대할 수 있음을 보여준다. Fig F도 앞서 AlphaFold3의 신뢰도 지표를 사용했을 때보다 훨씬 DockQ와의 correlation이 높은 것을 알 수 있다.  


