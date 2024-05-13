---
layout: post
title: "how to ada your lora: an in-depth guide"
image: true
---

AdaLoRA ([Zhang et al, 2023](https://arxiv.org/pdf/2303.10512.pdf)), a technique that drops the LoRA decomposition in favor of an SVD-flavored adaptation and iteratively prunes singular values based on their "importance". The new reparameterization becomes:

$$W = W_0 + \Delta W = W_0 + P \Lambda Q$$

where $$P \in \mathbb{R}^{d \times r}$$, $$Q \in \mathbb{R}^{r \times k}$$, $$\Lambda = \text{diag}({\lambda_i}, \ 1 \leq i \leq r)$$, and again $$r << min(d, k)$$. It's important to note that we aren't actually calculating the SVD of the updates; rather, we *enforce* the orthogonality of $$P$$ and $$Q$$ (using the following regularization term) and trust our SVD intuition applies to the tuned $$\lambda_i $$ values. The regularization is simply:

$$R(P,Q) = \|P^\top P - I\|_F + \|QQ^\top - I\|_F$$

We can update $$P$$ and $$Q$$ in the standard gradient descent fasion, but pruning $$\Lambda$$ takes some additional steps. We first calculate $$S^{(t)}$$, the "importance" score of all triplets (i.e. all $$\lambda_i$$ values and their singular vector equivalents), then update $$\Lambda$$ as:

$$\Lambda^{(t+1)}_k = \mathcal{T}(\tilde{\Lambda}^{(t)}_k, S^{(t)}_k), \text{ with } \mathcal{T}(\tilde{\Lambda}^{(t)}_k, S^{(t)}_k)_{ii} = \begin{cases}\tilde{\Lambda}^{(t)}_{k, ii} & S^{(t)}_{k, ii} \text{ is in the top-}b(t)\text{ of } S^{(t)}, \\ 0 & o.w. \end{cases}$$

noting that $$b(t)$$ denotes our parameter budget at timestep $$t$$ (can also think of $$b(t)$$ as the "total rank" of our incremental matrices). This is a lot of fun notation to say each timestep we mask out the values of the *post-gradient step* $$\tilde{\Lambda}^{(t)}$$ at layer $$k$$ that fall outside our shrinking budget, ranked by their sensitivity to the training loss with some smoothing.

1. The modules of pretrained models tend to have a low intrinsic dimension, meaning we can still reach the solution manifold while navigating a lower-dimensional subspace of the objective landscape.
2. We can capitalize on this by reparameterizing the accumulated update to our pretrained weights as low-rank matrices.
3. Not all parameters are created equal. Given a fixed budget, we should maximize the effect of our incremental updates through selective application across layers and modules.

We can build some intuition for this last point by thinking about contextualized embeddings in deep nets. It's a well-established result that embedding quality increases (almost) monotonically alongside model depth. We can see this in the figure below (thanks [Jay](http://jalammar.github.io/illustrated-bert/)), which compares performance across six embedding choices:

![VeRA Visualized](/assets/posts/contextualized-embeddings.png){:class="img-responsive"}
*Figure 2: Feature-based NER Ablation Results*

We can hypothesize that earlier layers are responsible for capturing local features of the input data, while deeper layers are increasingly abstract and capture more complex or global relationships. As it propagates, the representation can gain hierarchical and nuanced features by incorporating dependencies from a broader context. However, the results show the *second-to-last* layer achieved the best single-layer performance, which could imply that the last layer is more aligned to the surrogate task and contains less general information. The AdaLoRA paper also included two experiments that showed performance boosts when fine-tuning feed-forward networks over attention modules and deep layers over early ones. Deep nets are not just soups of homogenous weights and should be treated accordingly when working within constraints.
