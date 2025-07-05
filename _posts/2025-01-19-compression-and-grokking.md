---
layout: post
title: "compression and grokking"
image_name: "boy"
tag: "deep learning"
---

>“He should have known better because, early in his learnings under his brother Mahmoud, he had discovered that long human words (the longer the better) were easy, unmistakable, and rarely changed their meanings . . . but short words were slippery, unpredictable, changing their meanings without any pattern. Or so he seemed to grok. Short human words were never like a short Martian word—such as “grok” which forever meant exactly the same thing. Short human words were like trying to lift water with a knife.”
> <author>Robert Heinlein, "Stranger in a Strange Land"</author>

I had to convert a folder to a .zip the other day and had a little moment. I think I often take it for granted that _lossless compression_ is a thing we can just do. The [Wikipedia page](https://en.wikipedia.org/wiki/ZIP_(file_format)#Compression_methods) for ZIP was honestly kind of bland, although I did find out that the "zip" component was originally of the "moving at high speed" flavor and not of "fasten with a zipper". Maybe it's time to replace those little zipper icons with a 🏃🏻💨.

Compression, as I first learned from Ilya Sutskever in a recorded [lecture](https://www.youtube.com/watch?time_continue=1273&v=AKMuA_TVz3A&embeds_referring_euri=https%3A%2F%2Fwww.lesswrong.com%2F&source_ve_path=Mjg2NjY) he gave at the Simons Institute, is not just relevant for sending big files through email. It seems to be very deeply connected with learning and understanding. Marcus Hutter, researcher at DeepMind and organizer of the [Hutter Prize](https://en.wikipedia.org/wiki/Hutter_Prize), believes that "text compression and AI are equivalent problems":

> "In order to compress data, one has to find regularities in them, which is intrinsically difficult... So compressors beating the current 'dumb' compressors need to be smart(er). Since the prize wants to stimulate developing 'universally' smart compressors, we need a 'universal' corpus of data. Arguably the online encyclopedia Wikipedia is a good snapshot of the Human World Knowledge. So the ultimate compressor of it should 'understand' all human knowledge, i.e. be really smart."

If the distribution/generator your data comes from is not just uniform, then we can expect some variables to occur more frequently than others. We can find correlations and identify patterns. "Dog" is probably more likely to follow the phrase "My new pet..." than "axolotl", and this _tells_ us something about humans and dogs and axolotls.

One lens we can understand language modeling through is the statistical next-token predictor. The autoregressive text generator extremely fitted to the distribution curve of natural language that spits out high-probability tokens accordingly. This idea is actually super old, going back at least to the 1950's with Shannon's _Prediction and Entropy of Printed English_. Here, Shannon references two key concepts to language modeling: _entropy_ and _redundancy_.

> "If the language is translated into binary digits (0 or 1) in the most efficient way, the entropy is the average number of binary digits required per letter of the original language. The redundancy, on the other hand, measures the amount of constraint imposed on a text in the language due to its statistical structure, e.g., in English the high frequency of the letter E, the strong tendency of H to follow T or of V to follow Q."
<cite>[^1]</cite>

The 'dumb' compressors referenced by Hutter exploit this redundancy of natural language. If characters in english were uniformly random, we would predict each letter to have an entropy of around 5 bits. Instead, due to redundancy, it's probably closer to 1-1.5 bits. This gap is what enables lossless compression, algorithms like [LZ77](https://en.wikipedia.org/wiki/LZ77_and_LZ78), to actually work! This fact also lends itself to algorithms like [Byte-Pair Encoding](https://en.wikipedia.org/wiki/Byte_pair_encoding) (used during tokenization for all mainstream LLMs) that chunk commonly co-occuring chars into larger n-grams.

By Shannon’s source coding theorem, the expected optimal coding rate for samples drawn from a distribution is equal to the entropy of the distribution. If we can decrease the apparent entropy of the data by identifying patterns in it, we can develop a better encoding! Let's apply these ideas to language models and formalize.

We can model our distribution entropy as $$H(D)$$ and our model complexity as $$C(M)$$. We know that the distribution entropy is fixed, but we can reduce $$H(D \vert M)$$, or the _data entropy under the model_. The sum of these two terms is what we want to minimize in our final trained model.

$$\min_M H (D \vert M) + C(M)$$

If our model is just a lookup table of our training data, we have achieved $$H(D \vert M) = 0$$. The apparent entropy of the data has been reduced completely. However, this approach is so naïve that our $$C(M)$$ is exactly equal to $$H(D)$$, and we haven't compressed anything!

We want a model that compresses the data best, and is itself most compressible. 

Before we dive in to this idea and how it relates to [Occam's Razor](https://en.wikipedia.org/wiki/Occam%27s_razor), [MDL](https://en.wikipedia.org/wiki/Minimum_description_length), [Kolmogorov Complexity](https://en.wikipedia.org/wiki/Kolmogorov_complexity), [Algorithmic Information Theory](https://en.wikipedia.org/wiki/Algorithmic_information_theory), and [Solmonoff Induction](https://en.wikipedia.org/wiki/Solomonoff%27s_theory_of_inductive_inference) (warning: high rabbit-hole potential), I want to introduce the deep learning phenomena known as _grokking_.

> “Grok’ means to understand so thoroughly that the observer becomes a part of the observed – to merge, blend, intermarry, lose identity in group experience."
> <author>Robert Heinlein</author>

The OpenAI team behind the og grokking paper[^2] identified that small, algorithmically generated datasets could prove to be a fertile ground for studying generalization of overparametrized neural networks beyond memorization of the training data. In practice, they showed that significantly into the overfitting regime (according to our classical understanding of deep learning), the models would suddenly jump from random guesswork to perfect generalization on validation data.

From an information theory perspective[^3], the writing had been on the wall. Results had already demonstrated that DNN training involves two distinct phases: first, the network learns to fully represent the training data; then, it learns to compress the representation of the input and forgets irrelevant details. But, by studying algorithmically generated datasets, e.g. training on division mod 97, researchers could now explore the exact recipes and circuits that cause this to happen.

The first follow-up paper I read was on "Unifying Grokking and Double Descent"[^4], where the authors viewed both phenomena as inductive biases applying gradient pressure towards better-generalizing but slower to learn patterns. This is the lens of _pattern learning_, which captures the idea that neural networks can use distinct kinds of mechanisms to classify different inputs; for example, they may memorize some examples while applying heuristics to classify others. Grokking happens when slow patterns generalize well and are ultimately preferred by the training regime, but are preceded by faster patterns which generalize poorly. Patterns can be viewed as a function of model size as well as a function of training time, and learning dynamics are similar between these two cases.

Earlier, I mentioned we can think about language models in their base autoregressive text generator state. Another mental model I like, borrowed from [Francois Chollet](https://fchollet.substack.com/p/how-i-think-about-llm-prompt-engineering), is the _continuous, interpolative_ program database. 

>"Instead of being stored as a set of discrete entries, your data is stored as a vector space — a curve. You can move around on the curve (it’s semantically continuous, as we discussed) to explore nearby, related points. And you can interpolate on the curve between different data points to find their in-between."

This framing seems relevant to pattern learning and better suited to thinking about grokking. If the model has simply memorized the training data, we can't expect interpolation to perform pretty well. For division mod 97, our program database would just be `return 15`, `return 71`, etc. If instead, we have learned a general rule – `lambda x, y, z: (x / y) % z` – or some nuanced abstractions, our model may perform surprisingly well, even perfectly, when tested on unseen samples. It's incredibly difficult to imagine how powerful small side-steps in a well-structured latent space can eventually become.

>"Looking at it, it certainly looks like these LLM's and these Al systems, in some sense they're just interpolators, but the level of abstraction at which they're interpolating keeps going up and up and up, and we keep sort of riding up that chain of abstractions. And then, presumably from a sufficiently elevated point of view, the invention of General Relativity from Newtonian physics is just interpolation at some sufficiently grandiose level of abstraction that perhaps tells us something about the nature of intelligence, human intelligence, as well as about these large language models."
><author>Adam Brown</author>
><cite>[^5]</cite>

And we actually do have evidence that both memorization and procedural knowledge are at play in current models[^6]. By using attribution methods, we can actually _see_ that answering factual questions like "What is the largest ocean on Earth" strongly attribute to documents from pretraining that answer them specifically. Problems that require reasoning, like math or code, instead have weak attribution spread over many examples of related programs/abstractions. This implies that reasoning might just require a dense enough sample distribution of reasoning traces such that continuous interpolation along the ridges of the reasoning latent space can be applied generically.

So how do we push models towards learning circuits/programs that generalize? Here's where we circle back to MDL and Solmonoff Induction. We need to implement an inductive bias towards less-complex programs: gradient pressure towards the Occam's Razor solution. But how do we measure complexity? Naive parameter count doesn't work.

If we crack out our mechanistic interpretability toolkit, we can start talking about the underlying _circuits_ in the model actually responsible for computation. As Neel Nanda et al. show[^7], there are multiple circuits that are learned during training and achieve strong performance. When we include weight decay, over time our model will learn to prefer circuits with high “efficiency”, i.e. circuits that require less parameter norm to produce a given logit value. In _Explaining grokking through circuit efficiency_[^8], these results are supported; their interp identified three phases: memorization, then formation of circuits (before grokking phase), then cleanup (grokking), as the circuit both solves the task well and has lower weight than the memorized parameters, meaning weight decay encourages the network to shed the memorized solution.

I can buy that parameter norm can act as a proxy for complexity and is a good inductive bias to include, but it doesn't feel like it's capturing the full picture. If we want to see MDL in action, we need to actually lock in on a complexity metric. What if we try compression?

Unfortunately, naïvely trying to compress the weights and tracking the size reduction seems to produce poor complexity bounds. This is due to our model containing random information left over from initialization and the stochasticity of training. If we could surgically strip away this random information, compressing the final parameters would give much tighter bounds. However, it is actually undecidable whether information is random or not!

> Given a machine M, and a string x of length n, construct a decider D which, given the machine and two strings, accepts if the first string is a program that encodes the second string, and rejects if the first string does not encode the second string. Create a machine that will run this decider on all strings of length n-1, and accept iff the decider accepts for one of the strings. If one or more of the strings of length n-1 does not halt, then the decider would be able to solve the halting problem. Since this is not possible, creating a machine which determines if a string is Kolmogorov random has to also be impossible.
> [^9]

Thankfully, there's a workaround. We can decide if the information is random or not by _seeing if removing it affects model performance_. We can use the difference in loss before/after compression as our rate-distortion metric! To do so, we can take the spectral entropy of each weight matrix (the effective rank) and measure the intrinsic dimension of the transformation parameterized by M. Adding this term to our loss function speeds up time to grokking![^10] We've now moved from parameter-norm as our surrogate for complexity to actual compressability.

Past other methods[^11] that use our understanding of grokking to speed up the phase-transition, I'm looking for future optimization techniques to speed-run memorization and more directly target generalization. At the limit, this looks like a lightweight (~1B?) reasoning model with little to no factual content memorized. A model that, given enough context[^12], may start to do incredible things on your local machine.


[^1]: <https://www.princeton.edu/~wbialek/rome/refs/shannon_51.pdf>
[^2]: <https://arxiv.org/pdf/2201.02177>
[^3]: <https://lilianweng.github.io/posts/2017-09-28-information-bottleneck/>
[^4]: <https://arxiv.org/pdf/2303.06173>
[^5]: <https://youtu.be/XhB3qH_TFds?si=JwnrIhCEB7bWToyB&t=2460>
[^6]: <https://arxiv.org/pdf/2411.12580>
[^7]: <https://arxiv.org/pdf/2309.02390>
[^8]: <https://arxiv.org/pdf/2301.05217>
[^9]: <https://www.cs.princeton.edu/courses/archive/fall11/cos597D/L10.pdf>
[^10]: <https://arxiv.org/pdf/2412.09810>
[^11]: <https://arxiv.org/pdf/2405.20233>
[^12]: <https://jan.ai/docs/jan-models/jan-nano-128>