---
layout: post
title: vector-based random matrix adaptions
image: true
---
In the regime of massively overparameterized language models pretrained on trillions of token, task-specific fine-tuning becomes a logistical nightmare. Naively optimizing the entire model and storing the full set of weights for each adaption has a significant computational and memory overhead, especially as the number of tasks continues to scale.

A breakthrough of sorts came in the form of LoRA: Low-Rank Adaption of Large Language Models [(Hu et al., 2022)](https://arxiv.org/pdf/2106.09685.pdf). LoRA made use of findings that illustrated the low intrinsic dimension these models' weight matrices tended to operate on. In LoRA, we approximate the gradient updates by using a low-rank matrix decomposition, which dramatically reduces the trainable parameter count while simultaneously decoupling the adaption from the backbone. Explicitly, for a pretrained weight matrix $$W \in \mathbb{R}^{m \times n}$$, we can refactor the classical update step as:

$$W_{t+1} = W_tx + \Delta W_tx \to W_{t+1} = W_tx + BAx$$

where $$A \in \mathbb{R}^{m \times r}$$, $$B \in \mathbb{R}^{r \times n}$$, and $$r << min(m, n)$$. This effectively restricts the feature space of our updates to the bottleneck dimension $$r$$. And at inference time, we don't even need to merge the tuned $$B$$ and $$A$$ back into the pretrained model; we can instead pass the embedding for each layer to $$W$$ and $$BA$$ separately and sum their outputs. This reduces the need for $$k$$ full sets of weights for $$k$$ tasks to just one full set (the pretrained backbone) and $$k$$ sets of low-rank matrices.

In further work, researchs quantized models ([Dettmers et al., 2023](https://arxiv.org/pdf/2305.14314.pdf)) and dynamically distributed parameters ([Zhang et al, 2023](https://arxiv.org/pdf/2303.10512.pdf)) and how low can we go? What is the minimum number of trainable parameters needed to adapt a pretrained model to a given task at a satisfactory level without incurring additional inference cost?

This leads us to VeRA: Vector-Based Random Matrix Adaptions [(Kopiczko et al., 2023)](https://arxiv.org/pdf/2310.11454.pdf). While not necessarily establishing the lower cap, VeRA reduces the parameter count from LoRA by 10-1000x while claiming equal to better performance.

The insight of VeRA comes from the combination of principles established in LoRA and other papers noting the efficiency of random projections in deep nets. Instead of training low-rank matrices $$A$$ and $$B$$, we instead train _vectors_ $$\Lambda_d$$ and $$\Lambda_b$$ and use these vectors to scale froze, random matrices $$A$$ and $$B$$. Formally, we can express this as:

$$W_{t+1} = W_tx + \Delta W_tx \to W_{t+1} = W_tx + \Lambda_b B \Lambda_d A x$$

with $$\Lambda_b \in \mathbb{R}^{b \times 1}, \Lambda_d \in \mathbb{R}^{d \times 1}$$