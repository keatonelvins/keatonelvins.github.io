---
layout: post
title: "transformer v7.1++ final (last)"
image_name: "nsa"
tag: "deep learning"
---

Found a [recent thread](https://x.com/i/bookmarks?post_id=1891039294275338452) where a bunch of highly online ML anons tossed out ideas on what the MVP Final Architecture™ might look like and wanted to unpack the takes + jargon (being real I needed to dig through a good amount of papers for this).

---
@Teortaxes
- ≈DS-MoE
- Neural memory layers 
- 1:4 MLA:GDN
- Inner looping (from SUT to Huginn-like?) for adaptive compute (1) 
- Well-fried reasoning (DeepScaleR+) for adaptive compute (2)
- ≈Entropix adaptive decoding 
- KV deliberation

The DS-MoE architecture is best explained by their own paper[^1], but the core is two key changes: each layer has the standard attention block replaced with a Multi-head Latent Attention (MLA) block and the FFN block with a DeepSeekMoE block (fine-grained experts with auxiliary-loss-free load balancing[^2]).

![DS-MoE Block](/assets/posts/deepseekmoe.png){:class="img-responsive"}
*Figure 1: DeepSeekMoE Block*

In MLA, we just cache the down-projected latent vector $$c^{𝐾𝑉}_t =𝑊^{𝐷𝐾𝑉}\mathbf{h}_t$$ at the $$t$$-th token (instead of both $$k_t = W^K\mathbf{h_t}$$ and $$v_t = W^V\mathbf{h_t}$$) and up-project it at inference time to get $$k^{C}_t =𝑊^{UK}c^{KV}_t$$ and $$v^{C}_t =𝑊^{UV}c^{KV}_t$$ (+ a similar process for the query and some RoPE shenanigans). This means for each additional token, given $$n_h$$ heads and $$d_h$$ dimension per head, we only need to cache $$\approx \frac{9}{2}d_hl$$ elements. Compared to $$2n_hd_hl$$ for MHA, this scales much better and performs very well!

For the MoE block in DeepSeek V3, we split the FFN block into $$N = N_s + N_r$$ parallel FFN's, or "experts". The $$N_s$$ chunk of experts are _shared_, which means they are used on every forward pass. The $$N_r$$ chunk are _gated_, so the input is only routed through the top-$$K_r$$ of them according to a calculated token-to-expert affinity value $$s_{i,t} = \sigma(\mathbf{u_t}^{\top}\mathbf{e_i})$$, where $$e_i$$ is the learned centroid of expert-$$i$$. And to mitigate a skewed distribution of affinity values (the so called "favorite child problem"), a bias term $$b_i$$ is directly added to $$s_{i,t}$$ before calculating the top-$$K_r$$. During training, the expert load is monitored after each step, and each bias term is nudged up or down accordingly. This enables balanced routing without any modification to the loss function (meaning no interfering gradients!).

The core idea of neural memory layers is just explicit read/write operations to external memory banks (see _Memory Layers at Scale_[^3] or _Titans_[^4]). Compared to attention layers, which already resemble differentiable lookup tables, the keys/values in memory layers are themselves _directly trainable_ (in attention, the keys/values are the _activations_ after projecting the sequence via trainable matrices). 

The 1:4 for MLA:GDN represents the ratio between layers that use local vs global attention; local attention is usually implemented with a sliding window (i.e. tokens only attend to each other within the window), while global is the standard flavor we're used to. For example, the Gemma 2[^5] family alternated between a local sliding window and global attention, while other ratios like 1:6[^6] and 1:3[^7] have also been seen in production. 

Inner looping is a really interesting idea where the depth of the transformer is set dynamically dependent on task difficulty. As seen in the recent _Latent Reasoning_[^8] paper,

...

---
@Grad
- 1:3 full attention with no RoPE with MLA
- SWA (window size ~ 8k) with RoPE (potentially replace with RNN)
- ~Deep MoE but not too deep for latency reasons
- Value residuals(?)
- Potentially diff attention (only on full or SWA layers)
- VSL for training (useful for long context, fast, stable)
- Metadata conditioning (backed by physics of llms)
- MEAP (mask 15% of tokens during training). May give some MTP behavior
- Great data (super important, see Nemotron CC)
- Reasoning and instruction following data in the annealing stage.
- Use simple muP with good inits for optimization
- Second order optimizer or Adam variant (Adam mini or adEMAmix)
- WSD lr schedule, can drop lr at some points like Deepseek
- Long CE for loss during ft/annealing for long context performance
- Batch size scheduler (see llama 3/deepseek)
- Rho type loss (maybe use an rnn model to sift through the data for that)

---
@xjdr

- DeepSeek MoE with more (and larger) experts and depth (at least 2x). 
- DS team appears to have optimized amazingly for hardware, but this is what you'd get if you optimized today for an 8xB200 or GHB200 nvl72 (~4T params or so?) trained for 30T - 40T tokens (using the gemini / gemma tokenizer) 
	- 40T tokens can be achieved with synthetic data including Chain of Thought and is probably the frontier minimum
- 1:8 Noam Style attention
- No RoPE or GQA on global (maybe MLA on global here tho im not completely sold), 
- RoPE on local with 4 - 12 sink tokens (although not necessarily all at the start), 4096 context window, and GQA
- DiLoCo / DiPaCO + PSD style optimizer curriculum pretraining 
- R1 Zero style post training
- Mixture of Depth based on complexity that dynamically adjusted layer count and number of contributing experts in the router MTP (with UL2) but like 12 - 24 heads (should also be part of the MoD) 
- Entropix sampler test time compute harness via entropy aware branching + search

---
@wh

- 1:4/1:8 global with no positional encoding, instead use MLA/some other efficient-attention flavor
- Standard SWA for the rest of the layers
- More depth (pureist view against architecture hacks) 
- Math/code/instruction following RLVR (will transfer to intelligence in other domains including multimodal!) 
- RLAIF for writing tasks 
- GRPO++ (maybe RLOO + PRIME, or some mix of vinePPO) instead of PPO
- Toggle reasoning on/off by a system prompt
- Search (likely what o* pro does)
- Better "learned functions"/PRMs/ORMs (e.g. take distilled reasoner, output CoT then special label token)

---
DeepSeek Native Sparse Attention: https://arxiv.org/pdf/2502.11089

much more elegant version of my preferred attention arch (1:8 global / swa local with MLA on the global)

https://research.character.ai/optimizing-inference/ (Noam Shazeer attention?)


[^1]: https://arxiv.org/pdf/2412.19437
[^2]: https://arxiv.org/pdf/2408.15664
[^3]: https://ai.meta.com/research/publications/memory-layers-at-scale/
[^4]: https://arxiv.org/pdf/2501.00663
[^5]: https://arxiv.org/pdf/2408.00118
[^6]: https://research.character.ai/optimizing-inference/
[^7]: https://arxiv.org/pdf/2412.13663
[^8]: https://www.arxiv.org/pdf/2502.05171
