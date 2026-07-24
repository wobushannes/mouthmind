# MouthMind – Technical FAQ

**For engineers, researchers, and critics.  
No fluff. No repetition. Just the details that matter.**

---

## 🔬 Model Architecture

**What is the model architecture exactly?**

The model is a **multi-view Seq2Seq architecture** with a **Transformer decoder**. It processes 8 distinct visual views through specialized encoders:

| View | Encoder | Input Dimension |
|------|---------|-----------------|
| Raw markers | Branchformer (4 layers) | 1434 |
| RP-plots | ResNet-18 | 3×64×64 |
| Mouth | MLP | 3 |
| Eyes | MLP | 3 |
| Jaw | MLP | 2 |
| Head | MLP | 3 |
| Lip Area | MLP | 1 |
| Asymmetry | MLP | 1 |

These views are **aligned in time**, **fused with learnable weights**, and projected to a **128-dimensional latent space**. The fusion weights are not fixed — they are learned during training, allowing the model to dynamically weigh the importance of each view.

The decoder is a **4-layer Transformer decoder** with:
- 4 attention heads
- 128-dimensional hidden size
- Causal masking (so predictions only depend on previous outputs)
- No teacher forcing

**Why no teacher forcing?**

Teacher forcing creates **exposure bias**. At inference time, the model must generate sequences from scratch — but during training, it's been conditioned on ground-truth tokens. This mismatch leads to brittle performance.

MouthMind **never sees ground-truth tokens** during training. It learns to generate sequences entirely from the visual signal. This is harder, but it results in:
- More robust emergent behaviors
- Better generalization to unseen words
- No exposure bias

**What is the training objective?**

The model minimizes **cross-entropy loss** between predicted character sequences and ground-truth labels. It uses:
- **AdamW optimizer** with weight decay
- **Gradient clipping** (1.0)
- **FP16 mixed precision** for efficiency
- **Batch size:** 128–256 (depending on phase)

---

## 📊 Data & Preprocessing

**How are the markers extracted?**

Using standard face-tracking libraries:
- **MediaPipe** (default) or **RetinaFace**
- Output: 478 3D landmarks per frame (x, y, z)

**What features are derived from the markers?**

- Raw positions (x, y, z)
- Velocity (first derivative)
- Acceleration (second derivative)
- Local curvature (third-order approximation)

These are concatenated into a **1434-dimensional feature vector** per frame.

**How are markers normalized?**

They are **centered** (subtract mean) and **scaled** (divide by standard deviation) per video. This removes camera-specific biases and ensures the model sees only the movement, not the absolute position.

**What are RP-plots?**

**Recurrence Plots** — a visualization technique that captures the temporal dynamics of the marker sequences. They encode how the movement repeats over time. The RP-plots are processed by a ResNet-18.

**Why so many views?**

Because each view captures a different aspect of speech:
- **Raw markers:** Full face geometry
- **RP-plots:** Temporal dynamics
- **Mouth:** Articulation
- **Eyes:** Emotional/attentional cues
- **Jaw:** Tension and stress patterns
- **Head:** Orientation and emphasis
- **Lip Area:** Acoustic-articulatory relationship
- **Asymmetry:** Biometric anomalies

**How were the labels created?**

1. Raw video is captured.
2. Markers are extracted.
3. The audio track is transcribed using **Whisper** (or similar ASR).
4. The transcription is aligned with the video frames.
5. The resulting (marker, text) pairs are stored as labeled sequences.

**Why is audio not used as input?**

- Audio introduces bias (speaker identity, accent, recording quality).
- The goal is visual speech recognition — not audio-visual.
- Audio is used only as a labeling tool.

**Is the data public?**

No. The dataset is not public — for legal and ethical reasons.

---

## 📈 Training Strategy

**How was the model trained?**

The model was trained in **multiple phases** to manage complexity:

| Phase | Data Fraction | Word Length | Epochs |
|-------|---------------|-------------|--------|
| 1 | 5% | 2–7 | 1000 |
| 2 | 10% | 2–7 | 300 |
| 3 | 20% | 2–7 | 100 |
| 4 | 5% | 2–6 | 100 |
| 5 | 10% | 2–7 | 300 |

Each phase **resumes** from the previous checkpoint. This staged approach allows the model to learn incrementally, without destabilizing.

**Why staged training?**

- **Controlled complexity:** The model learns short words first, then longer ones.
- **Stability:** Resuming from checkpoints prevents catastrophic forgetting.
- **Efficiency:** Each phase builds on the previous one.

**What is the current best performance?**

| Metric | Value |
|--------|-------|
| Loss | 0.62 |
| Letter Accuracy | >80% |
| WER | 0.24 |
| CER | 0.21 |

**What are the model's limitations?**

- Works best under **controlled conditions** (frontal faces, good lighting, clear articulation).
- Struggles with **visually similar letters** (e.g., 'e' vs 'ä').
- Fails on **proper names** and **rare words**.
- Performance drops on **unseen speakers** with very different articulation styles.

---

## 🔍 Emergent Behaviors

**What does "emergent generalization" mean?**

The model exhibits behaviors that were **not explicitly programmed**:

1. **Speaker independence:** Works on faces not seen during training.
2. **Word length generalization:** Transfers to longer words with reasonable accuracy.
3. **Head pose robustness:** Maintains performance across different head angles.
4. **Unseen vocabulary:** Recognizes words that were never in the training set.
5. **Cross-lingual transfer:** Reads English and Russian despite being trained on German.

**Why does this happen?**

Because the model learns **visemic patterns** — the underlying geometry of speech — rather than memorizing specific words. This is a direct result of:
- Using markers instead of pixels
- Training without teacher forcing
- Multi-view fusion

**Is the model perfect?**

No. It is a research prototype. It works under certain conditions — but there is no guarantee.

---

## 📁 Technical Data

**What is the feature dimension?**

1,434 — composed of:
- 478 markers × 3 coordinates = 1,434 raw features
- Plus velocity, acceleration, and curvature (not included in the count)

**What is the data format?**

HDF5, optimized for random-access I/O. Each sample contains:
- `raw`: 1434-dimensional features
- `rp`: 3×64×64 RP-plots
- `mouth`, `eyes`, `jaw`, `head`, `lip_area`, `asymmetry`: Individual views
- `labels`: Character sequences (with EOS token)

**What is the file size?**

~82 GB (compressed HDF5). The repository contains only aggregated statistics, not the raw data.

---

## 🧠 Model Understanding

**What does the model actually learn?**

It learns **visemic patterns** — the visual correlates of speech. Specifically:
- **Geometry:** How the face moves during speech.
- **Temporal dynamics:** How movements unfold over time.
- **Context:** How words and letters relate to each other.

It does **not** learn:
- Acoustic patterns
- Appearance-based shortcuts
- Semantic meaning

**How does the model handle different word lengths?**

The model uses an **EOS token** (End of Sequence) to handle variable-length words. It generates characters until it emits the EOS token — allowing it to produce words of any length.

**Why are the confidence scores low (~15%)?**

Low confidence indicates that the model is **conservative** — it does not hallucinate high-confidence predictions when uncertain. This is a feature, not a bug, for a high-risk AI system.

---

## 📚 Comparison with Other Approaches

**How does this compare to pixel-based models?**

| Aspect | Pixel-based | Marker-based (MouthMind) |
|--------|-------------|--------------------------|
| Input | RGB frames | 478 3D landmarks |
| Bias | Appearance, lighting, skin color | None |
| Generalization | Poor across speakers | Good across speakers |
| Efficiency | Millions of parameters | Compact |
| Privacy | Can reconstruct faces | Cannot reconstruct video |

**How does this compare to CTC-based models?**

| Aspect | CTC-based | Seq2Seq (MouthMind) |
|--------|-----------|---------------------|
| Dependency modeling | Independent per frame | Full context |
| Variable-length words | Struggles | Natural (EOS token) |
| Teacher forcing | Not applicable | Deliberately avoided |
| Generalization | Prone to overfitting | Robust emergent behaviors |

**What is the main innovation?**

The combination of:
1. **Pure marker-based input** (no pixels, no audio)
2. **No teacher forcing**
3. **Multi-view fusion**
4. **Staged training**

This combination results in a model that is **bias-free, efficient, and generalizes well** — without requiring massive datasets.
