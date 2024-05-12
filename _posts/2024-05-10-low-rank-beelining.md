---
layout: post
title: "beelining in low-rank structures"
image: true
---

In the regime of over-parameterized pretrained models, task-specific adaptation proves tricky at scale. The classical full fine-tuning approach of optimizing the entire model and storing the complete set of weights for each task is just prohibitive given the computational and memory requirements. And for students/hobbyists, consumer hardware can't keep up with the trend of "throw more data at a bigger transformer".

Thankfully, there is a host of techniques categorized as [parameter-efficient fine-tuning](https://github.com/huggingface/peft) methods, or PEFT, that are working to democratize the custom model. After considerable work in prompt tuning and adapter modules, a breakthrough of sorts came in the form of LoRA: Low-Rank Adaptation of Large Language Models [(Hu et al., 2022)](https://arxiv.org/pdf/2106.09685.pdf)

For context, previous findings ([Li et al, 2018](https://arxiv.org/abs/1804.08838)) ([Aghajanyan et al., 2020](https://arxiv.org/pdf/2012.13255.pdf)) have illustrated the low **intrinsic dimension** that typical over-parameterized pretrained weight matrices tend to reside on. In other words, large pretrained models empirically have some lower-dimensional subspace that captures enough of the objective landscape to enable optimization, so it's possible to restrict the dimensionality of our parameter space during fine-tuning without degrading performance.

By analogy, imagine navigating down a hyperdimensional mountain range by always heading for the steepest slope. You might worry you'll end up stuck in a hole where every direction is up, but soon you'll find every valley is actually a saddle point, and the *real* danger is running out of steam in your meandering descent through them. It might be helpful to use a map! Using an $$r$$-D projection of the landscape (where $$r$$ is small) could help prune your options and identify a more direct route down. But if $$r$$ is too low, you're cooked (imagine how helpful a 1-D hiking map would be).



This leads us to LoRA, where we approximate the **incremental update** $$\Delta W$$, i.e. the accumulated gradient steps to the pretrained model from fine-tuning, using a low-rank matrix decomposition. Explicitly, for a pretrained weight matrix $$W_0 \in \mathbb{R}^{d \times k}$$, we can reparameterize as:

$$W = W_0 + \Delta W = W_0 + BA$$

with $$B \in \mathbb{R}^{d \times r}$$, $$A \in \mathbb{R}^{r \times k}$$, and $$r << min(d, k)$$. By leaning on the assumption that $$W_0$$ has a low intrinsic dimension (meaning the update $$\Delta W$$ should have a low "intrinsic rank"), we *enforce* the bottleneck dimension of $$r$$ on $$\Delta W$$ and only optimize the parameters of $$BA$$.

![LoRA Visualized](/assets/posts/lora-visualized.png){:class="img-responsive"}
*Figure 1: LoRA Reparameterization*

The benefits of this approach are threefold. First, we've gone from $$d \times k$$ trainable parameters per module to $$(d + k) \times r$$. Depending on the scale of our model and our selection for $$r$$ (e.g., $$r = 8$$ for $$d = k = 1024$$), this can be as dramatic as a $$1000$$x reduction in parameter count! Second, as opposed to previous adapter methods, this approach incurs no additional inference cost. At deployment, we can compute and store $$W = W_0 + BA$$, use $$W$$ as usual, then efficiently update $$W$$ as $$W' = W - BA + B'A'$$ to switch to a new downstream task.

And third, LoRA often *outperforms* full fine-tuning! Without the 

by analogy. We can't always expect a low-rank matrix to capture the objective landscape. For example, imagine we pretrain with language data and fine-tune on a vision task; full fine-tuning would crush LoRA. Intrinsic dimension, and is directly related to task difficulty.

pretrained weight matrices present as big scary full-rank meatheads but are really rank-deficient losers

lora ablations and why they're important