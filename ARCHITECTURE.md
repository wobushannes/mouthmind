## Architecture Overview

MouthMind is built around a **multi-view Seq2Seq architecture** with a **hierarchical, modular, and dynamically reweighted fusion mechanism** — because one view is never enough, and static fusion is for people who don't understand the problem.

The following schematic provides a conceptual overview of the data flow — from raw input to character output. It is intentionally dense.

---

### Architecture Schematic

**INPUT** → Raw Video (stream, variable FPS, normalized)  
**↓**  
**FEATURE EXTRACTION PIPELINE**  
- **Primary stream:** 478 facial landmarks (x, y, z) per frame → MediaPipe/RetinaFace  
- **Secondary stream:** Optical flow approximation from landmark displacement (not pixel-based, calm down)  
- **Tertiary stream:** Recurrence plots (RP) from landmark trajectories → 3×64×64  
- **Derived features:** Velocity (Δ), acceleration (Δ²), local curvature (κ), jerk (Δ³) → 1434-dimensional feature vector per frame  
- **No pixels. No audio. No exceptions.**

**↓**  
**VIEW ORGANIZATION (8 complementary views)**

| View | Dimension | Encoder | Notes |
|------|-----------|---------|-------|
| Raw markers | 1434 | Branchformer (4 layers, 4 heads) | Full face geometry |
| RP-plots | 3×64×64 | ResNet-18 (pretrained, fine-tuned) | Temporal dynamics visualization |
| Mouth | 3 | MLP (64 → 128 → 256) | Articulation-specific |
| Eyes | 3 | MLP (64 → 128 → 256) | Saccadic & attentional cues |
| Jaw | 2 | MLP (64 → 128 → 256) | Tension, stress, emphasis |
| Head | 3 | MLP (64 → 128 → 256) | Orientation, gaze, prosody |
| Lip Area | 1 | MLP (64 → 128 → 256) | Acoustic-articulatory relationship |
| Asymmetry | 1 | MLP (64 → 128 → 256) | Biometric anomalies, micro-expressions |

**↓**  
**TEMPORAL ALIGNMENT & SYNCHRONIZATION**  
- All views padded to max length  
- Positional encoding applied per view  
- Cross-view temporal alignment via dynamic time warping (DTW) approximation  

**↓**  
**FUSION LAYER – DYNAMIC REWEIGHTING**  
- Learnable weights per view (initialized to 0.1)  
- Softmax normalization → attention-like weighting  
- Weighted sum → fused representation  
- Projected to 128-dimensional latent space via linear transformation + LayerNorm + GELU  
- **Weights are not static.** They are updated during training. The model learns which views matter, for which inputs, at which times.  
- *This is not attention.* This is fusion. There is a difference. If you don't know it, you're not the target audience.

**↓**  
**LATENT SPACE**  
- 128-dimensional, normalized, unit variance  
- Serves as the sole input to the decoder  
- Contains the fused, temporally aligned, geometrically abstracted representation of the entire utterance  

**↓**  
**TRANSFORMER DECODER**  
- 4 layers  
- 4 attention heads  
- 128-dimensional hidden size  
- Causal masking (bidirectional is for transformers, not for generation)  
- **No teacher forcing.** The model never sees ground-truth tokens during training. It learns to generate sequences from scratch. This is not a bug. It is the entire point.  
- *Exposure bias is avoided because exposure bias is avoided. If you don't understand why this matters, you are not the target audience.*

**↓**  
**OUTPUT**  
- Character sequence generated autoregressively  
- Vocabulary: 43 tokens (letters + PAD, BLANK, EOS)  
- EOS token signals end of word  
- Training objective: cross-entropy loss on character sequences  
- **No CTC. No HMM. No alignment tricks. Just raw generation.**

---

### Why This Complexity Is Necessary

This architecture is not designed to be simple. It is designed to be **correct**.

The problem of visual speech recognition is fundamentally multimodal, temporal, and noisy. Single-view approaches fail because they ignore critical information (e.g., head pose, asymmetry, temporal dynamics). Static fusion fails because the importance of each view varies by context. Teacher forcing creates exposure bias, which degrades generalization.

This architecture addresses these challenges directly:
- **Multi-view fusion** captures all relevant information.
- **Dynamic reweighting** adapts to context.
- **No teacher forcing** forces robust generalization.
- **Staged training** ensures incremental learning.

It is not a black box. It is a carefully engineered system. If it looks complicated, that is because the problem is complicated. If it looks like overengineering, you have not understood the problem.

---

### Data Strategy

The underlying dataset is not drawn from standard benchmarks (e.g., LRW, LRS3, GRID). Those datasets are lab-grown — controlled lighting, fixed head poses, limited speakers, and narrow phonetic coverage. They are not representative of real-world conditions.

MouthMind's data strategy prioritizes:
- **Uniqueness:** A large, diverse corpus that covers a broad range of speakers, accents, and recording conditions.  
- **Density:** Geometric markers instead of pixels, ensuring high information density and low redundancy.  
- **Coverage:** Over 1,100 speakers, multilingual content, and significant phonetic diversity (~22.9% of PHOIBLE 2.0).  

The goal is not to perform well on a single benchmark. The goal is to perform robustly in **real-world, non-laboratory conditions** — with variable lighting, head poses, speaking styles, and even partial occlusions. This is not a "clean data" project. It is designed to work where other models fail.

---

### What This Diagram Does Not Show

This description intentionally omits:
- Layer-specific weight initializations
- Exact optimizer configurations
- Batch-normalization placement
- Dropout rates
- Learning rate schedules
- Data augmentation strategies

These are implementation details. They are not necessary to understand the architecture, and they are not public.

---

This architecture ensures that the model learns **visemic patterns** — the geometric movements of speech — rather than appearance-based shortcuts or acoustic signals. The combination of multi-view fusion, no teacher forcing, and staged training results in robust emergent generalization across speakers, head poses, and even languages.
