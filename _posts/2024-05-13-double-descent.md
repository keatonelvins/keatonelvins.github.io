---
layout: post
title: "six flags: double descent"
image: true
image_name: "jetpack"
tag: "deep learning"
---

In the realm of statistics and machine learning, we have developed a deep, classical understanding of the *bias-variance tradeoff*. But its application in deep learning is slightly more alchemic (as we'll see later), and new paradigms have brought new theories. Let's start with what we understand well.

If we imagine we've run through the archetypal ML training cookbook and produced some predictor $$f_{\hat{\theta}}$$ using data $$X, Y$$, we will want to evaluate its performance. At test time, we look at the error, $$Y - f_{\hat{\theta}}(x)$$, and consider how well we are doing. There are three main sources of error:

1. **Irreducible error**: This is due to noise/randomness in $$Y\vert X$$ itself that is impossible for our model to account for. Even if the true response we are trying to learn is a deterministic function of $$X$$, randomness can still sneak into the measurement. A model that perfectly predicts the underlying signal could still disagree with the noisy data $$Y$$. In the classic setup of $$y = f_{true}(x) + \epsilon_{noise}$$, the $$\epsilon_{noise}$$ term contributes to the irreducible error.
2. **Approximation error**: This error comes from limited expressive power of $$f_\theta$$ as a finitely-parameterized model. In other words, our model isn't "flexible" enough to capture the true signal. As a toy example, imagine trying to fit the function $$y = \cos{(x)}$$ using a single 6th degree polynomial model, $$f_\theta(x) = \sum_{i=0}^6 a_ix^i$$.
3. **Estimation error**:
    1. *Bias*: Bias captures the systematic error of our learning algorithm and training data w.r.t. making predictions. We write this mathematically by $$\mathbb{E}_{\epsilon, \mathcal{D}}[f_{\hat{\theta}(\mathcal{D})}(X) - Y \ \vert\ X]$$. Note that $$\epsilon$$ respresents the randomness in $$Y\vert X$$ and $$\mathcal{D}$$ represents the randomness in the training process used to get $$\hat{\theta}$$. For example, this could include the training data set we used or the splits made in random forest trees. Traditionally, $$X$$ is separate from $$\mathcal{D}$$ and is not random. It's common to look at the squared bias to prevent positive and negative biases for different observations from canceling.
    2. *Variance*: This is the variable part of error. We write it as $$\mathbb{E}_{\mathcal{D}}[(f_{\hat{\theta}(\mathcal{D})}(X) - \mathbb{E}_{\mathcal{D}}[f_{\hat{\theta}(\mathcal{D})}(X) ])^2\ \vert \ X]$$, where $$\mathcal{D}$$ again represents the randomness of the training process. It describes how much our prediction "shakes" as a function of the randomness in the training.

This perspective can be useful and many papers utilize the bias-variance decomposition in their derivations. However, it doesn't always match what our intuition might be, especially in deep learning settings. The following figure is intended to help us understand this point (this drawing is for high-level intuition, so we musn't allow ourselves to become confused over the exact dimensions and projections).

On the left, the line passing through the origin represents the subspace of all models that we are considering. Any estimator $$f_\theta$$ that we learn will fall somewhere on this line. As we can see, the true model is not in this subspace, so we will suffer a certain amount of approximation error in our final result.

For the graph on the right, we project the true model onto the subspace of models that we are considering. The result is the best possible approximation and is the goal of our learning algorithm. We see in the figure that our predicted model is not quite equal to the best approximation. In this perspective, instead of thinking of things in terms of bias and variance, we consider how much of the "true" and "false" directions we are incorporating into our prediction.

We make this a bit more formal by discussing *survival* and *contamination*. Survival reflects, in expectation, how much of the true pattern survives the estimation/learning process. Contamination reflects how much useless information is "learned", e.g. spurious features that our model is capable of picking up but don't help with the prediction task. Survival and contamination can be thought of as the intuition behind bias and variance, respectively.

References:
- https://mlu-explain.github.io/double-descent/
- https://arxiv.org/pdf/2402.15175
