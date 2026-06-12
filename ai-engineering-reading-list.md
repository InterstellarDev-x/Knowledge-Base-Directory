# AI Engineering: From First Principles
### A Curated Learning Curriculum — Papers, Blogs & Engineering Lessons

> **How to use this guide:** Follow the stages in order. Each stage has a *why it matters* framing, *what to understand* before moving on, and *questions to sit with* after. The goal is not speed — it's building an accurate mental model of how these systems actually work.

> Claims marked ✅ were adversarially verified across 115 agents, 32 sources, 158 claims extracted — 18 confirmed, 7 killed.

---

## Table of Contents

- [The Learning Path at a Glance](#the-learning-path-at-a-glance)
- [Stage 1 — Mathematical & ML Foundations (Week 1–2)](#stage-1--mathematical--ml-foundations)
- [Stage 2 — Deep Learning Architectures (Week 2–4)](#stage-2--deep-learning-architectures)
- [Stage 3 — Attention & The Transformer (Week 4–5)](#stage-3--attention--the-transformer)
- [Stage 4 — Pre-training & Scaling Laws (Week 5–7)](#stage-4--pre-training--scaling-laws)
- [Stage 5 — Large Language Models (Week 7–9)](#stage-5--large-language-models)
- [Stage 6 — Training at Scale (Week 9–11)](#stage-6--training-at-scale)
- [Stage 7 — Alignment & Instruction Tuning (Week 11–12)](#stage-7--alignment--instruction-tuning)
- [Stage 8 — Inference & Serving (Week 12–14)](#stage-8--inference--serving)
- [Stage 9 — AI Engineering in Production (Week 14–16)](#stage-9--ai-engineering-in-production)
- [Practitioner Blogs & Essential Reading](#practitioner-blogs--essential-reading)
- [Full Paper Reference](#full-paper-reference)
- [Concept Dependency Map](#concept-dependency-map)
- [Resources & Courses](#resources--courses)

---

## The Learning Path at a Glance

```
FOUNDATIONS               ARCHITECTURE              SCALE & PRODUCTION
───────────               ────────────              ──────────────────
Math & ML basics   ──►   Transformer         ──►   Training at Scale
     │                        │                     (ZeRO, Megatron,
     ▼                        ▼                      FSDP, MixedPrec)
Deep Learning      ──►   Pre-training &              │
(CNN, RNN, LSTM,         Scaling Laws        ──►   Inference & Serving
 ResNet, BN, DO)              │                     (KV cache, vLLM,
                              ▼                      Speculative Dec,
                         LLMs                        Quantization)
                     (GPT, BERT, LLaMA,              │
                      Mistral, Gemini)        ──►   Production
                              │                     (RAG, Evals,
                              ▼                      Agents, LoRA,
                         Alignment                   Fine-tuning)
                     (RLHF, DPO, CAI, PPO)
```

**Estimated time:** ~16 weeks at 5–8 hrs/week. Stages 1–5 are load-bearing for everything else — don't skip them.

---

## Stage 1 — Mathematical & ML Foundations

> **The core question:** What is a learning algorithm, and how does it improve from data?

### Why start here

Everything in AI — transformers, LLMs, diffusion models — is gradient descent on a loss function, applied at scale. If you don't understand backpropagation, the chain rule, and optimization deeply, you will be cargo-culting your way through the rest of the field. This stage is short but non-negotiable.

---

### 1.1 — Backpropagation

**Paper:** Learning representations by back-propagating errors
**Authors:** Rumelhart, Hinton, Williams · **Year:** 1986 · **Difficulty:** ★★☆☆☆
**Link:** [Nature](https://www.nature.com/articles/323533a0)

**Why it matters:** The paper that made neural networks trainable. Backpropagation is just the chain rule applied systematically to compute gradients through a computational graph. Every training run in deep learning is a descendant of this paper. Read it once to understand the derivation; the insight is simpler than it seems.

**What to understand:**
- The chain rule: if `L = f(g(x))`, then `dL/dx = (dL/dg)(dg/dx)`
- How the forward pass computes activations; how the backward pass accumulates gradients
- Why we need to store intermediate activations during the forward pass

**Questions to sit with:**
- Why does backprop through time (BPTT) cause vanishing gradients in RNNs?
- What does it mean for a gradient to "vanish" — why is a near-zero gradient bad for learning?

---

### 1.2 — Optimization: SGD, Adam, and Beyond

**Key papers:**
| Paper | Authors | Year | Link | Notes |
|---|---|---|---|---|
| Stochastic Gradient Descent (original) | Robbins & Monro | 1951 | — | The algorithm itself |
| Adam: A Method for Stochastic Optimization | Kingma & Ba | 2014 | [arXiv](https://arxiv.org/abs/1412.6980) | Default optimizer for most LLMs |
| Decoupled Weight Decay Regularization (AdamW) | Loshchilov & Hutter | 2017 | [arXiv](https://arxiv.org/abs/1711.05101) | Adam + correct L2 regularization |
| An Overview of Gradient Descent Optimization Algorithms | Ruder | 2016 | [arXiv](https://arxiv.org/abs/1609.04747) | Best survey of optimizers |

**Why Adam matters:** Adam is the optimizer behind GPT, BERT, LLaMA, Stable Diffusion, and essentially every large model trained today. It adapts learning rates per-parameter using first and second moment estimates of gradients. AdamW fixes a subtle but important bug in Adam's weight decay implementation.

**What to understand:**
- Why mini-batch SGD works: the gradient of a random subset is an unbiased estimator of the true gradient
- Momentum: why does using past gradient history help?
- Adam's `m` (first moment) and `v` (second moment) estimates and the bias correction terms
- Why learning rate warmup is almost always used with Adam for LLMs

---

### 1.3 — Regularization & Generalization

**Key papers:**
| Paper | Authors | Year | Link | Notes |
|---|---|---|---|---|
| Dropout: A Simple Way to Prevent Neural Networks from Overfitting | Srivastava et al. | 2014 | [JMLR](https://jmlr.org/papers/v15/srivastava14a.html) | |
| L1 and L2 Regularization | — | — | — | Classical — textbook coverage |
| Deep Double Descent | Nakkiran et al. | 2019 | [arXiv](https://arxiv.org/abs/1912.02292) | Why bigger models generalize better — the modern view |

**Why Double Descent matters:** Classical ML says larger models overfit. Deep learning disproves this empirically: past a certain model size, test loss *decreases* again. This paper formalizes why — the modern justification for scaling LLMs to billions of parameters.

---

### 1.4 — Representation Learning

**Paper:** Representation Learning: A Review and New Perspectives
**Authors:** Bengio, Courville, Vincent · **Year:** 2012/2014 · **Difficulty:** ★★★☆☆
**Link:** [arXiv:1206.5538](https://arxiv.org/abs/1206.5538)

> ✅ **Verified:** The success of ML algorithms depends critically on the choice of data representation — different representations can entangle or expose the underlying explanatory factors of variation in data. This motivates *learning* representations rather than hand-crafting features.

**Why it matters:** This is the philosophical foundation of deep learning. The question isn't just "what algorithm?" — it's "what representation makes the algorithm's job easy?" Word embeddings, image features, KV caches, attention patterns — all of these are representations a model learns. Understanding *why* representation matters gives you the lens to evaluate every architectural choice in the field.

---

## Stage 2 — Deep Learning Architectures

> **The core question:** What architectural primitives — convolutions, recurrence, normalization, residual connections — make deep learning tractable, and why do they work?

---

### 2.1 — Convolutional Neural Networks

**Key papers:**
| Paper | Authors | Year | Link | Notes |
|---|---|---|---|---|
| Gradient-Based Learning Applied to Document Recognition | LeCun et al. | 1998 | [IEEE](http://yann.lecun.com/exdb/publis/pdf/lecun-01a.pdf) | LeNet — backprop-trained CNN |
| ImageNet Classification with Deep CNNs (AlexNet) | Krizhevsky, Sutskever, Hinton | 2012 | [NeurIPS](https://papers.nips.cc/paper/2012/hash/c399862d3b9d6b76c8436e924a68c45b-Abstract.html) | The paper that started the deep learning era |
| Very Deep Convolutional Networks (VGGNet) | Simonyan & Zisserman | 2014 | [arXiv](https://arxiv.org/abs/1409.1556) | Depth over width |
| Going Deeper with Convolutions (Inception/GoogLeNet) | Szegedy et al. | 2014 | [arXiv](https://arxiv.org/abs/1409.4842) | Parallel convolution paths |

**Why AlexNet matters:** AlexNet won ImageNet 2012 by a 10% margin — larger than any prior competition improvement. It validated that GPUs + ReLU + dropout + deep CNNs could work at scale. This single result redirected the entire ML community from hand-crafted features to learned representations.

**What to understand:**
- What does convolution do differently from a fully connected layer? (Translation equivariance, weight sharing)
- What is a receptive field? How does pooling change it?
- Why ReLU (`max(0, x)`) replaced sigmoid/tanh as the default activation?

---

### 2.2 — Normalization

**Key papers:**
| Paper | Authors | Year | Link | Notes |
|---|---|---|---|---|
| Batch Normalization | Ioffe & Szegedy | 2015 | [arXiv](https://arxiv.org/abs/1502.03167) | Normalizes across batch dimension |
| Layer Normalization | Ba et al. | 2016 | [arXiv](https://arxiv.org/abs/1607.06450) | Normalizes across feature dimension — used in Transformers |
| RMSNorm | Zhang & Sennrich | 2019 | [arXiv](https://arxiv.org/abs/1910.07467) | Simplified LayerNorm — used in LLaMA |

**Why this matters:** Without normalization, deep networks are extremely sensitive to weight initialization and learning rate. BatchNorm made deep networks trainable reliably. LayerNorm replaced BatchNorm in Transformers because LM batch sizes can be 1 and batch statistics are unstable. LLaMA uses RMSNorm for efficiency — understanding this evolution tells you why architectural choices in LLMs look the way they do.

---

### 2.3 — Residual Networks

**Paper:** Deep Residual Learning for Image Recognition (ResNet)
**Authors:** He, Zhang, Ren, Sun · **Year:** 2015 · **Difficulty:** ★★★☆☆
**Link:** [arXiv:1512.03385](https://arxiv.org/abs/1512.03385) — 100,000+ citations

> ✅ **Verified:** ResNets improve optimization by reformulating layer targets as residual functions `F(x) := H(x) - x` relative to their inputs rather than unreferenced functions `H(x)`. This makes it easier for layers to learn identity mappings (drive weights to zero) and enables training networks with 152 layers.

**Why it matters:** Residual connections are in every modern architecture — GPT, BERT, LLaMA, ViT, Stable Diffusion. The idea is simple: instead of learning `H(x)` directly, learn the *residual* `F(x) = H(x) - x`, so the network can easily represent the identity function by setting `F(x) = 0`. This removes the degradation problem that made very deep networks impossible to train before 2015.

**What to understand:**
- Why does reformulating as a residual make identity mappings easier to learn?
- What is the degradation problem (not the vanishing gradient problem — they're related but distinct)?
- How do skip connections affect gradient flow during backpropagation?

**Questions to sit with:**
- The Transformer has residual connections around every attention layer and FFN layer. Why is this necessary?
- What would happen to a 96-layer GPT-3 if you removed all residual connections?

---

### 2.4 — Recurrent Networks & LSTMs

**Key papers:**
| Paper | Authors | Year | Link | Notes |
|---|---|---|---|---|
| Long Short-Term Memory (LSTM) | Hochreiter & Schmidhuber | 1997 | [Neural Computation](https://direct.mit.edu/neco/article/9/8/1735/6109/Long-Short-Term-Memory) | Solves vanishing gradient for sequences |
| Learning to Forget: Continual Prediction with LSTM | Gers, Schmidhuber, Cummins | 2000 | — | Adds the forget gate |
| Sequence to Sequence Learning with Neural Networks | Sutskever, Vinyals, Le | 2014 | [arXiv](https://arxiv.org/abs/1409.3215) | Encoder-decoder; the architecture Transformers replaced |
| Neural Machine Translation by Jointly Learning to Align and Translate | Bahdanau et al. | 2014 | [arXiv](https://arxiv.org/abs/1409.0473) | Attention — the direct predecessor to Transformers |

> ✅ **Verified:** The vanishing gradient problem was first identified in Hochreiter's 1991 diploma thesis. LSTM was introduced in a 1995 technical report and published in Neural Computation in 1997. It solves the vanishing gradient problem via the Constant Error Carousel — a constant error flow through the memory cell that bypasses the recurrent weight matrix.

**Why Bahdanau attention matters:** This is the paper that invented attention. In the Seq2Seq encoder-decoder architecture, the decoder looks at a weighted combination of all encoder hidden states rather than just the final one. The weights are learned — the model learns *where to look* in the input. This is the direct conceptual ancestor of self-attention in Transformers.

**What to understand:**
- What are the four LSTM gates (input, forget, output, cell)? What does each control?
- Why does the cell state in LSTM allow gradients to flow without decay? (Constant error carousel)
- What problem does the Bahdanau attention mechanism solve that vanilla Seq2Seq doesn't?

---

## Stage 3 — Attention & The Transformer

> **The core question:** What if, instead of processing tokens sequentially, every token could attend to every other token simultaneously?

### Why this is the hinge point of the entire curriculum

The Transformer (2017) is the most consequential architecture in the history of AI. GPT, BERT, LLaMA, Gemini, Claude, Stable Diffusion, AlphaFold — all Transformers. Everything in Stages 4–9 is built on top of this. Read the paper carefully.

---

### 3.1 — Attention Is All You Need (The Transformer)

**Paper:** Attention Is All You Need
**Authors:** Vaswani, Shazeer, Parmar, Uszkoreit, Jones, Gomez, Kaiser, Polosukhin · **Year:** 2017 · **Difficulty:** ★★★☆☆
**Link:** [arXiv:1706.03762](https://arxiv.org/abs/1706.03762) — ~100,000 citations

> ✅ **Verified:** The Transformer achieved 28.4 BLEU on WMT 2014 English-to-German (surpassing all prior results including ensembles by more than 2 BLEU points) and 41.8 BLEU on WMT 2014 English-to-French (new single-model state-of-the-art).

**Why it matters:** The Transformer replaces recurrence with self-attention, enabling full parallelism during training. Every token attends to every other token. This makes training dramatically faster on GPUs (no sequential dependency) and allows the model to learn long-range dependencies that RNNs struggle with.

**What to understand:**
- **Scaled dot-product attention:** `Attention(Q,K,V) = softmax(QKᵀ/√d_k)V` — what does each of Q, K, V represent?
- **Multi-head attention:** why split into multiple heads? What does each head potentially specialize in?
- **Positional encoding:** the Transformer has no built-in notion of order — why? How do positional encodings inject order?
- **The encoder-decoder structure** in the original paper vs. the decoder-only (GPT) and encoder-only (BERT) variants that followed
- **Why the `1/√d_k` scaling?** (Prevents softmax from saturating in high dimensions)
- **Feed-forward sublayers:** what do the two linear layers + ReLU do? Why are they needed at all?

**Questions to sit with:**
- Attention is O(n²) in sequence length. Why is this a problem for very long sequences?
- In multi-head attention, the model learns Q, K, V projection matrices. What would happen if they were all initialized identically?
- The paper uses both encoder and decoder. Why did GPT use only the decoder? Why did BERT use only the encoder?

---

### 3.2 — Variants That Improved the Transformer

**Key papers:**
| Paper | Authors | Year | Link | Notes |
|---|---|---|---|---|
| Improving Language Understanding by Generative Pre-Training (GPT-1) | Radford et al. | 2018 | [OpenAI](https://openai.com/research/language-unsupervised) | Decoder-only Transformer for unsupervised pre-training |
| BERT: Pre-training of Deep Bidirectional Transformers | Devlin et al. | 2018 | [arXiv:1810.04805](https://arxiv.org/abs/1810.04805) | Encoder-only, masked LM pre-training |
| Transformer-XL: Attentive Language Models Beyond a Fixed-Length Context | Dai et al. | 2019 | [arXiv](https://arxiv.org/abs/1901.02860) | Recurrence mechanism for longer context |
| RoPE: Rotationally Positional Encoding | Su et al. | 2021 | [arXiv](https://arxiv.org/abs/2104.09864) | Relative positional encoding — used in LLaMA, GPT-NeoX |
| FlashAttention: Fast and Memory-Efficient Exact Attention | Dao et al. | 2022 | [arXiv](https://arxiv.org/abs/2205.14135) | IO-aware attention — 2–4× faster, critical for long context |
| FlashAttention-2 | Dao | 2023 | [arXiv](https://arxiv.org/abs/2307.08691) | Further improvements |

> **BERT verified:** ✅ BERT uses a Masked Language Model objective masking 15% of all WordPiece tokens at random, enabling deep **bidirectional** representations (unlike GPT which is unidirectional left-to-right). BERT-Base: 110M params (12 layers, 768 hidden, 12 heads). BERT-Large: 340M params (24 layers, 1024 hidden, 16 heads). BERT-Large achieved a GLUE score of 80.5% (+7.7% absolute improvement) and SQuAD v1.1 F1 of 93.2, setting new state-of-the-art on eleven NLP tasks.

**Why FlashAttention matters:** Standard attention materializes the full N×N attention matrix in GPU HBM (slow memory). FlashAttention reorders the computation to operate in fast SRAM, never writing the full matrix to HBM. This isn't an approximation — it produces the same result but 2–4× faster and with O(N) memory instead of O(N²). Almost every modern LLM training and inference stack uses it.

**What to understand:**
- GPT-1 vs BERT: what does the choice of unidirectional vs bidirectional pre-training imply for the downstream tasks each is suited to?
- Why is BERT bad at text generation? Why is GPT bad at token classification?
- What is RoPE and why is it preferred over learned absolute positional encodings?

---

## Stage 4 — Pre-training & Scaling Laws

> **The core question:** How much does scale matter — and is there an optimal ratio of model size to training data?

---

### 4.1 — The Language Modeling Pre-training Paradigm

**Key papers:**
| Paper | Authors | Year | Link | Notes |
|---|---|---|---|---|
| GPT-1 | Radford et al. | 2018 | [OpenAI](https://openai.com/research/language-unsupervised) | Pre-train decoder, fine-tune on downstream tasks |
| Language Models are Unsupervised Multitask Learners (GPT-2) | Radford et al. | 2019 | [OpenAI](https://openai.com/research/better-language-models) | Zero-shot task completion; "too dangerous to release" |
| Scaling Laws for Neural Language Models | Kaplan et al. | 2020 | [arXiv](https://arxiv.org/abs/2001.08361) | Power-law relationships between compute, model size, data |

**Why GPT-2 matters:** GPT-2 (1.5B params) showed that a language model, with no task-specific training, could perform translation, summarization, and QA just by conditioning on text. This was the first concrete evidence that scale → capability, and that pre-training on raw text was a general-purpose algorithm.

**What to understand (Kaplan scaling laws):**
- Loss scales as a power law with model size, dataset size, and compute
- Kaplan's recommendation: if compute budget is fixed, scale *model size* faster than data
- Why this led to GPT-3 (175B params, relatively little data) — and why this was later shown to be wrong

---

### 4.2 — GPT-3: Scale and In-Context Learning

**Paper:** Language Models are Few-Shot Learners (GPT-3)
**Authors:** Brown et al. (OpenAI) · **Year:** 2020 · **Difficulty:** ★★★☆☆
**Link:** [arXiv:2005.14165](https://arxiv.org/abs/2005.14165)

> ✅ **Verified:** GPT-3 has 175 billion parameters — 10× larger than any prior non-sparse language model (prior leader: Turing NLG at 17B). Scaling greatly improves task-agnostic few-shot performance, sometimes reaching competitiveness with fine-tuned state-of-the-art. In-context learning requires **no weight updates**: tasks and demonstrations are specified purely via text at inference time.

**Why it matters:** GPT-3 introduced *in-context learning* — giving the model examples of a task in the prompt and having it continue the pattern, with no gradient updates. This was a fundamental shift: instead of fine-tuning for each task, you prompt-engineer. The entire prompt engineering and LLM application ecosystem descended from this discovery.

**What to understand:**
- What is zero-shot vs one-shot vs few-shot prompting?
- Why does in-context learning work? (No consensus — gradient descent via attention is one theory)
- What are the failure modes of GPT-3 vs fine-tuned models?
- What is prompt sensitivity and why does it matter for evaluations?

---

### 4.3 — Chinchilla: The Scaling Laws Revisited

**Paper:** Training Compute-Optimal Large Language Models (Chinchilla)
**Authors:** Hoffmann et al. (DeepMind) · **Year:** 2022 · **Difficulty:** ★★★☆☆
**Link:** [arXiv:2203.15556](https://arxiv.org/abs/2203.15556)

> ✅ **Verified:** For compute-optimal training, model size and training tokens should scale **equally** — for every doubling of model size, training tokens should also double, yielding ~20 tokens per parameter. Existing large models (GPT-3, Gopher, Jurassic-1, Megatron-Turing NLG) were **significantly undertrained** because prior scaling work focused on increasing model size while keeping training data constant.

**Why it matters:** Chinchilla invalidated the design of almost every large model trained before 2022. Kaplan's laws said scale the model; Chinchilla says scale model *and data equally*. A 70B model trained on 1.4T tokens beats a 280B model trained on 300B tokens. Every LLM trained after 2022 — LLaMA, Mistral, Falcon, Phi — uses Chinchilla-style training.

**What to understand:**
- The three estimation approaches in the paper and why one has since been questioned
- Why the ~20 tokens/parameter is compute-optimal for *training* but not necessarily for *inference*
- Why LLaMA deliberately violates Chinchilla compute-optimality: they train a smaller model on *more* tokens to reduce inference cost

**Questions to sit with:**
- If you have a fixed compute budget, Chinchilla says split it equally between model size and data. But if you're deploying at scale, you want inference to be cheap — how should you trade off training cost vs inference cost?
- Why do Chinchilla's scaling laws apply specifically to *next-token prediction loss* — and why might they not generalize to capabilities like coding or reasoning?

---

## Stage 5 — Large Language Models

> **The core question:** What does the current frontier of open and closed LLMs look like, and what architectural choices distinguish them?

---

### 5.1 — The GPT-4 / Closed Model Era

| Model | Org | Year | Paper / Report | Notes |
|---|---|---|---|---|
| GPT-4 | OpenAI | 2023 | [Technical Report](https://arxiv.org/abs/2303.08774) | No architectural details released |
| PaLM: Scaling Language Modeling with Pathways | Google | 2022 | [arXiv](https://arxiv.org/abs/2204.02311) | 540B params, Pathways system |
| PaLM 2 | Google | 2023 | [Technical Report](https://arxiv.org/abs/2305.10403) | Compute-efficient, multilingual |
| Gemini 1.0 | Google DeepMind | 2023 | [arXiv](https://arxiv.org/abs/2312.11805) | Natively multimodal |
| Claude 1/2/3 | Anthropic | 2023– | [Model Card](https://www.anthropic.com/news/claude-3-family) | Constitutional AI trained |

---

### 5.2 — The Open-Weight Era: LLaMA & Descendants

**Paper:** LLaMA: Open and Efficient Foundation Language Models
**Authors:** Touvron et al. (Meta AI) · **Year:** 2023 · **Difficulty:** ★★★☆☆
**Link:** [arXiv:2302.13971](https://arxiv.org/abs/2302.13971)

> ✅ **Verified:** LLaMA-13B outperforms GPT-3 (175B) on most benchmarks despite being over 10× smaller. LLaMA-65B is competitive with Chinchilla-70B and PaLM-540B — outperforming PaLM-540B on most commonsense reasoning benchmarks.

**Why it matters:** LLaMA changed the field. Before it, capable models were locked behind APIs. LLaMA demonstrated that with Chinchilla-style data scaling (training smaller models on *more* tokens), you could get GPT-3-level performance at 13B parameters. The entire open-source LLM ecosystem — Alpaca, Vicuna, WizardLM, Mistral, Gemma, Phi — descends from LLaMA.

**Architectural choices to understand:**
- Pre-normalization (RMSNorm before attention, not after) vs. post-normalization
- SwiGLU activation in FFN instead of ReLU
- Rotary positional embeddings (RoPE) instead of learned absolute positional encodings
- Why LLaMA trains longer than Chinchilla-optimal: inference efficiency at deployment

---

### 5.3 — Efficient Architecture Innovations

**Key papers:**
| Paper | Authors | Year | Link | Notes |
|---|---|---|---|---|
| **Mistral 7B** | Jiang et al. (Mistral AI) | 2023 | [arXiv:2310.06825](https://arxiv.org/abs/2310.06825) | GQA + SWA |
| **LLaMA 2** | Touvron et al. | 2023 | [arXiv](https://arxiv.org/abs/2307.09288) | GQA for large variants |
| **Grouped Query Attention (GQA)** | Ainslie et al. | 2023 | [arXiv](https://arxiv.org/abs/2305.13245) | Reduces KV cache size |
| **Multi-Query Attention (MQA)** | Shazeer | 2019 | [arXiv](https://arxiv.org/abs/1911.02150) | Single KV head — used in Falcon |
| **Mixtral 8×7B** | Mistral AI | 2024 | [arXiv](https://arxiv.org/abs/2401.04088) | Mixture of Experts (MoE) |
| **Phi-2 / Phi-3** | Microsoft | 2023/24 | [arXiv](https://arxiv.org/abs/2309.05463) | Small models trained on high-quality synthetic data |

> ✅ **Verified (Mistral 7B):** Uses Sliding Window Attention with window size W=4,096 (theoretical span: 32×4,096 = ~131K tokens) and Grouped Query Attention with 8 KV heads vs. 32 query heads. GQA significantly accelerates inference and reduces KV cache memory, enabling higher batch sizes and throughput.

**Why GQA matters practically:** In standard multi-head attention, every query head has its own K and V projections. GQA groups multiple query heads to share a single K/V pair. This dramatically reduces the KV cache size at inference time — the bottleneck for long-context LLM serving. LLaMA 2 70B, Mistral 7B, Gemma, and almost every modern LLM uses GQA.

---

## Stage 6 — Training at Scale

> **The core question:** A 70B model has 70 billion floating-point weights. A single A100 GPU has 80GB of memory. How do you train it?

---

### 6.1 — The Memory Problem

Before understanding parallelism strategies, understand the memory breakdown:
- **Model weights:** 70B params × 2 bytes (fp16) = 140GB
- **Optimizer states (Adam):** 2× model size = 280GB  
- **Gradients:** 1× model size = 140GB
- **Activations:** variable, but large
- **Total:** ~700GB+ for a 70B model — a single A100 holds 80GB

You need multiple GPUs, and you need to partition this memory across them.

---

### 6.2 — Data Parallelism & ZeRO

**Key papers:**
| Paper | Authors | Year | Link | Notes |
|---|---|---|---|---|
| **ZeRO: Memory Optimizations Toward Training Trillion Parameter Models** | Rajbhandari et al. (Microsoft) | 2019 | [arXiv:1910.02054](https://arxiv.org/abs/1910.02054) | Partitions optimizer states, gradients, params |
| **ZeRO-Offload** | Ren et al. | 2021 | [arXiv](https://arxiv.org/abs/2101.06840) | Offloads to CPU memory |
| **ZeRO-Infinity** | Rajbhandari et al. | 2021 | [arXiv](https://arxiv.org/abs/2104.07857) | Offloads to NVMe storage |
| **PyTorch FSDP** | Zhao et al. | 2023 | [arXiv](https://arxiv.org/abs/2304.11277) | ZeRO-3 equivalent in PyTorch |

**Why ZeRO matters:** Standard data parallelism replicates the full model on every GPU — memory doesn't scale. ZeRO (Zero Redundancy Optimizer) partitions the three main memory consumers across GPUs:
- **ZeRO-1:** Partition optimizer states (4× memory reduction)
- **ZeRO-2:** + Partition gradients (8× reduction)
- **ZeRO-3:** + Partition parameters (memory scales with GPU count)

**What to understand:**
- What is the communication overhead of ZeRO-3 vs ZeRO-1?
- When would you choose FSDP (ZeRO-3) over model parallelism?
- What is gradient checkpointing and why does it trade compute for memory?

---

### 6.3 — Model Parallelism

**Key papers:**
| Paper | Authors | Year | Link | Notes |
|---|---|---|---|---|
| **Megatron-LM: Training Multi-Billion Parameter Language Models Using Model Parallelism** | Shoeybi et al. (NVIDIA) | 2019 | [arXiv:1909.08053](https://arxiv.org/abs/1909.08053) | Tensor parallelism |
| **GPipe: Efficient Training of Giant Neural Networks Using Pipeline Parallelism** | Huang et al. | 2018 | [arXiv](https://arxiv.org/abs/1811.06965) | Pipeline parallelism |
| **PipeDream: Fast and Efficient Pipeline Parallel DNN Training** | Narayanan et al. | 2019 | [arXiv](https://arxiv.org/abs/1806.03377) | Async pipeline |
| **Efficient Large-Scale Language Model Training on GPU Clusters Using Megatron-LM** | Narayanan et al. | 2021 | [arXiv](https://arxiv.org/abs/2104.04473) | 3D parallelism: DP + TP + PP |

**The three parallelism strategies:**
- **Data Parallelism:** same model, different data batches. Gradients all-reduced across GPUs.
- **Tensor Parallelism (Megatron):** split individual matrix multiplications across GPUs within a layer
- **Pipeline Parallelism:** split layers across GPUs (GPU 1 runs layers 1–12, GPU 2 runs layers 13–24, etc.)

**What to understand:**
- Why does tensor parallelism require all-reduce after every attention and FFN layer?
- What is the pipeline bubble in pipeline parallelism and how does micro-batching reduce it?
- Why does GPT-3 / LLaMA-65B / GPT-4 training likely use all three (3D parallelism)?

---

### 6.4 — Mixed Precision & Efficiency

**Key papers:**
| Paper | Authors | Year | Link | Notes |
|---|---|---|---|---|
| Mixed Precision Training | Micikevicius et al. | 2017 | [arXiv](https://arxiv.org/abs/1710.03740) | fp16 forward/backward, fp32 master weights |
| Reducing Activation Recomputation in Large Transformer Models (Selective Recomputation) | Korthikanti et al. | 2022 | [arXiv](https://arxiv.org/abs/2205.05198) | |
| **FlashAttention** | Dao et al. | 2022 | [arXiv](https://arxiv.org/abs/2205.14135) | IO-aware, tiled attention — 2–4× faster |

**Why mixed precision matters:** Training in fp32 doubles memory vs fp16. Mixed precision trains in fp16 (halving memory, doubling throughput on Tensor Cores) while keeping fp32 master weights for optimizer stability. This is standard in virtually all large model training.

---

## Stage 7 — Alignment & Instruction Tuning

> **The core question:** A pre-trained LLM predicts next tokens. How do you make it helpful, harmless, and honest?

---

### 7.1 — Instruction Fine-tuning

**Key papers:**
| Paper | Authors | Year | Link | Notes |
|---|---|---|---|---|
| Finetuned Language Models are Zero-Shot Learners (FLAN) | Wei et al. (Google) | 2021 | [arXiv](https://arxiv.org/abs/2109.01652) | Multi-task instruction fine-tuning improves zero-shot |
| Super-NaturalInstructions | Wang et al. | 2022 | [arXiv](https://arxiv.org/abs/2204.07705) | 1,600+ NLP tasks for instruction tuning |
| Scaling Instruction-Finetuned Language Models (FLAN-T5/PaLM) | Chung et al. | 2022 | [arXiv](https://arxiv.org/abs/2210.11416) | More tasks + more scale = better zero-shot |

**Why it matters:** A pre-trained model is good at completing text, not at answering questions helpfully. Instruction fine-tuning trains the model on thousands of (instruction, response) pairs. FLAN showed that a model fine-tuned on 1,836 NLP tasks improves zero-shot performance dramatically on *unseen* tasks — the model generalizes the concept of "following instructions."

---

### 7.2 — RLHF & InstructGPT

**Paper:** Training language models to follow instructions with human feedback (InstructGPT)
**Authors:** Ouyang et al. (OpenAI) · **Year:** 2022 · **Difficulty:** ★★★☆☆
**Link:** [arXiv:2203.02155](https://arxiv.org/abs/2203.02155)

**Why it matters:** InstructGPT introduced RLHF to LLMs — the technique behind ChatGPT, Claude, and every modern assistant model. The three-step process:
1. **Supervised Fine-tuning (SFT):** fine-tune on human-written demonstrations
2. **Reward Model (RM):** train a model to predict human preference scores on pairs of outputs
3. **PPO:** optimize the SFT model against the reward model using Proximal Policy Optimization

**What to understand:**
- Why does RLHF produce better outputs than pure SFT, even with the same data?
- What is reward hacking and why is it a fundamental problem in RLHF?
- What is KL divergence penalty in PPO-RLHF and why is it needed? (Prevents the model from drifting too far from the SFT baseline)

**Questions to sit with:**
- The reward model is trained on human preference pairs. What happens when humans have inconsistent or biased preferences?
- RLHF with PPO requires running four models simultaneously (policy, reference policy, reward model, value model). Why is this expensive?

---

### 7.3 — Constitutional AI

**Paper:** Constitutional AI: Harmlessness from AI Feedback
**Authors:** Bai et al. (Anthropic) · **Year:** 2022 · **Difficulty:** ★★★☆☆
**Link:** [arXiv:2212.08073](https://arxiv.org/abs/2212.08073)

**Why it matters:** Constitutional AI (CAI) replaces human preference labelers with AI feedback guided by a set of written principles (a "constitution"). The model critiques and revises its own outputs according to the constitution (RLAIF — RL from AI Feedback). This dramatically reduces the human labeling burden and makes the alignment process more transparent and scalable.

---

### 7.4 — DPO: Alignment Without RL

**Paper:** Direct Preference Optimization: Your Language Model is Secretly a Reward Model
**Authors:** Rafailov et al. · **Year:** 2023 · **Difficulty:** ★★★★☆
**Link:** [arXiv:2305.18290](https://arxiv.org/abs/2305.18290)

**Why it matters:** RLHF with PPO is complex, expensive, and unstable. DPO shows you can skip the reward model entirely and directly optimize the policy on preference data using a classification objective. DPO is now the dominant alignment technique in open-source models (LLaMA-2 Chat, Zephyr, Mistral-Instruct). It's simpler, more stable, and achieves comparable or better results than PPO on many benchmarks.

**What to understand:**
- DPO's key insight: the optimal RLHF policy has a closed-form solution in terms of the reference policy. You can rearrange this into a loss function that doesn't require an explicit reward model.
- What is `(chosen, rejected)` preference data and where does it come from?
- When might PPO outperform DPO? (Tasks requiring exploration; non-binary preferences)

---

### 7.5 — Other Alignment Papers

| Paper | Authors | Year | Link | Notes |
|---|---|---|---|---|
| Proximal Policy Optimization (PPO) | Schulman et al. (OpenAI) | 2017 | [arXiv](https://arxiv.org/abs/1707.06347) | The RL algorithm behind InstructGPT |
| RLHF original (summarization) | Stiennon et al. | 2020 | [arXiv](https://arxiv.org/abs/2009.01325) | RLHF applied to summarization |
| Reward Model Ensembles | — | — | — | Mitigates reward hacking |
| RLAIF: Scaling Reinforcement Learning from Human Feedback with AI Feedback | Lee et al. | 2023 | [arXiv](https://arxiv.org/abs/2309.00267) | AI feedback scales better than human |

---

## Stage 8 — Inference & Serving

> **The core question:** Training happens once. Inference happens billions of times. How do you make it fast and cheap?

---

### 8.1 — The KV Cache

**Why it matters:** During autoregressive generation, every token attends to all previous tokens. Without caching, every step recomputes attention for the entire context. The KV cache stores the Key and Value tensors from previous steps and reuses them — reducing each generation step from O(n²) to O(n). This is the single most important optimization for LLM inference.

**What to understand:**
- How much memory does a KV cache consume? For LLaMA-65B with 4K context and fp16: `2 × layers × heads × d_head × seq_len × 2 bytes` ≈ several GB per request
- Why does GQA (Stage 5.3) directly reduce KV cache size?
- What is prefix caching (prompt caching)? Why is it useful for RAG and long system prompts?

---

### 8.2 — Continuous Batching & vLLM

**Key papers:**
| Paper | Authors | Year | Link | Notes |
|---|---|---|---|---|
| **Efficient Memory Management for Large Language Model Serving with PagedAttention** | Kwon et al. (Berkeley) | 2023 | [arXiv](https://arxiv.org/abs/2309.06180) | The vLLM paper |
| Orca: A Distributed Serving System for Transformer-Based Generative Models | Yu et al. | 2022 | [OSDI](https://www.usenix.org/conference/osdi22/presentation/yu) | Continuous batching |

**Why vLLM/PagedAttention matters:** Traditional KV cache management allocates a contiguous memory block per request — but you don't know the output length in advance. PagedAttention borrows the virtual memory paging concept from operating systems: divide KV cache into pages, allocate non-contiguous blocks on demand. This eliminates nearly all KV cache memory fragmentation. Combined with continuous batching (processing new requests as soon as a sequence finishes), vLLM achieves 24× higher throughput than HuggingFace Transformers.

**What to understand:**
- What is static batching vs. continuous batching?
- Why does fragmentation waste so much memory in naive KV cache allocation?
- How does PagedAttention's attention kernel differ from standard attention?

---

### 8.3 — Speculative Decoding

**Key papers:**
| Paper | Authors | Year | Link | Notes |
|---|---|---|---|---|
| Fast Inference from Transformers via Speculative Decoding | Leviathan et al. (Google) | 2022 | [arXiv](https://arxiv.org/abs/2211.17192) | Small draft model + large verifier |
| Medusa: Simple LLM Inference Acceleration with Multiple Decoding Heads | Cai et al. | 2024 | [arXiv](https://arxiv.org/abs/2401.10774) | Multiple prediction heads — no draft model |

**Why it matters:** LLM decoding is memory-bandwidth-bound, not compute-bound — the GPU spends most of its time moving model weights from HBM. Adding more compute doesn't help. Speculative decoding uses a small, fast *draft model* to propose multiple tokens at once, then the large *target model* verifies them in a single parallel forward pass. If the draft was right, you get multiple tokens for the price of one step. Expected speedup: 2–4× for free.

**What to understand:**
- Why is autoregressive decoding memory-bandwidth-bound?
- What is the acceptance criterion for draft tokens? (The target model's probability for that token divided by the draft model's probability — rejection sampling)
- Why does speculative decoding produce *identical* outputs to standard decoding? (Not an approximation)

---

### 8.4 — Quantization

**Key papers:**
| Paper | Authors | Year | Link | Notes |
|---|---|---|---|---|
| GPTQ: Accurate Post-Training Quantization for Generative Pre-trained Transformers | Frantar et al. | 2022 | [arXiv](https://arxiv.org/abs/2210.17323) | 4-bit weight-only quantization |
| AWQ: Activation-Aware Weight Quantization for LLM Compression and Acceleration | Lin et al. | 2023 | [arXiv](https://arxiv.org/abs/2306.00978) | Preserves salient weights |
| GGUF / llama.cpp | Gerganov | 2023 | [GitHub](https://github.com/ggerganov/llama.cpp) | CPU inference for quantized models |
| SmoothQuant | Xiao et al. | 2022 | [arXiv](https://arxiv.org/abs/2211.10438) | W8A8 quantization |

**Quantization levels:**
- **fp16 / bf16:** standard training and inference (2 bytes/param)
- **int8 (W8A8):** weights and activations in 8-bit (1 byte/param)
- **int4 (W4A16):** weights in 4-bit, activations in fp16 — typical for GPTQ/AWQ (0.5 bytes/param)
- **int4 grouped (Q4_K_M):** GGUF format for CPU inference

**What to understand:**
- Post-training quantization (PTQ) vs. quantization-aware training (QAT)
- Why quantizing weights is easier than quantizing activations
- How GPTQ uses second-order gradient information to minimize quantization error layer by layer

---

## Stage 9 — AI Engineering in Production

> **The core question:** You have a capable LLM. How do you build reliable, evaluable, observable production systems with it?

---

### 9.1 — Fine-tuning: LoRA & QLoRA

**Key papers:**
| Paper | Authors | Year | Link | Notes |
|---|---|---|---|---|
| **LoRA: Low-Rank Adaptation of Large Language Models** | Hu et al. (Microsoft) | 2021 | [arXiv:2106.09685](https://arxiv.org/abs/2106.09685) | Freeze base model, train low-rank adapters |
| **QLoRA: Efficient Finetuning of Quantized LLMs** | Dettmers et al. | 2023 | [arXiv](https://arxiv.org/abs/2305.14314) | LoRA on 4-bit quantized models |
| Prefix Tuning | Li & Liang | 2021 | [arXiv](https://arxiv.org/abs/2101.00190) | Prepend trainable prefix tokens |
| Prompt Tuning | Lester et al. | 2021 | [arXiv](https://arxiv.org/abs/2104.08691) | Soft prompts |

**Why LoRA changed fine-tuning:** Full fine-tuning a 70B model requires 70B× gradient + optimizer state memory = infeasible on consumer hardware. LoRA freezes the base model and adds low-rank decomposition matrices (`A × B` where rank `r << d`) to selected weight matrices. Only `A` and `B` are trained — typically < 1% of original parameters. Quality is nearly identical to full fine-tuning on most tasks.

**QLoRA** takes it further: 4-bit quantize the frozen base model (reducing memory 8×), then LoRA fine-tune in fp16. A 65B model fine-tuned on a single 48GB GPU. This democratized fine-tuning of large models.

**What to understand:**
- Why does the low-rank decomposition work? What does it assume about the structure of fine-tuning updates?
- What is rank `r` in LoRA and how do you choose it?
- What is bfloat16 and why does QLoRA use it for the LoRA adapters (not the quantized base)?

---

### 9.2 — RAG: Retrieval-Augmented Generation

**Key papers:**
| Paper | Authors | Year | Link | Notes |
|---|---|---|---|---|
| Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks | Lewis et al. (Meta AI) | 2020 | [arXiv](https://arxiv.org/abs/2005.11401) | The original RAG paper |
| REALM: Retrieval-Enhanced Language Model Pre-training | Guu et al. | 2020 | [arXiv](https://arxiv.org/abs/2002.08909) | Retrieval during pre-training |
| Self-RAG | Asai et al. | 2023 | [arXiv](https://arxiv.org/abs/2310.11511) | LLM decides when to retrieve |
| Dense Passage Retrieval (DPR) | Karpukhin et al. | 2020 | [arXiv](https://arxiv.org/abs/2004.04906) | Dense retrieval with bi-encoders |

**Why RAG matters:** LLMs have stale knowledge cutoffs and hallucinate facts. RAG retrieves relevant documents at inference time and feeds them to the model as context. The LLM becomes a *reader* on top of a *retriever*. This solves knowledge staleness without retraining and reduces (though doesn't eliminate) hallucination.

**RAG stack to understand:**
1. **Chunking:** how do you split documents?
2. **Embedding:** how do you vectorize chunks? (OpenAI ada-002, E5, BGE, etc.)
3. **Vector database:** Pinecone, Weaviate, Qdrant, pgvector, FAISS — what are the tradeoffs?
4. **Retrieval:** dense (embedding similarity) vs. sparse (BM25) vs. hybrid
5. **Generation:** how do you construct the prompt? How do you handle retrieved context that's wrong or irrelevant?

---

### 9.3 — Vector Databases & Embeddings

| System | Notes |
|---|---|
| FAISS (Facebook AI Similarity Search) | [GitHub](https://github.com/facebookresearch/faiss) — local, battle-tested |
| Pinecone | Managed vector DB, serverless option |
| Weaviate | Open-source, hybrid search |
| Qdrant | Open-source, Rust-based, fast |
| Chroma | Lightweight, developer-friendly |
| pgvector | PostgreSQL extension — great if you're already on Postgres |
| Milvus | Cloud-native, highly scalable |

**Key paper:** Approximate Nearest Neighbor (ANN) algorithms: HNSW (Hierarchical Navigable Small World Graphs) — [arXiv](https://arxiv.org/abs/1603.09320) — the indexing algorithm behind most modern vector DBs.

---

### 9.4 — LLM Evaluation

**Key papers & resources:**
| Resource | Notes |
|---|---|
| HELM: Holistic Evaluation of Language Models | [arXiv](https://arxiv.org/abs/2211.09110) — Stanford's comprehensive benchmark suite |
| BIG-Bench: Beyond the Imitation Game | [arXiv](https://arxiv.org/abs/2206.04615) — 204 tasks, crowdsourced |
| MMLU: Measuring Massive Multitask Language Understanding | [arXiv](https://arxiv.org/abs/2009.03300) — 57-subject multiple choice |
| MT-Bench / Chatbot Arena | [arXiv](https://arxiv.org/abs/2306.05685) — LLM-as-judge; human preference arena |
| Evals (OpenAI) | [GitHub](https://github.com/openai/evals) — framework for LLM evaluation |
| LangSmith / Braintrust / Weave | Evaluation platforms for production LLM apps |

**The fundamental evaluation problem:** LLMs are general — no single benchmark captures capability. LLM-as-judge (using a powerful model to evaluate outputs) is increasingly standard but introduces its own biases. Chatbot Arena's Elo-based human preference ranking is currently the most reliable signal for overall quality.

---

### 9.5 — Agents & Tool Use

**Key papers:**
| Paper | Authors | Year | Link | Notes |
|---|---|---|---|---|
| ReAct: Synergizing Reasoning and Acting in Language Models | Yao et al. | 2022 | [arXiv](https://arxiv.org/abs/2210.03629) | Interleave reasoning traces and actions |
| Toolformer: Language Models Can Teach Themselves to Use Tools | Schick et al. | 2023 | [arXiv](https://arxiv.org/abs/2302.04761) | Self-supervised tool use learning |
| HuggingGPT / JARVIS | Shen et al. | 2023 | [arXiv](https://arxiv.org/abs/2303.17580) | LLM as orchestrator of specialized models |
| OpenAI Function Calling | OpenAI | 2023 | [Docs](https://platform.openai.com/docs/guides/function-calling) | Structured tool call format |
| Tree of Thoughts | Yao et al. | 2023 | [arXiv](https://arxiv.org/abs/2305.10601) | Deliberate search over thought trees |
| Chain-of-Thought Prompting | Wei et al. | 2022 | [arXiv](https://arxiv.org/abs/2201.11903) | "Think step by step" — CoT |

**Why CoT matters:** "Let's think step by step" dramatically improves LLM performance on reasoning tasks. Wei et al. showed that chain-of-thought reasoning is an emergent property of scale — it doesn't appear in smaller models. ReAct (reason + act) interleaves reasoning traces with tool calls, enabling agents to use search, code execution, and APIs.

---

### 9.6 — Prompt Engineering

**Essential reads:**
| Resource | Notes |
|---|---|
| Prompt Engineering Guide | [promptingguide.ai](https://www.promptingguide.ai/) — comprehensive |
| Chain-of-Thought Prompting (Wei et al.) | [arXiv](https://arxiv.org/abs/2201.11903) |
| Self-Consistency Improves Chain of Thought | [arXiv](https://arxiv.org/abs/2203.11171) — sample multiple reasoning paths, vote |
| Large Language Models as Optimizers (OPRO) | [arXiv](https://arxiv.org/abs/2309.03409) — LLM-written prompts |
| Anthropic Prompt Engineering Docs | [Docs](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview) |

---

## Practitioner Blogs & Essential Reading

### The Must-Read Blogs

| Blog | Author | Why it matters |
|---|---|---|
| [**Lilian Weng's Blog**](https://lilianweng.github.io/) | Lilian Weng (OpenAI) | The single best technical blog in AI. Deep, accurate, well-cited posts on LLM agents, RLHF, diffusion, attention, and everything in between. Start with her posts on LLMs, RLHF, and AI agents. |
| [**Andrej Karpathy's Blog**](https://karpathy.github.io/) | Andrej Karpathy | "The Unreasonable Effectiveness of RNNs," "A Recipe for Training Neural Networks," makemore/nanoGPT. His explainers are the clearest in the field. |
| [**Sebastian Raschka's Substack**](https://magazine.sebastianraschka.com/) | Sebastian Raschka | Rigorous, code-first deep dives into LLMs, fine-tuning, training tricks. His "Understanding LLMs" reading list is a curriculum in itself. |
| [**Chip Huyen's Blog**](https://huyenchip.com/) | Chip Huyen | AI engineering in production — RAG, serving, evals, MLOps. Bridges research and deployment. |
| [**Eugene Yan's Blog**](https://eugeneyan.com/) | Eugene Yan (Amazon) | Applied ML, LLM system design, evaluation, retrieval. Practical and deeply researched. |

---

### Company Engineering Blogs

#### OpenAI
| Post | Notes |
|---|---|
| [GPT-4 Technical Report](https://arxiv.org/abs/2303.08774) | What they revealed |
| [Introducing ChatGPT](https://openai.com/blog/chatgpt) | The RLHF product |
| [Scaling Laws for Neural Language Models](https://arxiv.org/abs/2001.08361) | Kaplan et al. |
| [OpenAI Research Blog](https://openai.com/research) | |

#### Anthropic
| Post | Notes |
|---|---|
| [Constitutional AI](https://www.anthropic.com/research/constitutional-ai-harmlessness-from-ai-feedback) | RLAIF |
| [Core Views on AI Safety](https://www.anthropic.com/news/core-views-on-ai-safety) | Interpretability + alignment philosophy |
| [Scaling Laws and Interpretability](https://www.anthropic.com/research) | Circuits, features, monosemanticity |
| [Anthropic Research Blog](https://www.anthropic.com/research) | |

#### Google DeepMind
| Post | Notes |
|---|---|
| [Gemini Technical Report](https://arxiv.org/abs/2312.11805) | Natively multimodal architecture |
| [PaLM Paper](https://arxiv.org/abs/2204.02311) | Pathways, 540B, chain-of-thought emergence |
| [Google AI Blog](https://ai.googleblog.com/) | |

#### Meta AI
| Post | Notes |
|---|---|
| [LLaMA Paper](https://arxiv.org/abs/2302.13971) | Open weights, efficiency focus |
| [LLaMA 2 Paper](https://arxiv.org/abs/2307.09288) | RLHF + safety fine-tuning |
| [Meta AI Research](https://ai.meta.com/research/) | |

#### Hugging Face
| Post | Notes |
|---|---|
| [RLHF Blog Post](https://huggingface.co/blog/rlhf) | Best practical explainer of RLHF |
| [LoRA Blog Post](https://huggingface.co/blog/lora) | How to use LoRA in practice |
| [TRL Documentation](https://huggingface.co/docs/trl) | SFT, RLHF, DPO in code |
| [Hugging Face Blog](https://huggingface.co/blog) | |

#### Mistral AI
| Post | Notes |
|---|---|
| [Mistral 7B Blog](https://mistral.ai/news/announcing-mistral-7b/) | GQA + SWA explained |
| [Mixtral 8×7B Blog](https://mistral.ai/news/mixtral-of-experts/) | MoE explained |

#### Microsoft (DeepSpeed / Phi)
| Post | Notes |
|---|---|
| [DeepSpeed Blog](https://www.microsoft.com/en-us/research/blog/deepspeed-extreme-scale-model-training-for-everyone/) | ZeRO explained accessibly |
| [Phi-2 Blog](https://www.microsoft.com/en-us/research/blog/phi-2-the-surprising-power-of-small-language-models/) | Quality data > scale |

#### Stability AI
| Post | Notes |
|---|---|
| [Stable Diffusion Announcement](https://stability.ai/news/stable-diffusion-announcement) | |
| [Stability AI Research](https://stability.ai/research) | |

---

## Full Paper Reference

### Foundations
| Paper | Authors | Year | Link |
|---|---|---|---|
| Learning representations by back-propagating errors | Rumelhart, Hinton, Williams | 1986 | [Nature](https://www.nature.com/articles/323533a0) |
| Adam: A Method for Stochastic Optimization | Kingma & Ba | 2014 | [arXiv](https://arxiv.org/abs/1412.6980) |
| Decoupled Weight Decay Regularization (AdamW) | Loshchilov & Hutter | 2017 | [arXiv](https://arxiv.org/abs/1711.05101) |
| Dropout | Srivastava et al. | 2014 | [JMLR](https://jmlr.org/papers/v15/srivastava14a.html) |
| Deep Double Descent | Nakkiran et al. | 2019 | [arXiv](https://arxiv.org/abs/1912.02292) |
| Representation Learning | Bengio, Courville, Vincent | 2013 | [arXiv](https://arxiv.org/abs/1206.5538) |

### Deep Learning Architectures
| Paper | Authors | Year | Link |
|---|---|---|---|
| LeNet | LeCun et al. | 1998 | [IEEE](http://yann.lecun.com/exdb/publis/pdf/lecun-01a.pdf) |
| AlexNet | Krizhevsky, Sutskever, Hinton | 2012 | [NeurIPS](https://papers.nips.cc/paper/2012/hash/c399862d3b9d6b76c8436e924a68c45b-Abstract.html) |
| VGGNet | Simonyan & Zisserman | 2014 | [arXiv](https://arxiv.org/abs/1409.1556) |
| Batch Normalization | Ioffe & Szegedy | 2015 | [arXiv](https://arxiv.org/abs/1502.03167) |
| ResNet | He et al. | 2015 | [arXiv](https://arxiv.org/abs/1512.03385) |
| Layer Normalization | Ba et al. | 2016 | [arXiv](https://arxiv.org/abs/1607.06450) |
| LSTM | Hochreiter & Schmidhuber | 1997 | [Neural Computation](https://direct.mit.edu/neco/article/9/8/1735/6109/Long-Short-Term-Memory) |
| Seq2Seq | Sutskever, Vinyals, Le | 2014 | [arXiv](https://arxiv.org/abs/1409.3215) |
| Bahdanau Attention | Bahdanau et al. | 2014 | [arXiv](https://arxiv.org/abs/1409.0473) |

### Transformers & Pre-training
| Paper | Authors | Year | Link |
|---|---|---|---|
| Attention Is All You Need | Vaswani et al. | 2017 | [arXiv](https://arxiv.org/abs/1706.03762) |
| GPT-1 | Radford et al. | 2018 | [OpenAI](https://openai.com/research/language-unsupervised) |
| BERT | Devlin et al. | 2018 | [arXiv](https://arxiv.org/abs/1810.04805) |
| GPT-2 | Radford et al. | 2019 | [OpenAI](https://openai.com/research/better-language-models) |
| Transformer-XL | Dai et al. | 2019 | [arXiv](https://arxiv.org/abs/1901.02860) |
| GPT-3 | Brown et al. | 2020 | [arXiv](https://arxiv.org/abs/2005.14165) |
| Scaling Laws (Kaplan) | Kaplan et al. | 2020 | [arXiv](https://arxiv.org/abs/2001.08361) |
| RoPE | Su et al. | 2021 | [arXiv](https://arxiv.org/abs/2104.09864) |
| FlashAttention | Dao et al. | 2022 | [arXiv](https://arxiv.org/abs/2205.14135) |
| Chinchilla | Hoffmann et al. | 2022 | [arXiv](https://arxiv.org/abs/2203.15556) |
| FlashAttention-2 | Dao | 2023 | [arXiv](https://arxiv.org/abs/2307.08691) |

### Large Language Models
| Paper | Authors | Year | Link |
|---|---|---|---|
| PaLM | Chowdhery et al. | 2022 | [arXiv](https://arxiv.org/abs/2204.02311) |
| GPT-4 Technical Report | OpenAI | 2023 | [arXiv](https://arxiv.org/abs/2303.08774) |
| LLaMA | Touvron et al. | 2023 | [arXiv](https://arxiv.org/abs/2302.13971) |
| LLaMA 2 | Touvron et al. | 2023 | [arXiv](https://arxiv.org/abs/2307.09288) |
| Mistral 7B | Jiang et al. | 2023 | [arXiv](https://arxiv.org/abs/2310.06825) |
| Mixtral 8×7B | Mistral AI | 2024 | [arXiv](https://arxiv.org/abs/2401.04088) |
| Gemini | Google DeepMind | 2023 | [arXiv](https://arxiv.org/abs/2312.11805) |
| Grouped-Query Attention | Ainslie et al. | 2023 | [arXiv](https://arxiv.org/abs/2305.13245) |

### Training at Scale
| Paper | Authors | Year | Link |
|---|---|---|---|
| Mixed Precision Training | Micikevicius et al. | 2017 | [arXiv](https://arxiv.org/abs/1710.03740) |
| GPipe | Huang et al. | 2018 | [arXiv](https://arxiv.org/abs/1811.06965) |
| Megatron-LM | Shoeybi et al. | 2019 | [arXiv](https://arxiv.org/abs/1909.08053) |
| ZeRO | Rajbhandari et al. | 2019 | [arXiv](https://arxiv.org/abs/1910.02054) |
| 3D Parallelism (Megatron v2) | Narayanan et al. | 2021 | [arXiv](https://arxiv.org/abs/2104.04473) |
| ZeRO-Infinity | Rajbhandari et al. | 2021 | [arXiv](https://arxiv.org/abs/2104.07857) |
| PyTorch FSDP | Zhao et al. | 2023 | [arXiv](https://arxiv.org/abs/2304.11277) |

### Alignment
| Paper | Authors | Year | Link |
|---|---|---|---|
| FLAN | Wei et al. | 2021 | [arXiv](https://arxiv.org/abs/2109.01652) |
| PPO | Schulman et al. | 2017 | [arXiv](https://arxiv.org/abs/1707.06347) |
| RLHF (summarization) | Stiennon et al. | 2020 | [arXiv](https://arxiv.org/abs/2009.01325) |
| InstructGPT | Ouyang et al. | 2022 | [arXiv](https://arxiv.org/abs/2203.02155) |
| Constitutional AI | Bai et al. | 2022 | [arXiv](https://arxiv.org/abs/2212.08073) |
| RLAIF | Lee et al. | 2023 | [arXiv](https://arxiv.org/abs/2309.00267) |
| DPO | Rafailov et al. | 2023 | [arXiv](https://arxiv.org/abs/2305.18290) |

### Inference & Serving
| Paper | Authors | Year | Link |
|---|---|---|---|
| Speculative Decoding | Leviathan et al. | 2022 | [arXiv](https://arxiv.org/abs/2211.17192) |
| GPTQ | Frantar et al. | 2022 | [arXiv](https://arxiv.org/abs/2210.17323) |
| SmoothQuant | Xiao et al. | 2022 | [arXiv](https://arxiv.org/abs/2211.10438) |
| Orca (continuous batching) | Yu et al. | 2022 | [OSDI](https://www.usenix.org/conference/osdi22/presentation/yu) |
| vLLM / PagedAttention | Kwon et al. | 2023 | [arXiv](https://arxiv.org/abs/2309.06180) |
| AWQ | Lin et al. | 2023 | [arXiv](https://arxiv.org/abs/2306.00978) |
| Medusa | Cai et al. | 2024 | [arXiv](https://arxiv.org/abs/2401.10774) |

### Fine-tuning & Production
| Paper | Authors | Year | Link |
|---|---|---|---|
| RAG | Lewis et al. | 2020 | [arXiv](https://arxiv.org/abs/2005.11401) |
| DPR | Karpukhin et al. | 2020 | [arXiv](https://arxiv.org/abs/2004.04906) |
| LoRA | Hu et al. | 2021 | [arXiv](https://arxiv.org/abs/2106.09685) |
| Chain-of-Thought Prompting | Wei et al. | 2022 | [arXiv](https://arxiv.org/abs/2201.11903) |
| HELM | Liang et al. | 2022 | [arXiv](https://arxiv.org/abs/2211.09110) |
| QLoRA | Dettmers et al. | 2023 | [arXiv](https://arxiv.org/abs/2305.14314) |
| ReAct | Yao et al. | 2022 | [arXiv](https://arxiv.org/abs/2210.03629) |
| HNSW (vector indexing) | Malkov & Yashunin | 2016 | [arXiv](https://arxiv.org/abs/1603.09320) |

---

## Concept Dependency Map

```
┌──────────────────────────────────────────────────────────────────────┐
│                          FOUNDATIONS                                 │
│   Backprop ──► SGD/Adam ──► Regularization ──► Representation       │
│      │              │              │              Learning           │
│      └──────────────┴──────────────┘                                 │
│                          │                                           │
└──────────────────────────┼───────────────────────────────────────────┘
                           │
          ┌────────────────┼──────────────────┐
          ▼                ▼                  ▼
   ┌─────────────┐  ┌────────────┐  ┌──────────────────┐
   │    CNNs     │  │  RNNs /    │  │  Normalization   │
   │ (LeNet,     │  │  LSTMs /   │  │  (BatchNorm,     │
   │  AlexNet,   │  │  Seq2Seq + │  │   LayerNorm,     │
   │  VGG,       │  │  Attention │  │   RMSNorm)       │
   │  ResNet)    │  │            │  │                  │
   └──────┬──────┘  └─────┬──────┘  └────────┬─────────┘
          │               │                   │
          └───────────────┴───────────────────┘
                          │
                          ▼
          ┌───────────────────────────────┐
          │        TRANSFORMER            │
          │  Self-Attention + FFN +       │
          │  Residual + LayerNorm +       │
          │  Positional Encoding         │
          └───────────────┬───────────────┘
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
   ┌────────────┐  ┌────────────┐  ┌────────────────┐
   │ BERT       │  │    GPT     │  │  Scaling Laws  │
   │ (encoder,  │  │ (decoder,  │  │  Kaplan →      │
   │  masked    │  │  causal    │  │  Chinchilla    │
   │  LM)       │  │  LM)       │  │                │
   └────────────┘  └─────┬──────┘  └────────────────┘
                         │
          ┌──────────────┼──────────────────┐
          ▼              ▼                  ▼
   ┌────────────┐  ┌──────────────┐  ┌──────────────┐
   │ LLMs       │  │  Training    │  │  Alignment   │
   │ (GPT-3/4,  │  │  at Scale    │  │  (RLHF,      │
   │  LLaMA,    │  │  (ZeRO,      │  │   DPO, CAI,  │
   │  Mistral,  │  │   Megatron,  │  │   PPO)       │
   │  Gemini)   │  │   FSDP,      │  └──────┬───────┘
   └─────┬──────┘  │   MixedPrec) │         │
         │         └──────────────┘         │
         └──────────────────────────────────┘
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
   ┌────────────┐  ┌────────────┐  ┌────────────────┐
   │ Inference  │  │ Fine-tuning│  │  Production    │
   │ (KV Cache, │  │ (LoRA,     │  │  (RAG, Evals,  │
   │  vLLM,     │  │  QLoRA,    │  │   Agents, CoT, │
   │  Spec Dec, │  │  SFT)      │  │   VectorDB)    │
   │  Quant)    │  └────────────┘  └────────────────┘
   └────────────┘
```

---

## Resources & Courses

| Resource | Format | Notes |
|---|---|---|
| [**fast.ai: Practical Deep Learning**](https://course.fast.ai/) | Course | Top-down, code-first. Best starting point for practitioners. |
| [**Andrej Karpathy — Neural Networks: Zero to Hero**](https://karpathy.ai/zero-to-hero.html) | YouTube | Build backprop, makemore, nanoGPT from scratch. The best hands-on course. |
| [**Stanford CS224N: NLP with Deep Learning**](https://web.stanford.edu/class/cs224n/) | Course | Transformers, BERT, LLMs, alignment. Graduate-level. |
| [**Sebastian Raschka — LLM from Scratch**](https://github.com/rasbt/LLMs-from-scratch) | Book + Code | Implement a GPT from scratch in PyTorch |
| [**Chip Huyen — Designing Machine Learning Systems**](https://huyenchip.com/books/) | Book | Production ML systems — training pipelines, deployment, monitoring |
| [**Hugging Face NLP Course**](https://huggingface.co/learn/nlp-course) | Course | Transformers in practice: fine-tuning, tokenizers, pipelines |
| [**Lilian Weng — LLM Powered Agents**](https://lilianweng.github.io/posts/2023-06-23-agent/) | Blog post | The canonical agents explainer |
| [**The Illustrated Transformer**](https://jalammar.github.io/illustrated-transformer/) | Blog post | Best visual explainer of attention |
| [**Awesome LLM**](https://github.com/Hannibal046/Awesome-LLM) | GitHub list | Comprehensive paper list, updated regularly |
| [**annotated_deep_learning_paper_implementations**](https://github.com/labmlai/annotated_deep_learning_paper_implementations) | GitHub | 50+ papers implemented and annotated side-by-side |
| [**ML Papers Explained (DAIR.AI)**](https://github.com/dair-ai/ML-Papers-Explained) | GitHub | One-para summaries of key ML papers |

---

*Research verified via adversarial multi-agent review — 115 agents, 32 primary sources, 158 claims extracted, 18 confirmed at 2/3+ confidence. Last updated: June 2026.*
