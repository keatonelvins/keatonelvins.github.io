---
layout: post
title: vector-based random matrix adaptions
image: true
---
In the regime of massively overparameterized language models pretrained on billions of token, task-specific fine-tuning becomes a logistical nightmare. Naively optimizing the entire model and storing the full set of weights for each adaption has a significant computational and memory overhead, especially as the number of tasks continues to scale.

A breakthrough of sorts came in the form of LoRA: Low-Rank Adaption of Large Language Models [(Hu et al., 2022)](https://arxiv.org/pdf/2106.09685.pdf), which made use of findings ([Li et al, 2018](https://arxiv.org/abs/1804.08838)) ([Aghajanyan et al., 2020](https://arxiv.org/pdf/2012.13255.pdf)) illustrating the low intrinsic dimension these overparameterized weight matrices tended to operate in. In other words, large pretrained models empirically have some lower-dimensional subspace that captures enough of the objective landscape to enable optimization, so we can restrict the dimensionality of our parameter space during fine-tuning (prior work has demonstrated the effectiveness of randomized projections) with minimal loss in performance.

In LoRA, we approximate the accumulated gradient update during fine-tuning, denoted $$\Delta W$$, by using a low-rank matrix decomposition. Explicitly, for a pretrained weight matrix $$W_0 \in \mathbb{R}^{d \times k}$$, we can refactor as:

$$W = W_0 + \Delta W = W_0 + BA$$

with $$B \in \mathbb{R}^{d \times r}$$, $$A \in \mathbb{R}^{r \times k}$$, and $$r << min(d, k)$$. By leaning on the assumption that $$W_0$$ has a low intrinsic-dimension (meaning the update $$\Delta W$$ should have a low "intrinsic rank"), we _enforce_ the bottleneck dimension of $$r$$ on $$\Delta W$$ and only optimize the parameters of $$BA$$.

![LoRA Visualized](/assets/posts/lora-visualized.png){:class="img-responsive"}
*Figure 1: LoRA Reparameterization*

The benefits of this approach are twofold. First, we've gone from $$d \times k$$ trainable parameters per module to $$(d + k) \times r$$. Depending on the scale of our model and our selection for $$r$$ (e.g., $$r = 8$$ for $$d = k = 1024$$), this can be as dramatic as a $$1000$$x reduction in parameter count! Second, as opposed to previous adapter methods, this approach incurs no additional inference cost. At deployment, we can compute and store $$W = W_0 + BA$$, use $$W$$ as usual, then efficiently update $$W$$ as $$W' = W - BA + B'A'$$ to switch to a new task.

Further work addressed a limitation in LoRA, namely that the parameter budget was allocated uniformly across modules. Out of this came AdaLoRA ([Zhang et al, 2023](https://arxiv.org/pdf/2303.10512.pdf)), a technique that drops the LoRA decomposition in favor of an SVD-flavored adaption and iteratively prunes singular values based on their "importance". The new reparameterization becomes:

$$W = W_0 + \Delta W = W_0 + P \Lambda Q$$

where $$P \in \mathbb{R}^{d \times r}$$, $$Q \in \mathbb{R}^{r \times k}$$, $$\Lambda = diag({\lambda_i}, \ 1 \leq i \leq r)$$, and again $$r << min(d, k)$$. It's important to note that we aren't actually calculating the SVD of the updates; rather, we _enforce_ the orthogonality of $$P$$ and $$Q$$ (using the following regularization term) and trust our SVD intuition applies to the tuned $$\lambda_i $$ values. The regularization is simply:

$$R(P,Q) = \|P^\top P - I\|_F + \|QQ^\top - I\|_F$$

We can update $$P$$ and $$Q$$ in the standard gradient descent fasion, but pruning $$\Lambda$$ takes some additional steps. We first calculate $$S^{(t)}$$, the "importance" score of all triplets (i.e. all $$\lambda_i$$ values and their singular vector equivalents), then update $$\Lambda$$ as:

$$\Lambda^{(t+1)}_k = \mathcal{T}(\tilde{\Lambda}^{(t)}_k, S^{(t)}_k), \text{ with } \mathcal{T}(\tilde{\Lambda}^{(t)}_k, S^{(t)}_k)_{ii} = \begin{cases}\tilde{\Lambda}^{(t)}_{k, ii} & S^{(t)}_{k, ii} \text{ is in the top-}b(t)\text{ of } S^{(t)}, \\ 0 & o.w. \end{cases}$$

noting that $$b(t)$$ denotes our parameter budget at timestep $$t$$ (can also think of $$b(t)$$ as the "total rank" of our incremental matrices). This is a lot of fun notation to say each timestep we mask out the values of the _post-gradient step_ $$\tilde{\Lambda}^{(t)}$$ at layer $$k$$ that fall outside our shrinking budget, ranked by their sensitivity to the training loss with some smoothing. For more information about how we schedule the budget, how the sensitivity-based importance function works, and further implementation details outside the scope of this post, I highly recommend reading the original paper.

As a mid-point summary, there should be three key takeaways from this context to matrix adaption techniques:

1. The modules of pretrained models tend to have a low intrinsic dimension, meaning we can still reach the solution manifold while navigating a lower-dimensional subspace of the objective landscape.
2. We can capitalize on this by reparameterizing the accumulated update to our pretrained weights as low-rank matrices.
3. Not all parameters are created equal. Given a fixed budget, we should maximize the effect of our incremental updates through selective application across layers and modules.

We can build some intuition for this last point by thinking about contextualized embeddings in deep nets. It's a well-established result that embedding quality increases (almost) monotonically alongside model depth. We can see this in the figure below (thanks [Jay](http://jalammar.github.io/illustrated-bert/)), which compares performance across six embedding choices:

![VeRA Visualized](/assets/posts/contextualized-embeddings.png){:class="img-responsive"}
*Figure 2: Feature-based NER Ablation Results*

We can hypothesize that earlier layers are responsible for capturing local features of the input data, while deeper layers are increasingly abstract and capture more complex or global relationships. As it propagates, the representation can gain hierarchical and nuanced features by incorporating dependencies from a broader context. However, the results show the _second-to-last_ layer achieved the best single-layer performance, which could imply that the last layer is more aligned to the surrogate task and contains less general information. The AdaLoRA paper also included two experiments that showed performance boosts when fine-tuning feed-forward networks over attention modules and deep layers over early ones. Deep nets are not just soups of homogenous weights and should be treated accordingly when working within constraints.

With all the context out of the way, we can finally get to VeRA: Vector-Based Random Matrix Adaptions [(Kopiczko et al., 2023)](https://arxiv.org/pdf/2310.11454.pdf). VeRA reduces the parameter count from LoRA by 10-1000x while claiming similar performance (no data on performance vs AdaLoRA yet). Instead of training low-rank matrices $$A$$ and $$B$$, we instead train _vectors_ $$\Lambda_d$$ and $$\Lambda_b$$ and use these vectors to scale froze, random matrices $$A$$ and $$B$$. Formally, we can express this as:

$$h = W_0x + \Delta Wx = W_0x + \Lambda_b B \Lambda_d A x$$

with $$\Lambda_b \in \mathbb{R}^{b \times 1}, \Lambda_d \in \mathbb{R}^{d \times 1}$$.

![VeRA Visualized](/assets/posts/vera-visualized.png){:class="img-responsive"}
*Figure 3: VeRA Reparameterization*
