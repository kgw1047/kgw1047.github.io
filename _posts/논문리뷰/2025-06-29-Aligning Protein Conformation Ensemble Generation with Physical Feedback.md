---
title: "Aligning Protein Conformation Ensemble Generation with Physical Feedback"
last_modified_at: 2025-06-29
categories:
  - 논문리뷰
tags:
  - Protein Ensemble
  - Diffusion Model
  - ICML
# excerpt: "DF-GAN 논문 리뷰 (CVPR 2022 Oral)"
use_math: true
classes: wide
---

> ICML 2025. [[Paper](https://openreview.net/forum?id=Asr955jcuZ&noteId=L9xGstsHvD)]  
> Jiarui Lu, Xiaoyin Chen, Stephen Zhewen Lu, Aurélie Lozano, Vijil Chenthamarakshan, Payel Das, Jian Tang  
> 30 May 2025 


# Keypoints
## Diffusion-based Generative Model + Energy-based Alignment
- **Boltzmann Factor**: 분배함수 계산을 피하기 위해 두 구조의 에너지 차이를 활용
- **Finite-state Approximation**: 전체 분배함수 계산이 불가능하므로 미니배치 단위로 분배함수 계산
- **Loss Function**:  
  
  
# Limitations
1. AlphaFold3 구조의 모델을 사용했기 때문에 long time generation에 적합하지 않다.
2. Energy, 즉 forcefield의 정확도 문제.
3. Monomer만 가능.
4. 다른 generative model의 가능성(flow matching 같은)
  
# References
## Protein Conformation Generations
- [Boltzmann generator](https://arxiv.org/abs/1812.01729)
- [EigenFold](https://arxiv.org/abs/2304.02198)
- [Str2Str](http://arxiv.org/abs/2306.03117)
- [DiG](https://arxiv.org/abs/2405.18428)
- [ConfDiff](https://arxiv.org/abs/2403.14088)
- [AlphaFlow](http://arxiv.org/abs/2402.04845)
- [ESMDiff](https://arxiv.org/abs/2410.18403)
- [MDGen](https://arxiv.org/abs/2409.17808)

## Alignment methods for generative models
- [Diffusion-DPO](https://arxiv.org/abs/2311.12908)
- [ABDPO](https://arxiv.org/html/2403.16576v1)
- [ALIDIFF](https://arxiv.org/abs/2407.01648)
- [DECOMPDPO](https://arxiv.org/abs/2407.13981)
