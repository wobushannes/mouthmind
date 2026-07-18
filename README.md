# MouthMind

**A research framework for visual speech recognition — based on a multi-view Seq2Seq architecture without teacher forcing.**

---

## TL;DR

MouthMind rethinks visual speech recognition not as classification, but as **sequential character generation**.

Using a **multi-view fusion architecture** and a **Transformer decoder**, the model learns to generate words directly from visual input — without teacher forcing. This leads to **robust emergent generalization**, including the ability to recognize unseen words and adapt to different head poses.

---

## Overview

MouthMind is the result of an **eight-month solo research effort**.

Initial experiments with CTC-based models proved too rigid for the natural variance of lip reading. The breakthrough came with the shift to a **Seq2Seq architecture** — framing the problem as sequential character generation rather than static classification.

### Key Properties

- **Multi-View Fusion** – Integrates complementary visual signals from the face into a unified representation.
- **No Teacher Forcing** – The model learns exclusively from visual patterns, leading to more robust generalization.
- **Emergent Generalization** – Transfers to unseen words, speakers, and head poses.
- **Speaker-Agnostic Design** – Handles individual variance in articulation, anatomy, and camera perspective.

---

## Architecture

MouthMind uses a **multi-view fusion architecture** that integrates multiple complementary visual signals from the face — including lip movement, head orientation, and facial asymmetry — into a unified representation.

Each signal is processed by a dedicated encoder (Branchformer, ResNet-18, or MLP), and the resulting features are aligned, fused with learnable weights, and projected into a shared latent space.

A **Transformer decoder** with causal masking then generates the character sequence **autoregressively** — without teacher forcing. This forces the model to learn genuine patterns of speech production rather than relying on shortcut supervision.

---

## Results

### Training Metrics

| Metric | Value |
|--------|-------|
| Training Corpus | 300,000+ word clips |
| Vocabulary | 1.5M+ characters |
| Best Loss | ~0.85 |
| Best WER | 0.0 |
| Best CER | 0.0 |
| Letter Accuracy | >50% on unseen words |

### Example Predictions

| Target | Prediction | Status |
|--------|------------|--------|
| ich | ich | ✅ |
| der | der | ✅ |
| aber | aber | ✅ |

---

## Video Demos

The following videos demonstrate MouthMind's ability to generalize to **unseen vocabulary** — words that were **never part of the training set**.

These tests were performed under varying head poses and speaking styles.

### Demo: Unseen Vocabulary

| Video | Description |
|-------|-------------|
| [a1.mp4](./video/a1.mp4) | Unseen word — stable prediction |
| [a2.mp4](./video/a2.mp4) | Different head angle — robust recognition |
| [a3.mp4](./video/a3.mp4) | Unseen phrase — partial but meaningful output |
| [a4.mp4](./video/a4.mp4) | Varied speaking style — consistent performance |

> **Note:** These words were **not** part of the training data. The model's ability to recognize them is a direct result of its emergent generalization — it has learned underlying patterns of speech production, not memorized a fixed vocabulary.

---

## Emergent Behaviors

MouthMind demonstrates **emergent generalization** across multiple dimensions:

- **Speaker Independence** – Works on faces not seen during training.
- **Word Length Generalization** – Transfers to longer words with reasonable accuracy.
- **Head Pose Robustness** – Maintains performance across different head angles.
- **Unseen Vocabulary** – Recognizes words that were never in the training set.

---

## Research Context

### Comparison with CTC-Based Models

CTC models treat visual speech as a static classification problem — they predict characters independently.

MouthMind's Seq2Seq approach, by contrast, generates characters **conditioned on the full visual context**, leading to:
- Higher accuracy on longer words
- Better generalization to unseen vocabulary
- Natural handling of word boundaries

### The Role of Teacher Forcing

Most Seq2Seq models rely on teacher forcing (TF) to stabilize training.

MouthMind deliberately **does not use TF** — the model must generate sequences entirely from visual input. This results in:
- More robust emergent behaviors
- Better generalization to unseen words
- No exposure bias — the model learns to correct its own errors

---

## Risks & Responsible AI

MouthMind is a **high-risk AI research project**.

Visual speech recognition has significant potential for misuse in surveillance, privacy invasion, and other harmful applications.

**Why we don't share the code:**

- Preventing misuse — the technology is powerful and could be weaponized
- Protecting privacy — the model can read silent speech, which is a privacy concern
- Ethical constraints — we take our responsibility seriously

**What we do share:**

- Research findings and insights
- Demo videos in a controlled environment
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

This project is **not open source**.

All rights reserved.

© 2026 MouthMind Research

---

*"We show what's possible — not how it's done."*
