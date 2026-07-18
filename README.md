# MouthMind

**A research framework for visual speech recognition — based on a multi-view Seq2Seq architecture without teacher forcing.**

---

## TL;DR

MouthMind rethinks visual speech recognition not as classification, but as **sequential character generation**.

Using a **multi-view fusion architecture** and a **Transformer decoder**, the model learns to generate words directly from visual input — without teacher forcing. This leads to **robust emergent generalization**, including the ability to recognize unseen words and adapt to different head poses.

---

## From Mesh to Mind: The Data Foundation

This work builds on a **multilingual mesh-based VSR dataset** I introduced earlier ([`multilingual-mesh-vsr`](https://github.com/wobushannes/multilingual-mesh-vsr)). That dataset established the core principle: **pure geometric landmarks instead of pixels** – bias-free, dense, and speaker-agnostic.

For MouthMind, I transitioned from **phoneme-level labels** to **character-level sequences**, enabling a more direct, word-based training paradigm while retaining the data's inherent advantages.

**Key metrics of the underlying mesh dataset:**

| Metric | Value |
|--------|-------|
| Total segments | 237,025 |
| Unique speakers | 1,121 |
| Total frames | 14.3M |
| Feature dimension | 1,434 (3D landmarks + derivatives) |
| Effective pixel equivalents | ~20.5T |
| Phoneme coverage | ~22.9% of PHOIBLE 2.0 |

This is the bias-free, high-density foundation that MouthMind builds upon. The dataset is not publicly available – it is a research asset.

---

## Overview

MouthMind is the result of an **eight-month solo research effort** by **Johannes Wobus**.

The journey began with CTC-based models, which proved too rigid for the natural variance of lip reading. The breakthrough came with the shift to a **Seq2Seq architecture** — framing the problem as sequential character generation rather than static classification.

A central design decision was the use of **facial markers instead of raw pixels** — for three reasons:

- **Bias reduction** — markers abstract away from skin color, lighting, and individual physiognomy.
- **Generalization** — marker-based representations transfer better across speakers and recording conditions.
- **Information density** — markers encode the essential geometric information without the noise of pixel-level data.

This approach ensures the model learns genuine patterns of speech production rather than superficial correlations. It also makes the system **computationally efficient** — working with a compact, high-level representation instead of processing millions of pixels per frame.

---

## Why This Matters

Visual speech recognition — the ability to understand what someone is saying by watching their face — has been a long-standing challenge in computer vision. Traditional approaches have struggled with the variability of human speech: different speakers, accents, head poses, lighting conditions, and camera angles.

Most existing systems are trained on massive datasets of labeled video, often requiring hundreds of thousands of hours of footage. Even then, they fail to generalize to new speakers or environments. They are brittle, biased, and expensive to deploy.

MouthMind takes a different path. By focusing on the **geometric structure of speech** — the actual movements of the face — and by treating speech as a **sequential generation problem**, the model learns to understand what is being said without relying on appearance-based shortcuts.

This is not just a technical improvement. It is a shift in perspective: from recognizing patterns to **understanding the underlying process of speech production**.

---

## Architecture

### Marker-Based Feature Extraction

MouthMind does not process raw video frames. Instead, it uses **facial markers** — a set of geometrically meaningful points on the face that track movement over time. These markers are extracted using standard face tracking and provide a **compact, bias-free representation** of the speaker's face.

The use of markers offers several advantages:

- **Independence from appearance** — the model sees only movement, not skin color, clothing, or lighting.
- **Robustness to recording conditions** — markers are stable across different cameras and environments.
- **High information density** — each marker encodes precise geometric information relevant to speech.

### Multi-View Fusion

The marker data is organized into **multiple complementary views**, each capturing a different aspect of facial movement. These views are processed by specialized encoders and then fused into a unified representation.

The multi-view approach ensures that the model captures **all relevant aspects** of speech production — from the fine movements of the lips to the global orientation of the head.

### Transformer Decoder

The fused representation is passed to a **Transformer decoder** that generates the character sequence **autoregressively** — one character at a time. The decoder uses **causal masking** to ensure that predictions at each step depend only on previous predictions.

Crucially, the model is trained **without teacher forcing**. This means it never receives the ground truth characters as input during training. Instead, it must learn to generate sequences entirely from the visual signal. This is a more difficult training objective, but it leads to **more robust generalization** and eliminates the problem of exposure bias.

---

## Results

### Training Logs

Raw training logs (JSON) are available in the [`stats/`](./stats) folder.

Selected logs include full prediction logs with WER, CER, and Letter Accuracy.

### Key Metrics

| Metric | Best Value | Epoch |
|--------|------------|-------|
| Loss | 0.37 | 650 |
| WER | 0.24 | 958 |
| CER | 0.21 | 958 |
| Letter Accuracy | 80.3% | 958 |

### Visualizations

All training logs and visualizations are available in the [`charts/`](./charts) folder.

### Video Demos

Demo videos are available in the [`video/`](./video) folder.

> **Note:** The overlay text in the videos is the model's **real-time prediction**.  
> These are inference demos, not training data. The videos show the model's live prediction output during inference on unseen input.

---

## Emergent Behaviors

MouthMind demonstrates **emergent generalization** across multiple dimensions:

- **Speaker Independence** – Works on faces not seen during training, demonstrating that the model has learned speaker-agnostic features.
- **Word Length Generalization** – Transfers to longer words with reasonable accuracy, showing understanding of sequential structure.
- **Head Pose Robustness** – Maintains performance across different head angles, indicating invariance to viewpoint changes.
- **Unseen Vocabulary** – Recognizes words that were never in the training set, proving that the model has not simply memorized a fixed vocabulary.

These emergent behaviors are the result of the model's architecture and training regime — particularly the absence of teacher forcing, which forces the model to learn genuine patterns rather than relying on shortcuts.

---

## Research Context

### Why Markers Instead of Pixels?

Most visual recognition systems operate on raw pixels. While this works well for large-scale datasets, it introduces significant biases:

- **Appearance bias** — models learn to associate words with skin color, lighting, or clothing, rather than with the actual movements of speech.
- **Limited generalization** — performance drops sharply when the recording setup changes, because the model has learned superficial correlations rather than meaningful patterns.
- **Information overload** — pixels contain massive amounts of irrelevant data (background, texture, lighting) that the model must learn to ignore, wasting capacity.

By using facial markers, MouthMind avoids these problems entirely. The model sees only the **geometric structure of speech** — the movements that actually carry information. This leads to:

- **Bias-free learning** — no correlation with skin color, lighting, or personal appearance.
- **Better generalization** — the model transfers across speakers, cameras, and environments because it has learned the underlying structure of speech.
- **Higher information density** — each marker carries meaningful information about speech, making the model more efficient.

### Comparison with CTC-Based Models

Connectionist Temporal Classification (CTC) is a common approach to sequence labeling problems. CTC models treat visual speech as a static classification problem — they predict characters independently at each frame, relying on the CTC loss to handle alignment.

However, CTC has several limitations:

- It does not model dependencies between characters, leading to errors in longer words.
- It struggles with variable-length sequences because it cannot condition predictions on previous outputs.
- It is prone to overfitting because it treats each frame independently.

MouthMind's Seq2Seq approach, by contrast, generates characters **conditioned on the full visual context**, leading to:

- **Higher accuracy on longer words** — because the model can use contextual information.
- **Better generalization to unseen vocabulary** — because it has learned sequential structure.
- **Natural handling of word boundaries** — via explicit EOS tokens.

### The Role of Teacher Forcing

Most Seq2Seq models rely on teacher forcing (TF) to stabilize training. With TF, the model is fed the ground truth output at each step, rather than its own predictions. This makes training easier because the model never has to recover from its own errors.

However, TF introduces a serious problem: **exposure bias**. At inference time, the model must generate sequences using its own predictions, which may be flawed. If it has never learned to correct its own errors during training, performance suffers.

MouthMind deliberately **does not use TF** — the model must generate sequences entirely from visual input. This is a more difficult training objective, but it leads to:

- **More robust emergent behaviors** — the model learns to recover from its own errors.
- **Better generalization to unseen words** — because it has learned to generate sequences rather than memorize them.
- **No exposure bias** — the model is trained in the same way it is used at inference time.

---

## Ethical Considerations

MouthMind is a **high-risk AI research project**. Visual speech recognition has significant potential for misuse in surveillance, privacy invasion, and other harmful applications.

We take this responsibility seriously. This project is guided by the following ethical principles:

### 1. Transparency Without Vulnerability

We share our research findings and insights openly — including the architecture, training approach, and results. However, we do not share the code, model weights, or training data. This balance ensures that the scientific community can benefit from our work without enabling misuse.

### 2. Privacy by Design

The model is designed to work with facial markers rather than raw video frames. This reduces the privacy impact of the system — markers are anonymized geometric points that cannot be used to reconstruct the original video.

### 3. Controlled Access

We do not provide open access to the model. Access is restricted to institutions with proven research experience and a clear ethical framework. Commercial use is strictly prohibited.

### 4. Active Misuse Prevention

We monitor how our work is discussed and used. If we become aware of misuse, we reserve the right to take appropriate action, including issuing public warnings and engaging with relevant authorities.

### 5. Commitment to Public Benefit

While we restrict access to the model itself, we remain committed to the public good. We share our findings, insights, and lessons learned so that others can build on our work in responsible ways.

These principles are not optional. They are core to the project.

---

## Risks & Responsible AI

MouthMind is a **high-risk AI research project**.

Visual speech recognition has significant potential for misuse in surveillance, privacy invasion, and other harmful applications.

**Why we don't share the code:**

- **Preventing misuse** — the technology is powerful and could be weaponized for surveillance, social control, or discrimination.
- **Protecting privacy** — the model can read silent speech, which is a profound privacy concern. Even with markers, the technology is sensitive.
- **Ethical constraints** — we take our responsibility seriously. Building technology is not enough; we must also ensure it is not used for harm.

**What we do share:**

- Research findings and insights
- Demo videos in a controlled environment
- Technical documentation (this README)
- Discussion of the ethical implications of the technology

We believe in **open science** — but we also believe in **responsible disclosure**. Some technologies are too dangerous to be shared freely. MouthMind is one of them.

---

## FAQ

**Can I get the code or model?**  
No. Code, weights, and datasets are not public — and never will be. This is not negotiable.

**I'm a researcher / student. Can I get access?**  
No. Access is restricted to institutions with proven research experience and a clear ethical framework.

**Can I use it commercially?**  
No. Commercial use is strictly prohibited. No licenses. No exceptions.

**Can I get a live demo?**  
No. We provide demo videos in this repository. Live demos are not available.

**Why so restrictive?**  
Because visual speech recognition has misuse potential. We take this seriously — even if others don't.

**Can I collaborate?**  
Yes — if you're from a university, institute, or comparable research environment. Contact us with your profile, institution, and concrete proposal. All other inquiries will be ignored.

**I'm from a law enforcement or security agency.**  
We take legitimate security inquiries seriously. Contact us with your credentials and a formal request.

**Can I discuss the research with you?**  
Yes — in academic or policy contexts. Contact us with a clear proposal.

**Can I hire you as a consultant?**  
We offer consulting and research collaboration for institutions with a clear ethical framework. Contact us for details.

---

## Contact

Serious inquiries only.

**ProtonMail:** `blende_32@protonmail.com`  
**Threema:** `BA46EWMP`

**Before contacting us:**

- Provide full name and institution
- State your concrete purpose
- Don't ask for code or access — it will be ignored

---

## License

This project is **not open source**.

All rights reserved.

© 2026 Johannes Wobus — MouthMind Research

## Acknowledgments

Thanks to the three people who reviewed this work — for their honesty, their questions, and their time.

The rest was just me and the machine.

---

*"We show what's possible — not how it's done."*
