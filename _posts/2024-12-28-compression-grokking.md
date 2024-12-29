---
layout: post
title: "i just wanna grok"
image_name: "jetpack"
tag: "deep learning"
---

>“He should have known better because, early in his learnings under his brother Mahmoud, he had discovered that long human words (the longer the better) were easy, unmistakable, and rarely changed their meanings . . . but short words were slippery, unpredictable, changing their meanings without any pattern. Or so he seemed to grok. Short human words were never like a short Martian word—such as “grok” which forever meant exactly the same thing. Short human words were like trying to lift water with a knife.”
>Robert Heinlein, _Stranger in a Strange Land_

I had to convert a folder to a .zip the other day and had a little moment. I think I often take it for granted that _lossless compression_ is a thing we can just do. The [Wikipedia page](https://en.wikipedia.org/wiki/ZIP_(file_format)#Compression_methods) for ZIP was honestly kind of bland, although I did find out that the "zip" component was originally of the "moving at high speed" flavor and not of "fasten with a zipper". Maybe it's time to replace those little zipper icons with a 🏃🏻💨.

Compression, as I first learned from Ilya Sutskever in a recorded [lecture](https://www.youtube.com/watch?time_continue=1273&v=AKMuA_TVz3A&embeds_referring_euri=https%3A%2F%2Fwww.lesswrong.com%2F&source_ve_path=Mjg2NjY) he gave at the Simons Institute, is not just relevant for sending big files through email. It seems to be very deeply connected with learning and understanding. Marcus Hutter, researcher at DeepMind and organizer of the [Hutter Prize](https://en.wikipedia.org/wiki/Hutter_Prize), believes that "text compression and AI are equivalent problems":

> "In order to compress data, one has to find regularities in them, which is intrinsically difficult... So compressors beating the current 'dumb' compressors need to be smart(er). Since the prize wants to stimulate developing 'universally' smart compressors, we need a 'universal' corpus of data. Arguably the online encyclopedia Wikipedia is a good snapshot of the Human World Knowledge. So the ultimate compressor of it should 'understand' all human knowledge, i.e. be really smart."

If the distribution/generator your data comes from is not just uniform, then we can expect some variables to occur more frequently than others. We can find correlations and identify patterns. "Dog" is probably more likely to follow the phrase "My new pet..." than "axolotl", and this _tells_ us something about humans and dogs and axolotls.

One lens we can understand language modeling through is the statistical next-token predictor. The autoregressive text generator extremely fitted to the distribution curve of natural language that spits out high-probability tokens accordingly. This idea is actually super old, going back at least to the 1950's with Shannon's _Prediction and Entropy of Printed English_[^2]. Here, Shannon references two key concepts to language modeling: _entropy_ and _redundancy_.

> "If the language is translated into binary digits (0 or 1) in the most efficient way, the entropy is the average number of binary digits required per letter of the original language. The redundancy, on the other hand, measures the amount of constraint imposed on a text in the language due to its statistical structure, e.g., in English the high frequency of the letter E, the strong tendency of H to follow T or of V to follow Q."

The 'dumb' compressors referenced by Hutter exploit this redundancy of natural language. If characters in english were uniformly random, we would predict each letter to have an entropy of around 5 bits. Instead, due to redundancy, it's probably closer to 1-1.5 bits. This gap is what enables lossless compression, algorithms like [LZ77](https://en.wikipedia.org/wiki/LZ77_and_LZ78), to actually work! This fact also lends itself to algorithms like [Byte-Pair Encoding](https://en.wikipedia.org/wiki/Byte_pair_encoding) (used during tokenization for all mainstream LLMs) that chunk commonly co-occuring chars into larger n-grams.

By Shannon’s source coding theorem, the expected optimal coding rate for samples drawn from a distribution is equal to the entropy of the distribution. If we can decrease the apparent entropy of the data by identifying patterns in it, we can develop a better encoding! Let's apply these ideas to language models and formalize.

We can model our distribution entropy as $H(D)$ and our model complexity as $C(M)$. We know that the distribution entropy is fixed, but we can reduce $H(D\  |\  M)$, or the _data entropy under the model_. The sum of these two terms is what we want to minimize in our final trained model.

$$ \min_M H (D\ |\ M ) + C(M)$$

If our model is just a lookup table of our training data, we have achieved $H(D\  |\  M) = 0$. The apparent entropy of the data has been reduced completely. However, this approach is so naïve that our $C(M)$ is exactly equal to $H(D)$, and we haven't compressed anything!

We want a model that compresses the data best, and is itself most compressible. 

Before we dive in to this idea and how it relates to [Occam's Razor](https://en.wikipedia.org/wiki/Occam%27s_razor), [MDL](https://en.wikipedia.org/wiki/Minimum_description_length), [Kolmogorov Complexity](https://en.wikipedia.org/wiki/Kolmogorov_complexity), [Algorithmic Information Theory](https://en.wikipedia.org/wiki/Algorithmic_information_theory), and [Solmonoff Induction](https://en.wikipedia.org/wiki/Solomonoff%27s_theory_of_inductive_inference) (warning: high rabbit-hole potential), I want to introduce the deep learning phenomena known as _grokking_.

> “Grok’ means to understand so thoroughly that the observer becomes a part of the observed – to merge, blend, intermarry, lose identity in group experience."
> Robert Heinlein, _Stranger in a Strange Land_

The OpenAI team behind the og grokking paper[^3] identified that small, algorithmically generated datasets could prove to be a fertile ground for studying generalization of overparametrized neural networks beyond memorization of the training data. In practice, they showed that significantly into the overfitting regime (according to our classical understanding of deep learning), the models would suddenly jump from random guesswork to perfect generalization on validation data.

From an information theory perspective[^4], the writing had been on the wall. Results had already demonstrated that DNN training involves two distinct phases: first, the network learns to fully represent the training data; then, it learns to compress the representation of the input and forgets irrelevant details. But, by studying algorithmically generated datasets, e.g. training on division mod 97, researchers could now explore the exact recipes and circuits that cause this to happen.

The first follow-up paper I read was on "Unifying Grokking and Double Descent"[^5], where the authors viewed both phenomena as inductive biases applying gradient pressure towards better-generalizing but slower to learn patterns. This is the lens of _pattern learning_, which captures the idea that neural networks can use distinct kinds of mechanisms to classify different inputs; for example, they may memorize some examples while applying heuristics to classify others. Grokking happens when slow patterns generalize well and are ultimately preferred by the training regime, but are preceded by faster patterns which generalize poorly. Patterns can be viewed as a function of model size as well as a function of training time, and learning dynamics are similar between these two cases.

Earlier I mentioned we can think about language models in their base autoregressive text generator state. Another mental model I like, borrowed from [Francois Chollet](https://fchollet.substack.com/p/how-i-think-about-llm-prompt-engineering), is the _continuous, interpolative_ program database. 

>"Instead of being stored as a set of discrete entries, your data is stored as a vector space — a curve. You can move around on the curve (it’s semantically continuous, as we discussed) to explore nearby, related points. And you can interpolate on the curve between different data points to find their in-between."

This framing seems relevant to pattern learning and better suited to thinking about grokking. If the model has simply memorized the training data, we can't expect interpolation to perform pretty well. For division mod 97, our program database would just be `return 15`, `return 71`, etc.  If instead, 

[1]: https://arxiv.org/pdf/2004.07780
[2]: https://www.princeton.edu/~wbialek/rome/refs/shannon_51.pdf
[3]: https://arxiv.org/pdf/2201.02177
[4]: https://lilianweng.github.io/posts/2017-09-28-information-bottleneck/
[5]: https://arxiv.org/pdf/2303.06173