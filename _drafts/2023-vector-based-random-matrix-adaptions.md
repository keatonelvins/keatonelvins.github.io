---
layout: post
title: the aperiodic monotile
image: true
---

With all the context out of the way, we can finally get to VeRA: Vector-Based Random Matrix Adaptions [(Kopiczko et al., 2023)](https://arxiv.org/pdf/2310.11454.pdf). Instead of training low-rank matrices $$A$$ and $$B$$, we instead train _vectors_ $$d$$ and $$b$$ and use these vectors to scale froze, random matrices $$A$$ and $$B$$. Formally, we can express this as:

$$W = W_0 + \Delta W = W_0 + \Lambda_b B \Lambda_d A $$

with $$\Lambda_b = \text{diag}(b) \in \mathbb{R}^{r \times r}$$ and $$\Lambda_d = \text{diag}(d) \in \mathbb{R}^{k \times k}$$. This gives a $$10-1000$$x reduction in parameter count from LoRA while claiming similar performance.

![VeRA Visualized](/assets/posts/vera-visualized.png){:class="img-responsive"}
*Figure 3: VeRA Reparameterization*

Let's unpack this. The matrices $$B$$ and $$A$$ are _shared_ across layers, so their memory overhead is fixed and doesn't scale with depth. And our trainable parameter count has dropped from $$(d+k) \times r$$ to just $$k+r$$.

It's unclear if we can assume that $$B$$ and $$A$$ are orthogonal to each other. This would be an 
