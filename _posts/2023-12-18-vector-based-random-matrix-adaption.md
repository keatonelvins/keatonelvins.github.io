---
layout: post
title: vector-based random matrix adaptions
image: true
---

In this post we will be going over VeRA: Vector-Based Random Matrix Adaptions [(Kopiczko et al., 2023)](https://arxiv.org/pdf/2310.11454.pdf) and building up a basic implementation of the technique in PyTorch. VeRA is an extension on the ideas established in LoRA: Low-Rank Adaption of Large Language Models [(Hu et al., 2021)](https://arxiv.org/pdf/2310.11454.pdf) and proposes several optimizations to significantly cut down on the number of trainable parameters needed during fine-tuning.
