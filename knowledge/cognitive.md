# cognitive — a ternary recurrent language model for a bandwidth-bound CPU

**Status:** parked 2026-10-25, deliberately, with the working code archived at
`~/cognitive-2026-10-25.tar.gz` (1.7 MB, the full source and the git history).
Ben's call: keep the idea, clear the project, come back with better hardware.

**One line:** train a language model from scratch, in Rust, on one CPU, with 2-bit
weights — because this machine is bandwidth-bound, so the model that fits is the
model you can train.

---

## 1. The idea

A language model is a map from text to a probability distribution over the next
byte. The quality of a model is bounded by what you can train; what you can
train is bounded by what fits in RAM and how fast the memory moves.

On the machine this was built for — an HP EliteBook 845 G7, AMD Ryzen 5 PRO 4650U
(Zen 2, 6 cores / 12 threads, AVX2 but no AVX-512, no VNNI, no AMX, no ROCm),
7.1 GiB RAM, ~12–16 GB/s of measured memory bandwidth, 8 MB L3, no usable GPU
for training — three facts decide everything:

1. **It is bandwidth-bound, not FLOP-bound.** A batch-1 forward over a
   4096×4096 layer runs at ~19 GMAC/s, not the ~480 the CPU could reach.
2. **RAM is the binding constraint.** A 135M model in fp16 with Adam needs
   2.2 GB; 360M needs 5.8 GB and does not fit. Ternary weights at 2.28
   bytes/parameter fit 360M in 1.6 GB.
3. **Compute is the wall on scale.** 200M params × 1B tokens = 1.2e18 FLOPs;
   this box delivers ~1.5e11 FLOP/s, so 200M on 1B tokens is a half-year no
   matter how good the kernel is. **That goal was wrong for this hardware and
   was dropped.**

So the design follows from the constraint: 2-bit weights (16× less traffic than
fp16), a recurrent architecture (constant memory per token, no KV cache), and
everything else arranged so the bytes that move are the bytes that matter.

## 2. The architecture that was built

A residual stack of ternary minGRU blocks. Per layer:

```
x_{l+1} = x_l + W_out( SubLN( minGRU( TokenShift4( PreRMSNorm(x_l) ), b_z ) ) )
```

- **Ternary weights, 2 bits.** Weights in {-1, 0, +1}, per-row AbsMean scaling,
  packed four to a byte in a striped layout, unpacked with `vpsignb` so a
  projection is integer add/subtract rather than floating-point multiply.
  Forward and backward both verified bit-exact against independent oracles.
- **A fp32 gate bias `b_z`, outside the quantiser.** This turned out to be the
  single most important detail. Without a bias the gate's pre-activation is
  zero-mean, so the gate sits at σ(0) = 0.5 for every channel and the model's
  effective memory is **about two bytes**. Measured on a trained model: channel
  retention p10 0.342 / p50 0.481 / p90 0.597 / max 0.741, and **no channel
  exceeded a three-byte half-life.** The bias is initialised by Chrono
  (retention timescales log-spaced across channels, 2.5–200 bytes) and trained
  with SGD and momentum, not Adam: Adam's m/√v cancels the natural z(1−z) damping
  and drove the long channels into the clamp within 300 steps.
- **The gate's *total* pre-activation is clamped** to ≥ −5, not just `b_z`.
  Clamping `b_z` alone does not prevent saturation — the ternary gate projection
  can produce a strongly negative z_raw, and z_raw + b_z still runs past the
  sigmoid's floor. With the total clamped, no channel can hold a half-life
  longer than ~149 bytes, so the recurrent state can never become an
  unweighted cumulative sum.
- **A parameter-free four-tap quarter-channel token shift.** Channels [0..d/4]
  see t, [d/4..d/2] see t−1, [d/2..3d/4] see t−2, [3d/4..d] see t−3. A
  one-step shift at byte level spans two characters, too narrow to see a
  morpheme. Zero parameters, zero FLOPs, and it gives every layer a four-byte
  causal receptive field.
- **SubLN before the readout, and the residual on the output stream, never on
  the recurrent state.** Feeding the state straight into the classifier forces
  every channel to represent the *current* token, which penalises channels that
  hold history. Adding a residual inside the state pushes the transition
  eigenvalue above 1 and explodes at L=512.
- **A tied fp32 embedding and softmax head, updated with AdamW.** BitNet b1.58
  keeps the embedding and the LM head in floating point precisely because they
  sit at the discrete-to-continuous boundary. Tying them also gives every
  vocabulary row a dense gradient on every step, which is what fixes the
  sparse-update problem below.
- **A low-rank sign update for the ternary matrices.** dW = P·U with a rank-r
  orthonormal projector, refreshed from the real gradient at step 1 and then
  every K steps. r = 128, not 16.

## 3. What was measured, and what it taught

Every number below was measured on the target machine.

**The objective was the first blocker, and it was not the architecture.**
Regressing the hidden state onto a one-hot target with MSE produced the
diagnostic failure: **the loss kept falling while the accuracy fell**
(accuracy 0.3297 at step 1000 → 0.2992 at step 4000; val loss 0.001714 →
0.001662). MSE pushes every wrong byte down by the same amount regardless of its
probability and rewards nothing for *ranking*. Replacing it with a softmax head
and cross-entropy fixed it completely.

**The rank of the low-rank projection is the dominant factor.** Same task, same
data, 1000 steps: r=16 → 0.3028, r=64 → 0.3205, r=128 → 0.3297, full-rank
Adam → 0.3205, full-rank sign → 0.3280. The update can only move the weights
inside a rank-r subspace; at r=16 that is 16 directions out of 65536. And full
rank is *not* the answer — it is no better and twice as slow, because it
materialises the gradient. A gradient-spectrum measurement then showed the
energy is far more concentrated than the accuracy implies (rank 32 holds 92.5%
of the squared Frobenius norm, rank 64 holds 95.9%), so the thing that matters
is the *sign* of the gradient's tail, not its magnitude.

**More data fixed the overfitting; then the model ran out of parameters
instead.** On 8 MB of prose the train/validation gap grew to 0.89 nats. On
729 MB of English Wikipedia the gap stayed at 0.19–0.28 and the validation
cross-entropy was 0.4 nats better — and then it plateaued, flat in *both* train
and validation loss, which is the signature of a model that has stopped
learning rather than one that has run out of data.

**The stack beat the single block decisively.** Four layers, d=256, L=512,
r=128: validation accuracy 0.4035 (1.57× a bigram) and cross-entropy 2.1043 at
3,000 steps, against the single block's 0.3724 (1.47×) and 2.2581 at 15,000
steps. Five times fewer steps for a better model, and still falling.

**Generation was the clearest signal that something had changed.** Greedy
decoding looped (4-gram repetition 0.97, "The expersing and a propes and in the
expersing..."). Sampled at temperature 0.8 the repetition was **0.02**, with
word-shaped fragments, dates and punctuation. Greedy decoding is the wrong
probe for a small model.

**A negative result worth keeping:** a *learned ternary, untied* embedding was
worse than a fixed random one (1.09× vs 1.17× bigram). The cause was measured,
not guessed — its ternary codes changed on only 1.4% of positions per 50 steps
against 15–25% for the recurrent matrices, because a row is touched by one or
two tokens per batch, so the *sign* of its gradient is noise. The lesson is not
"the sign update is bad for sparse tables" but "we had made the embedding
ternary and untied, and BitNet does neither".

## 4. The bar, measured on our own corpus

Exact stupid-backoff n-gram models over the training split, scored on the same
held-out text:

| baseline | CE (nats) | bits/byte | accuracy |
|---|---|---|---|
| unigram | 3.3355 | 4.812 | 0.1472 |
| bigram | 2.6053 | 3.759 | 0.2637 |
| trigram | 2.1523 | 3.105 | 0.3704 |
| **4-gram** | **1.7521** | **2.528** | 0.4805 |
| 6-gram | 1.3679 | 1.974 | 0.5898 |

The single-block model at 2.2581 was a *smoothed trigram* — the trigram's
accuracy is 0.3704 against its 0.3724. The four-block stack at 2.1043 already
beat the trigram's 2.1523. **The 4-gram at 1.7521 nats is the milestone.**

Published numbers (Graves' 7-layer LSTM 1.158, mLSTM 0.860, Transformer-XL
0.68–0.73) are on **enwik8**, a different corpus and a different dump
snapshot; they are an order-of-magnitude guide, not a comparison. Our own
n-gram table is the comparison.

## 5. What the hardware allowed, measured

Full stack, L=512, one training step, no evaluation in the loop:

| layers | d | ternary params | tokens/s | one epoch over 729M tokens |
|---|---|---|---|---|
| 4 | 256 | 0.79M | 3,350 | 2.5 days |
| 8 | 256 | 1.57M | 2,271 | 3.7 days |
| 4 | 512 | 3.15M | 1,613 | 5.2 days |
| 8 | 512 | 6.29M | 1,104 | 7.6 days |
| 4 | 1024 | 12.58M | 740 | 11.4 days |

An earlier table extrapolated these from a single-block benchmark and was wrong
by 2–4×; the numbers above are measured on the real thing. The per-parameter
rate degrades with size because the packed weights stop fitting in the 8 MB L3,
which is also why the loop-inverted forward (each weight row fetched once per
*sequence* instead of once per token) is worth only 1.15× at d=4096 rather than
the factor of L the traffic argument suggests.

**The useful range on this box was 3–13M parameters at one to two weeks per
corpus epoch. 200M was a half-year and was dropped.**

## 6. The corpus

English Wikipedia, and only English. The dump is streamed and cut at 4 GB of
XML, stripped of wiki markup, then filtered to English articles by the rate of
common English function words (3% threshold; 29.7% of articles rejected; the
kept corpus has a median of 33%). Result: **729 MB of clean English prose**,
reproducible from `data/clean_wiki.py` and `data/filter_english.py`.

English-only was Ben's decision, and it was right: the first version of the
corpus was everything under `/usr/share/doc`, and the model's sampled output
repeated `<ref>` because a meaningful fraction of it was template.

## 7. The bugs that cost the most time

Worth more than the code, because each one was invisible until measured.

1. **A rational "fast sigmoid" that was not one.** It was Lambert's [3/2] tanh
   Padé approximant applied without the x/2 scaling and past its stationary
   point, so it grew like x/18 instead of saturating, returned values above 1,
   and made the gate coefficient a = 1 − z negative. The lesson: an
   approximation needs a written acceptance test. The one that passed:
   max error 5.307e-4, output range [0.0003803, 0.9996197], strictly inside
   (0,1) for every finite f32.
2. **A projector seeded from a random pattern.** The low-rank subspace was
   initialised from a fixed sin() function and, with a refresh interval of 200,
   never saw a gradient during a 200-step run. The optimiser was projecting the
   gradient onto a random subspace. The projector must be initialised from the
   first gradient.
3. **A 30-load-average machine.** Two orphaned training processes from an
   interrupted command held the CPU busy and made every timing wrong by 6–30×.
   A backward pass measured at 316 ms was really 1.4 ms. **Check `uptime` and
   `ps --sort=-pcpu` before believing a timing, always.**
4. **Square matrices hide shape bugs.** The output head was constructed with its
   arguments swapped, `(d, 256)` instead of `(256, d)`. At d=256 the two are
   the same number, so every experiment was unaffected and the bug only
   appeared at d=512. Assert the shapes at construction.
5. **Zombie master weights.** The Q1.14 master weights could sit at ±1.999
   while that row's ternary threshold was 0.5·γ — a weight parked ten times
   above its own switching boundary, needing ten times as many opposite-sign
   steps to come back. Clamp |w| ≤ 1.6γ before packing.
6. **One pass of Gram-Schmidt does not orthogonalise.** On an ill-conditioned
   input the projector came out with a Gram entry of −0.81 where 0 was
   required. Two passes.

## 8. Where it was left, and the first thing to do with better hardware

Untested, in order:

1. **The 4-gram bar was never crossed.** The best validation cross-entropy
   measured was 2.1043 against a 4-gram's 1.7521. The last run (8 layers,
   d=512, one epoch, with the three fixes) was killed when the project was
   parked; its curve was not read.
2. **Depth.** Four layers beat one by a wide margin; eight was never compared
   against four at equal wall-clock.
3. **Tokenisation.** Byte-level throughout. A BPE vocabulary of 4K–8K would
   multiply the effective character horizon by roughly four, and with a 5–17
   byte memory horizon that is the difference between word-shaped fragments and
   prose.
4. **Distillation into a 2-bit student.** A teacher run offline on the iGPU
   through llama.cpp's Vulkan backend, its top-16 token indices and quantised
   log-probabilities written to disk, the student trained here against them.
   This is the only realistic route to a model that answers questions, and it
   is cheap: the teacher never shares the CPU's memory bus while training runs.
5. **The measurement nobody made.** What does a *good* small byte-level model
   score on this corpus? Every bar we had was an n-gram. Without a real
   reference the model's 2.10 nats could be mediocre.

With a GPU, the same engine trains the same model 10–50× faster, and the
questions above become days rather than weeks. **The engine was never the
bottleneck; the CPU was.**

## 9. What is where

- `~/cognitive-2026-10-25.tar.gz` — the full source and git history (1.7 MB).
  `tar xzf cognitive-2026-10-25.tar.gz` and read `PRE2.md` first: it is the
  round-4 brief with every measurement, the issue list, and the questions that
  were still open.
- `PRE2.md` inside the archive — the authoritative brief.
- `RESEARCHER.md` and `RESEARCHER2.md` inside the archive — the two handovers to
  the researcher, and the two rounds of answers. Their diagnostics were better
  than ours twice: the missing gate bias, and the fact that a signed update on
  a sparsely-updated table is a random walk.
- The corpus is not archived. It rebuilds from `data/clean_wiki.py` and
  `data/filter_english.py`; `data/README.md` has the exact commands.
