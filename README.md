# MouthMind – Visual Speech Recognition
## A Multi-View Seq2Seq Framework for Silent Speech Understanding

---

## TL;DR

**MouthMind** is a research framework that rethinks visual speech recognition — not as a classification problem, but as **sequential character generation**. Using a **multi-view fusion architecture** and a **Transformer decoder without teacher forcing**, the model learns to generate words directly from visual input, demonstrating robust emergent generalization to unseen vocabulary and head poses.

---

## Overview

MouthMind is the result of an eight-month research journey — a solo project built on the conviction that visual speech processing requires a fundamentally different approach. Initial experiments with CTC-based models proved too rigid for the natural variance of lip reading. The breakthrough came with the shift to a **Seq2Seq architecture with a Transformer decoder**, framing the problem as **sequential character generation** rather than classification.

### Key Features

- **Multi-View Fusion** – Integrates 8 complementary visual perspectives: Raw video, RP-plots, Mouth, Eyes, Jaw, Head, Lip Area, and Asymmetry.
- **No Teacher Forcing** – The model learns to generate characters **exclusively from visual patterns**, leading to more robust and generalizable capabilities.
- **Emergent Generalization** – The model transfers to unseen words and head poses, demonstrating it learns underlying patterns of speech production rather than memorizing.
- **Speaker Physiology Awareness** – Explicitly designed to handle individual variance in articulation, anatomy, and camera perspective.

---

## Architecture

### Multi-View Feature Extraction

MouthMind extracts features from **8 distinct views**:

| View | Description |
|------|-------------|
| Raw | Raw video frames |
| RP | Recurrence Plots |
| Mouth | Mouth region landmarks |
| Eyes | Eye region landmarks |
| Jaw | Jaw position |
| Head | Head orientation |
| Lip Area | Lip movement area |
| Asymmetry | Facial asymmetry |

These views are processed by specialized encoders:
- **Raw → Branchformer Encoder**
- **RP → ResNet-18**
- **Other views → MLP Encoders**

### Fusion & Decoding

All views are aligned, fused with learnable weights, and projected to a unified latent space. A **Transformer Decoder** with causal masking then generates the character sequence **autoregressively** — without teacher forcing.

---

## Results

### Training Performance

| Metric | Value |
|--------|-------|
| Training Corpus | 300,000+ word clips |
| Vocabulary | 1.5M+ characters |
| Best Loss | ~0.85 |
| Best WER | 0.0 |
| Best CER | 0.0 |
| Letter Accuracy | >50% on unseen words |

### Loss Progression

![Loss Curve](./charts/loss.png)
*Loss decreases over 400 epochs, showing stable learning with characteristic "exploration spikes" as the model adapts to new word lengths.*

### Example Predictions

| Target | Prediction | Status |
|--------|------------|--------|
| ich | ich | ✅ Perfect |
| der | der | ✅ Perfect |
| aber | aber | ✅ Perfect |
| zusammenzieht | zusammeniahee | ✅ 61% letter accuracy |
| sprechen | aer | ❌ False |

---

## Video Demos

The following videos demonstrate MouthMind's ability to generalize to **unseen, emotionally charged vocabulary** — words that were **never part of the training set**.

These tests were performed under varying head poses and speaking styles, showing the model's robustness and emergent understanding of visual speech patterns.

### Demo: Unseen Vocabulary

| Video | Description |
|-------|-------------|
| [a1.mp4](./video/a1.mp4) | Unseen word — model predicts with high confidence |
| [a2.mp4](./video/a2.mp4) | Different head angle — stable recognition |
| [a3.mp4](./video/a3.mp4) | Unseen phrase — partial but meaningful prediction |
| [a4.mp4](./video/a4.mp4) | Varied speaking style — robust performance |

> **Note:** These words were **not** part of the training data. The model's ability to recognize them is a direct result of its emergent generalization — it has learned underlying patterns of speech production, not memorized a fixed vocabulary.

---

## Emergent Behaviors

Although primarily trained on shorter words, MouthMind demonstrates **emergent generalization**:

- **Speaker Independence** – Works on faces not seen during training.
- **Word Length Generalization** – Transfers to longer words with reasonable accuracy.
- **Head Pose Robustness** – Maintains performance across different head angles.
- **Unseen Vocabulary** – Recognizes words that were never in the training set.

---

## Research Context

### Comparison with CTC-Based Models

CTC models treat visual speech as a static classification problem — they predict characters independently. MouthMind's Seq2Seq approach, by contrast, generates characters **conditioned on the full visual context**, leading to:

- **Higher accuracy** on longer words
- **Better generalization** to unseen vocabulary
- **Natural handling** of word boundaries (via EOS token)

### The Role of Teacher Forcing

Most Seq2Seq models rely on teacher forcing (TF) to stabilize training. MouthMind deliberately **does not use TF** — the model must generate sequences entirely from visual input. This results in:

- **More robust emergent behaviors**
- **Better generalization** to unseen words
- **No exposure bias** — the model learns to correct its own errors

---

## Risks & Responsible AI

MouthMind is a **high-risk AI research project**. Visual speech recognition has significant potential for misuse in surveillance, privacy invasion, and other harmful applications.

**Why we don't share the code:**

- **Preventing misuse** — The technology is powerful and could be weaponized.
- **Protecting privacy** — The model can read silent speech, which is a privacy concern.
- **Ethical constraints** — We take our responsibility seriously.

**What we do share:**

- Research findings and insights
- Demo videos (controlled environment)
- Technical documentation (this README)

---

## FAQ

**Can I get the code or model?**  
No. Code, weights, and datasets are not public — and never will be. This is not negotiable.

**I'm a researcher / student. Can I get access?**  
No. Access is restricted to institutions with proven research experience and a clear ethical framework.

**Can I use it commercially?**  
No. Commercial use is strictly prohibited. No licenses. No exceptions.

**Can I get a demo?**  
Yes — we have public demo videos in this repository. Live demos are only available under strict conditions (institutional background, clear purpose, confidentiality agreement).

**Why so restrictive?**  
Because visual speech recognition has misuse potential. We take this seriously — even if others don't.

**Can I collaborate?**  
If you're from a university, institute, or comparable research environment: Contact us with your profile, institution, and concrete proposal. All other inquiries will be ignored.

---

## Contact

Serious inquiries only via ProtonMail:  
`[your-email@protonmail.com]`

**Before contacting us:**

- Provide full name and institution
- State your concrete purpose
- Don't ask for code or access — it will be ignored

---

## License

This project is **not open source**. All rights reserved.

© 2026 MouthMind Research

---

*"We show what's possible — not how it's done."*
