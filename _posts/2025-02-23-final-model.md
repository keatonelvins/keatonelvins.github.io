---
layout: post
title: "transformer v7.1++ final (last)"
image_name: "nsa"
tag: "deep learning"
---

Found a [recent thread](https://x.com/i/bookmarks?post_id=1891039294275338452) where a bunch of highly online ML anons tossed out ideas on what the MVP Final Architecture™ might look like and wanted to unpack the takes + jargon (being real I needed to dig through a good amount of papers for this).

---
>- ≈DS-MoE
>- Neural memory layers 
>- 1:4 MLA:GDN
>- Inner looping (from SUT to Huginn-like?) for adaptive compute (1) 
>- Well-fried reasoning (DeepScaleR+) for adaptive compute (2)
>- ≈Entropix adaptive decoding 
>- KV deliberation
><author>@Teortaxes</author>

The DS-MoE architecture has changed over time, but the latest we know of comes from DeepSeek V3[^1]. There are two core changes from the vanilla transformer (plus some norming tricks): each layer has the standard attention block replaced with a Multi-head Latent Attention (MLA) block and the FFN block with a DeepSeekMoE block (fine-grained experts with auxiliary-loss-free load balancing[^2]).

![DS-MoE Architecture](/assets/posts/deepseekmoe.png){:class="img-responsive"}

In MLA, we just cache the down-projected latent vector $$c^{𝐾𝑉}_t =𝑊^{𝐷𝐾𝑉}\mathbf{h}_t$$ at the $$t$$-th token (instead of both $$k_t = W^K\mathbf{h_t}$$ and $$v_t = W^V\mathbf{h_t}$$) and up-project it at inference time to get $$k^{C}_t =𝑊^{UK}c^{KV}_t$$ and $$v^{C}_t =𝑊^{UV}c^{KV}_t$$ (+ a similar process for the query and some RoPE shenanigans). This means for each additional token, given $$n_h$$ heads and $$d_h$$ dimension per head, we only need to cache $$\approx \frac{9}{2}d_hl$$ elements. Compared to $$2n_hd_hl$$ for MHA, this scales much better and performs very well!

For the DeepSeekMoE block, we split the FFN into $$N = N_s + N_r$$ parallel FFN's, or "experts". The $$N_s$$ chunk of experts are _shared_, which means they are used on every forward pass. The $$N_r$$ chunk are _gated_, so the input is only routed through the top-$$K_r$$ of them according to a calculated token-to-expert affinity value $$s_{i,t} = \sigma(\mathbf{u_t}^{\top}\mathbf{e_i})$$, where $$e_i$$ is the learned centroid of expert-$$i$$. And to mitigate a skewed distribution of affinity values (the so called "favorite child problem"), a bias term $$b_i$$ is directly added to $$s_{i,t}$$ before calculating the top-$$K_r$$. During training, the expert load is monitored after each step, and each bias term is nudged up or down accordingly. This enables balanced routing without any modification to the loss function (meaning no interfering gradients!).

There are a bunch of takes on neural memory layers, but the basic idea is just augmenting the model with explicit read/write operations with some 'long-term memory module' (see _Memory Layers at Scale_[^3] or _Titans_[^4]). With attention layers (already softened differentiable lookups), the keys/values are the _activations_ after projecting the hidden state via trainable matrices. With a neural memory layer, the keys/values are themselves _directly trainable_.

The 1:4 ratio is how we select layers that use global vs local attention; local attention is usually implemented with a sliding window (i.e. tokens only attend to each other within the window), while global is the standard flavor we're used to. For example, the Gemma 2[^5] family alternated between global and sliding window attention (SWA), while other ratios like 1:6[^6] and 1:3[^7] have also been seen in production. One important detail is that the actual attention mechanism used within the respective global and SWA blocks can be anything (e.g. Global Query Attention[^8] + a million other variants on MHA/MQA).

Inner looping is a really interesting idea where the depth of the transformer can be set dynamically at test-time. As seen in the recent _Latent Reasoning_[^9] paper, the core computation is done by looping the sequence through a _recurrent block_, with just two separate blocks to project in and out of the latent recurrent space. At test time, we can unroll/telescope out the model to an arbitrary depth by just increasing the number of iterations through the recurrent block. This is a new method of inference-time scaling in contrast with simply increasing the number of forward passes (from CoT to reflection to something like s1[^10]).

DeepScaleR[^11] was another recent project using extended GRPO on `DeepSeek-R1-Distill-Qwen-1.5B`. Their key contribution was capping the sequence length at 8k tokens for the first 1000 steps of training, then extending to 16K and eventually 24k tokens. I think the "well-fried" here comes from taking a slightly less naïve approach than endless GRPO scaling, but I'm not yet convinced this is necessary. Feels like their should be a more elegant solution for getting reasoning models to respond to "Hi" without thinking for 8k tokens.

Entropix adaptive decoding[^12] is an _awesome_ decoding technique that relies on the entropy and varentropy of output logits at inference time to influence the sampling strategy. I've seen some related papers I couldn't track down, but generally there seems to be a ton of low-hanging fruit with sampling past greedy/beam/nucleus/etc. If we can dynamically invest compute into generating high-entropy tokens (token importance for a given sequence is NOT uniform) with branching or more sampling, we can see similar gains to inference-time scaling. 

I hadn't heard of KV-deliberation[^13] before, but it's a crazy idea. If we generate a KV-cache using a frozen LLM and pass it to a coprocessor, we can asynchronously create latent embeddings, use them to augment the original KV-cache, and continue decoding with the frozen model. The hope is that these embeddings will improve the fidelity of this subsequent decoding (and empirically it seems to consistently reduce perplexity + improves performance). To train the coprocessor, we can just use the decoder gradients while keeping the decoder itself frozen (frozen so that the coprocessor can operate offline/async, and the model won't collapse if the coprocessor is unavailable or if a given cache is deemed not to require extra computation).

I'd expect improvements from each technique to compound on each other, but it still feels like something data-related is missing. Maybe with n-trillion frontier lab quality tokens it's possible, but I wouldn't expect this + CommonCrawl to immanentize the eschaton. 

---
>- 1:3 full attention with no RoPE with MLA
>- SWA (window size ~ 8k) with RoPE (potentially replace with RNN)
>- ~Deep MoE but not too deep for latency reasons
>- Value residuals(?)
>- Potentially diff attention (only on full or SWA layers)
>- VSL for training (useful for long context, fast, stable)
>- Metadata conditioning (backed by physics of llms)
>- MEAP (mask 15% of tokens during training). May give some MTP behavior
>- Great data (super important, see Nemotron CC)
>- Reasoning and instruction following data in the annealing stage.
>- Use simple muP with good inits for optimization
>- Second order optimizer or Adam variant (Adam mini or adEMAmix)
>- WSD lr schedule, can drop lr at some points like Deepseek
>- Long CE for loss during ft/annealing for long context performance
>- Batch size scheduler (see llama 3/deepseek)
>- Rho type loss (maybe use an rnn model to sift through the data for that)
><author>@Grad</author>

As above, 1:3 implies 3 SWA blocks then 1 global block for each contiguous chunk of 4 layers. The SWA uses Rotary Positional Encoding  (used by Mistral, Qwen, Olmo, Deepseek) and can be extended to long context with YaRN[^14]. This architecture comes courtesy of the Cohere folks[^15] where they ablate RoPE and no positional encoding (NoPE) and end up interleaving 1:3 RoPE to NoPE layers in a novel approach (RNope-SWA though? really?). From analysis, NoPE layers in RNoPE had strong retrieval (spikes in attention mass on the retrieved tokens and attention sinks[^16] on beginning tokens) but bad recency bias compared to pure RoPE or pure NoPE. RoPE layers instead had bad retrieval (little/no attention sink on early or retrieved tokens) with very stronger recency bias. 1:3 seemed to strike a good balance on long (64k - 256k) and short (8k-32k) tasks while minimizing KV cache size and boosting inference speed. SWA feels analogous to hidden-state mechanics in RNN's, so I can see how hot-swapping might make sense.

Depth is sooo important. There's probably some $$N$$-layers sweet spot where inference is reasonable but there's still sufficient computational capacity per forward pass to reach critical mass, but at the moment I'm very depth-pilled (looking at you Mistral although I get the use case). Although I've seen multiple papers doing search in MoE-space that claim DSV3 is near-optimal.

Value residuals[^17] is part of a longstanding effort to enable effective propagation of initial information to deeper layers. Adding hidden state residuals or 'skip connections' (see ResNet) was huge for deep learning at large – addressing vanishing gradients and rank degeneration – but the problem is not solved. With very deep nets, blocks can become smooth with only minute difference between hidden states layer to layer. Value residuals introduce an additional residual stream between the value vectors of the current layer and the first layer before the attention operation. This allows token-level raw information to propagate directly from the input sequence.

---
>- DeepSeek MoE with more (and larger) experts and depth (at least 2x compared to V3). 
>- DS team appears to have optimized amazingly for hardware, but this is what you'd get if you optimized today for an 8xB200 or GHB200 nvl72 (~4T params or so?) trained for 30T - 40T tokens (using the gemini / gemma tokenizer) 
>	- 40T tokens can be achieved with synthetic data including CoT and is probably the frontier minimum
>- 1:8 Noam Style attention
>- No RoPE or GQA on global (maybe MLA on global here tho im not completely sold), 
>- RoPE on local with 4 - 12 sink tokens (although not necessarily all at the start), 4096 context window, and GQA
>- DiLoCo / DiPaCO + PSD style optimizer curriculum pretraining 
>- R1 Zero style post training
>- Mixture of Depth based on complexity that dynamically adjusted layer count and number of contributing experts in the router MTP (with UL2) but like 12 - 24 heads (should also be part of the MoD) 
>- Entropix sampler test time compute harness via entropy aware branching + search
><author>@xjdr</author>

Lot of similar notes as above: MLA, NoPE, RoPE, 1:8 ratio all mentioned. Interesting to see 40T tokens, we recently got the Qwen3 technical report which claims 36T tokens. A TON of these are synthetically generated from their domain specific Math and Coder lines. Expect to see more of this and scaling rollouts in diverse tool/MCP environments.

---
>- 1:4/1:8 global with no positional encoding, instead use MLA/some other efficient-attention flavor
>- Standard SWA for the rest of the layers
>- More depth (pureist view against architecture hacks) 
>- Math/code/instruction following RLVR (will transfer to intelligence in other domains including multimodal!) 
>- RLAIF for writing tasks 
>- GRPO++ (maybe RLOO + PRIME, or some mix of vinePPO) instead of PPO
>- Toggle reasoning on/off by a system prompt
>- Search (likely what o* pro does)
>- Better "learned functions"/PRMs/ORMs (e.g. take distilled reasoner, output CoT then special label token)
><author>@wh</author>

---

TODO:

https://arxiv.org/abs/2501.01956

https://arxiv.org/abs/2502.07490

https://arxiv.org/abs/2410.23771

https://arxiv.org/abs/2210.13432

https://arxiv.org/pdf/2502.11089

https://arxiv.org/pdf/2502.13189


[^1]: <https://arxiv.org/pdf/2412.19437>
[^2]: <https://arxiv.org/pdf/2408.15664>
[^3]: <https://ai.meta.com/research/publications/memory-layers-at-scale/>
[^4]: <https://arxiv.org/pdf/2501.00663>
[^5]: <https://arxiv.org/pdf/2408.00118>
[^6]: <https://research.character.ai/optimizing-inference/>
[^7]: <https://arxiv.org/pdf/2412.13663>
[^8]: <https://arxiv.org/pdf/2305.13245>
[^9]: <https://www.arxiv.org/pdf/2502.05171>
[^10]: <https://arxiv.org/pdf/2501.19393>
[^11]: <https://pretty-radio-b75.notion.site/DeepScaleR-Surpassing-O1-Preview-with-a-1-5B-Model-by-Scaling-RL-19681902c1468005bed8ca303013a4e2>
[^12]: <https://github.com/xjdr-alt/entropix>
[^13]: <https://arxiv.org/pdf/2412.17747>
[^14]: <https://arxiv.org/pdf/2309.00071>
[^15]: <https://arxiv.org/abs/2501.18795>
[^16]: <https://arxiv.org/pdf/2309.17453>
[^17]: <https://arxiv.org/pdf/2410.17897>
