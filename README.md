# TELOS Labs
**Exploring Statistical Process Control for AI Governance**

[![Status](https://img.shields.io/badge/status-research-blue)]()
[![License](https://img.shields.io/badge/license-research-orange)]()

> Can artificial intelligence systems be governed using proven quality management frameworks?

---

## The Research Question

Large language models exhibit measurable alignment degradation across extended conversations - a 20-40% reliability loss documented across multiple independent studies. This "semantic drift" creates compliance risks and operational challenges in production deployments.

Drawing from quality control expertise, we observed that this drift exhibits patterns familiar from manufacturing: measurable deviation from specification, temporal consistency, and response to intervention. This raises a fundamental question:

**Can Statistical Process Control mechanisms - proven over 70 years in safety-critical industries - effectively govern language model systems?**

TELOS Labs investigates this question through mathematical formalization, architectural analysis, and empirical research.

---

## The Problem: Semantic Drift

### Empirical Evidence

Multiple independent studies document systematic degradation:

- **Laban et al. (2025)**: Models lose instruction tracking with boundary violations after 20+ turns
- **Liu et al. (2024)**: "Lost in the Middle" - attention decay in 20-30 turn range
- **Wu et al. (2025)**: Position bias causes primacy effect decay
- **Gu et al. (2024)**: Attention sinks redistribute focus from constraints

This creates silent boundary violations that emerge gradually rather than suddenly.

### The Architectural Mechanism

**Key Discovery**: Drift emerges from reference point instability in transformer attention mechanisms.

Transformers compute similarity via scaled dot-product attention (Vaswani et al., 2017):

```
Attention(Q, K, V) = softmax(QK^T / √d_k) V
```

However, **RoPE positional encoding architecturally induces recency bias** (Yang et al., 2025), causing attention to measure similarity against recent context rather than original constraints.

**The core problem**: The similarity computation works perfectly. The reference point it measures against has drifted.

At turn 1: User declares constraint `P_0`
At turn 15: Model computes `Q_15 · K_14 ≈ high similarity` (recent context)
But: `K_14 · P_0 ≈ low similarity` (diverged from original constraint)

**Result**: Local coherence with global divergence.

This is an architectural limitation of attention mechanisms operating within context windows subject to positional encoding biases, not a training or prompting failure.

See [whitepaper](docs/whitepaper.md) for mathematical formalization.

---

## Our Approach

### Quality Management Perspective

My background in Lean Six Sigma revealed that semantic drift exhibits characteristics of process variation:

1. Measurable deviation from specification
2. Temporal degradation patterns
3. Response to intervention
4. Statistical predictability

This led to exploring whether quality control frameworks - DMAIC methodology, Statistical Process Control, process capability metrics - could inform AI governance approaches.

### Research Framework

We investigate whether concepts proven in manufacturing translate to semantic systems:

- **Define**: Formalize semantic drift as measurable phenomenon
- **Measure**: Establish quantitative metrics using embedding similarity
- **Analyze**: Identify root causes (reference point instability)
- **Improve**: Develop interventions preserving alignment
- **Control**: Implement continuous monitoring frameworks

**Important**: While inspired by established methodologies, specific applications to semantic systems involve novel adaptations that remain research subjects.

### Mathematical Foundation

TELOS treats alignment as a geometric problem in embedding space. By representing constraints and responses as vectors, we can measure fidelity using the same cosine similarity operation attention mechanisms employ internally - but with a stable external reference:

**Internal Attention** (drifts):
```
score_t = Q_t · K_recent^T
```

**External Measurement** (stable):
```
fidelity_t = cos(R_t, P_0) = (R_t · P_0) / (||R_t|| ||P_0||)
```

Where `P_0` is preserved from session initialization and never updated.

**Hypothesis**: External measurement with preserved reference points may provide governance metrics immune to attention mechanism drift.

---

## Research Status

**What we've established**:
- ✅ Mathematical formalization of drift measurement problem
- ✅ Architectural analysis of reference point instability
- ✅ Testable predictions for empirical validation
- ✅ Regulatory requirement mapping (EU AI Act Article 72, NIST AI RMF)
- ✅ Quality framework application concepts

**Open research questions**:
- Does fidelity measurement correlate with human alignment judgment?
- Can mathematical metrics detect drift before human observation?
- Do attention weight patterns predict fidelity degradation?
- Which SPC tools translate effectively to semantic systems?
- What quality levels are achievable (Six Sigma targets: <10 PPM)?

**Current phase**: Theoretical framework complete, empirical validation ongoing.

---

## Why External Measurement May Be Necessary

The model cannot fix reference point instability internally:

1. **Attention operates within context window** - no mechanism for stable external references
2. **RoPE is architectural** - recency bias built into positional encoding
3. **Training optimizes next-token prediction** - not governance persistence

This architectural constraint suggests external measurement with preserved reference points may be necessary where internal approaches cannot suffice.

---

## Regulatory Context

### EU AI Act Article 72
Requires "systematic and continuous post-market monitoring" with "procedures to actively gather, document, and analyze relevant data."

Current approaches lack:
- Continuous quantitative measurement during runtime
- Real-time detection before user exposure
- Quantitative deviation assessment
- Auditable evidence of active monitoring

**Research question**: Can quality control frameworks satisfy these requirements?

### NIST AI RMF
Requires "identified AI risks tracked over time" with "appropriate methods and metrics."

**Research question**: What mathematical frameworks provide suitable quantitative risk metrics?

---

## Research Opportunities

We welcome collaboration in seven major areas:

1. **Domain-Specific Validation** - Healthcare, legal, financial, education
2. **Cross-Vendor Testing** - GPT-4, Claude, Llama, Gemini
3. **Adversarial Robustness** - Novel attack strategies, defense mechanisms
4. **Theoretical Foundations** - Formal proofs, impossibility results
5. **Human Factors** - Trust calibration, UX design, organizational adoption
6. **Longitudinal Studies** - 6-12 month real-world deployments
7. **Economic & Policy** - Compliance ROI, regulatory harmonization

### Collaboration Model

**We share**: Theoretical framework, architectural analysis, research questions
**We seek**: Empirical validation, domain extensions, theoretical proofs
**IP policy**: Theory published openly, implementation methods remain research subjects

See [research opportunities](docs/research_opportunities.md) for details.

---

## Documentation

- **[Whitepaper](docs/whitepaper.md)** - Complete mathematical formalization
- **[Research Opportunities](docs/research_opportunities.md)** - Collaboration model
- **[References](docs/references.md)** - Bibliography

---

## About

TELOS Labs is a research initiative exploring quality management frameworks for AI governance. The work supports Lean Six Sigma Black Belt certification completion while investigating whether proven methodologies from safety-critical industries can address semantic drift challenges.

**Core insight**: Recognizing semantic drift through a quality control lens - as process variation rather than unsolvable alignment problem - suggests established frameworks may inform governance approaches.

**Research question**: Can this insight lead to practical, validated governance frameworks?

---

## Citation

```bibtex
@misc{teloslabs2025,
  title={Exploring Statistical Process Control for AI Governance},
  author={TELOS Labs},
  year={2025},
  howpublished={\url{https://github.com/TelosSteward/TELOS_Labs}},
  note={Research investigating quality management frameworks for language model governance}
}
```

---

## Contact

**Email**: telos.steward@gmail.com
**Research inquiries welcome**

---

## Acknowledgments

This work builds on foundational research in transformer architectures (Vaswani et al., 2017; Yang et al., 2025), semantic drift (Laban et al., 2025; Liu et al., 2024), quality control methodology (Shewhart, 1931; Deming, 1950), and regulatory frameworks (EU AI Act, 2024; NIST AI RMF, 2023).

See [references](docs/references.md) for complete bibliography.

---

*Investigating whether 70 years of quality management excellence can inform the next generation of AI governance.*
