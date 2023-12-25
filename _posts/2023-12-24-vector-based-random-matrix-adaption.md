---
layout: post
title: vector-based random matrix adaptions
image: true
---
In the regime of massively overparameterized language models pretrained on billions of token, task-specific fine-tuning becomes a logistical nightmare. Naively optimizing the entire model and storing the full set of weights for each adaption has a significant computational and memory overhead, especially as the number of tasks continues to scale.

A breakthrough of sorts came in the form of LoRA: Low-Rank Adaption of Large Language Models [(Hu et al., 2022)](https://arxiv.org/pdf/2106.09685.pdf), which made use of findings ([Li et al, 2018](https://arxiv.org/abs/1804.08838)) ([Aghajanyan et al., 2020](https://arxiv.org/pdf/2012.13255.pdf)) illustrating the low intrinsic dimension these overparameterized weight matrices tended to operate in. In other words, large pretrained models empirically have some lower-dimensional subspace that captures enough of the objective landscape to enable optimization, so we can restrict the dimensionality of our parameter space during finetuning (prior work has demonstrated the effectiveness of randomized projections) with minimal loss in performance.

In LoRA, we approximate the accumulated gradient update during finetuning, denoted $$\Delta W$$, by using a low-rank matrix decomposition. Explicitly, for a pretrained weight matrix $$W_0 \in \mathbb{R}^{d \times k}$$, we can refactor the forward pass $$h = Wx$$ as:

$$h =(W_0 + \Delta W) x = W_0 x + \Delta W x = W_0 x + BA x$$

with $$B \in \mathbb{R}^{d \times r}$$, $$A \in \mathbb{R}^{r \times k}$$, and $$r << min(d, k)$$. By leaning on the assumption that $$W_0$$ has a low intrinsic-dimension (meaning the update $$\Delta W$$ should have a low "intrinsic rank"), we _enforce_ the bottleneck dimension of $$r$$ on $$\Delta W$$ and only optimize the parameters of $$BA$$.

![LoRA Visualized](/assets/posts/lora-visualized.png){:class="img-responsive"}
*Figure 1: LoRA Reparameterization*

The benefits of this approach are twofold. First, we've gone from $$d \times k$$ trainable parameters per component to $$(d + k) \times r$$. Depending on the scale of our model and our selection for $$r$$, this can be as dramatic as a $$1000$$x reduction in parameter count! Second, as opposed to previous adapter methods, this approach incurs no additional inference cost. At deployment, we can compute and store $$W = W_0 + BA$$, use $$W$$ as usual, then update $$W$$ as $$W' = W - BA + B'A'$$ to switch to a new task.

In further work, researchers dynamically distributed parameters ([Zhang et al, 2023](https://arxiv.org/pdf/2303.10512.pdf)) to reach lower and lower memory footprints.

This leads us to VeRA: Vector-Based Random Matrix Adaptions [(Kopiczko et al., 2023)](https://arxiv.org/pdf/2310.11454.pdf). While not necessarily establishing the lower cap, VeRA reduces the parameter count from LoRA by 10-1000x while _claiming_ equal to better performance.

The insight of VeRA comes from the combination of principles established in LoRA and other papers noting the efficiency of random projections in deep nets. Instead of training low-rank matrices $$A$$ and $$B$$, we instead train _vectors_ $$\Lambda_d$$ and $$\Lambda_b$$ and use these vectors to scale froze, random matrices $$A$$ and $$B$$. Formally, we can express this as:

$$h = W_0x + \Delta Wx = W_0x + \Lambda_b B \Lambda_d A x$$

with $$\Lambda_b \in \mathbb{R}^{b \times 1}, \Lambda_d \in \mathbb{R}^{d \times 1}$$.

![VeRA Visualized](/assets/posts/vera-visualized.png){:class="img-responsive"}
*Figure 2: VeRA Reparameterization*
