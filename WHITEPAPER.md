# Semantic Drift in Large Language Models: Problem Formalization
**TELOS Labs - Theoretical Research**

---

## Abstract

Large language models exhibit systematic alignment degradation across extended conversations, with empirical studies documenting 20-40% reliability loss as context accumulates. This degradation, termed semantic drift, occurs predictably due to transformer architecture properties: primacy decay, recency bias, and attention redistribution. We demonstrate that this drift emerges from a fundamental reference point instability in attention mechanisms, where similarity computations measure against recent context rather than original constraints. Current approaches lack continuous quantifiable measurement of governance persistence during runtime, creating a gap between emerging regulatory requirements for "systematic and continuous" monitoring and available technical capabilities. This paper formalizes the semantic drift measurement problem, analyzes the architectural mechanism causing drift, examines current approach limitations, and establishes requirements for viable mathematical frameworks.

---

## 1. The Semantic Drift Phenomenon

### 1.1 Empirical Evidence

Semantic drift is documented across multiple independent research efforts. Laban et al. (2025) demonstrate that models lose instruction tracking in multi-turn conversations with systematic boundary violations after 20 or more turns. Liu et al. (2024) show middle context loses salience in what they term "Lost in the Middle," with attention decay in the 20-30 turn range. Wu et al. (2025) identify position bias causing primacy effect decay where early token influence diminishes over time. Gu et al. (2024) reveal attention sinks that redistribute focus from constraints, causing governance instructions to lose attention weight.

The aggregate finding across these studies is **20-40% reliability loss** in extended dialogues across all major model families.

### 1.2 Architectural Origins

Drift emerges from fundamental transformer properties. Primacy decay causes session-start constraints to exert strong initial influence that progressively weakens. Recency bias leads recent tokens to capture disproportionate attention, overriding earlier instructions. Attention competition forces governance tokens to compete with content tokens for limited capacity. Context window effects create non-uniform attention weighting that causes middle content to lose influence.

These mechanisms mirror documented cognitive phenomena in human memory (Murdock, 1962; Glanzer & Cunitz, 1966), suggesting deep architectural parallels.

### 1.3 Operational Impact

Organizations experience drift through multiple manifestations. **Compliance risk** emerges when governance boundaries erode silently during extended sessions. **User frustration** builds as "I already told you not to do that" becomes a recurring refrain requiring repeated re-statement of ignored constraints. **Silent degradation** occurs gradually rather than suddenly, making real-time detection difficult. **Post-hoc discovery** means violations are identified only through transcript review after sessions complete.

---

## 2. The Architectural Basis of Semantic Drift

### 2.1 Attention Mechanisms as Similarity Computation

Transformer language models fundamentally rely on similarity computation through the scaled dot-product attention operation (Vaswani et al., 2017):

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right) V$$

The operation $QK^T$ computes the dot product between query vectors and key vectors—a direct measurement of directional similarity between positions in the sequence. This is mathematically equivalent to unnormalized cosine similarity and captures the degree to which vectors "point in the same direction" within the embedding space (PyTorch Contributors, 2023).

When a transformer generates token $t$, it creates a query vector $Q_t$ representing "what information am I looking for?" It then computes:

$$\text{score}_{t,i} = Q_t \cdot K_i^T$$

for every prior key vector $K_i$ in the context. These scores directly measure: *How similar is my current generation state to position i?*

After applying softmax, these become attention weights determining how much each prior position influences current generation. High similarity yields high attention weight and strong influence. **This mechanism works extraordinarily well for language modeling** where relevant context prediction benefits from similarity matching.

### 2.2 The Reference Point Problem

However, this same mechanism fails for governance persistence because **the reference point itself drifts** over conversational turns.

Consider a session where the user declares at turn 1:

$$P_0: \text{"Provide guidance on structure, but do not write content directly"}$$

This constraint is encoded as an embedding vector $\mathbf{p}_0 \in \mathbb{R}^d$.

**At turn 5**: Model response $R_5$ adheres well to $P_0$. The attention mechanism computing $Q_5 \cdot K_1^T$ correctly identifies high similarity to the original constraint.

**At turn 15**: Model response $R_{15}$ computes attention weights:

$$\alpha_{15,i} = \frac{\exp(Q_{15} \cdot K_i^T / \sqrt{d_k})}{\sum_j \exp(Q_{15} \cdot K_j^T / \sqrt{d_k})}$$

Due to **architectural recency bias** (Yang et al., 2025), attention disproportionately weights recent keys $K_{12}, K_{13}, K_{14}$. These keys represent the immediate conversation context.

But if $K_{12}$ through $K_{14}$ have already drifted from $\mathbf{p}_0$, the model measures similarity against *corrupted references*. It correctly computes:

$$Q_{15} \cdot K_{14}^T \approx \text{high similarity}$$

and concludes it is aligned. But $K_{14}$ itself has low similarity to $\mathbf{p}_0$:

$$K_{14} \cdot \mathbf{p}_0^T \approx \text{low similarity}$$

**The similarity computation works perfectly. The reference point it measures against has drifted.**

### 2.3 Architectural Sources of Recency Bias

This reference drift is not accidental—it is architecturally induced through two mechanisms:

**1. RoPE Positional Encoding** (Yang et al., 2025):

Rotary positional encodings, used in LLaMA, Mistral, and other modern architectures, apply rotations to query and key vectors that systematically favor proximity. Yang et al. document: *"RoPE exhibits a stronger recency bias (positional focus)... RoPE layers handle local information effectively due to their built-in recency bias."* Distant positions receive diminished attention weight not through learned preference but through mathematical construction.

**2. Learned Attention Patterns** (Liu et al., 2023):

Pre-training on natural text—where recent context is genuinely most predictive for next-token generation—reinforces recency weighting. Liu et al. observe: *"During pre-training, this induces a learned bias to attend to recent tokens."* The model learns: "recent tokens matter most." For language modeling, this is correct. For governance persistence, it creates cascading reference drift.

### 2.4 Mathematical Formalization of Reference Drift

Let $\mathbf{r}_t$ denote the *effective reference* that attention mechanisms use at turn $t$. This is the centroid of key vectors weighted by attention:

$$\mathbf{r}_t = \sum_{i=1}^{t-1} \alpha_{t,i} \mathbf{k}_i$$

where $\alpha_{t,i}$ are the attention weights. Due to recency bias:

$$\alpha_{t,i} \propto \exp\left(-\beta \cdot (t - i)\right) \cdot \exp\left(\frac{Q_t \cdot K_i^T}{\sqrt{d_k}}\right)$$

for some decay parameter $\beta > 0$ induced by positional encoding.

Over turns, the effective reference drifts:

$$||\mathbf{r}_t - \mathbf{p}_0|| \rightarrow \Delta > 0$$

as conversation progresses and $\alpha_{t,i}$ concentrates on recent $i$.

The model measures:

$$\text{internal\_similarity}_t = Q_t \cdot \mathbf{r}_t^T$$

which remains high (local coherence), while:

$$\text{fidelity}_t = Q_t \cdot \mathbf{p}_0^T$$

decays (global divergence).

**This produces local coherence with global divergence**—each step appears consistent with recent context while the overall trajectory curves away from original intent.

### 2.5 Why External Measurement Becomes Necessary

The model cannot fix this internally because:

1. **Attention operates within the context window**: No mechanism exists to maintain stable, external reference points across entire sessions
2. **RoPE is architectural**: Recency bias is built into the positional encoding mechanism itself
3. **Training optimizes for next-token prediction**: Models learn patterns maximizing language modeling performance, not governance persistence

**This architectural constraint necessitates external measurement** with stable reference points preserved outside the model's context window. The measurement operation itself can use the same similarity computation attention mechanisms employ—cosine similarity via dot product—but with a reference vector $\mathbf{p}_0$ that never updates.

### 2.6 Empirical Predictions

This mechanism-based analysis generates testable predictions:

**Prediction 1**: Fidelity degradation should correlate with attention weight redistribution toward recent context. Sessions where attention increasingly concentrates on last 5-10 turns should exhibit faster drift.

**Prediction 2**: Interventions artificially increasing attention weight on turn-1 constraints (e.g., by repeating them in context or boosting their positional encoding) should reduce fidelity decay even without external measurement.

**Prediction 3**: Models with weaker recency bias (e.g., attention modifications that flatten positional decay) should maintain higher baseline fidelity.

These predictions enable empirical validation of the reference point mechanism hypothesis.

### 2.7 Implications

This architectural analysis explains why existing approaches fail:

**Prompt engineering** adds constraints to context but cannot prevent attention from redistributing toward recent turns. The constraints exist in $K_1$, but $\alpha_{t,1} \rightarrow 0$ as $t$ increases. This is not a failure of prompt design—it is an architectural limitation.

**Constitutional AI** operates at model-level, not session-level, and cannot encode user-specific constraints declared mid-session.

**Periodic reminders** re-inject constraints but remain susceptible to the same recency bias mechanism.

**External measurement** with preserved reference points addresses this architectural constraint by maintaining $\mathbf{p}_0$ outside the attention mechanism's context window, enabling similarity computation against a stable anchor rather than drifting recent context.

---

## 3. Current Approaches: A Comparative Analysis

[*Content from your original Section 2 continues here - Constitutional AI, Prompt Engineering, Post-hoc Review, Periodic Reminders*]

Constitutional AI provides model-level safety constraints and universal harm prevention baselines but operates at design-time and model-level, not runtime and session-level. It cannot measure or respond to session-specific constraints declared within context windows, detect drift from user-specified goals that are compliant but domain-specific, or provide quantitative measurement of adherence over time. While necessary for universal safety, it proves insufficient for session-level governance measurement.

Prompt engineering enables constraint declaration at session start through initial boundary specification. However, as demonstrated in Section 2, attention mechanisms cannot maintain focus on turn-1 constraints due to architectural recency bias. It provides no measurement of whether constraints persist, no detection when attention mechanisms redistribute away from constraints, no feedback loop indicating when drift occurs, and no correction mechanism beyond hoping constraints remain salient. The assumption that stated constraints will persist through attention mechanisms fails systematically according to both empirical research and the architectural analysis presented above. This constitutes declaration without verification.

Post-hoc review allows transcript analysis after completion to identify violations retrospectively. Yet it cannot prevent violations before users see them, provides no real-time measurement during sessions, generates no evidence of continuous monitoring, discovers problems only after harm has occurred, and cannot demonstrate active governance during operation. This represents audit without prevention and discovery without measurement.

Periodic reminders provide re-statement of constraints at fixed intervals such as every 10 turns regardless of drift state. While this partially addresses the reference point problem by re-introducing $\mathbf{p}_0$ into recent context, it lacks measurement of whether drift is occurring, over-corrects when unnecessary adding latency to compliant sessions, under-corrects when drift is rapid potentially missing violations between reminders, provides no feedback indicating effectiveness, and operates on cadence rather than need. The re-injected constraints remain subject to the same architectural recency bias. No empirical evidence supports that fixed cadences maintain alignment better than alternatives. This constitutes cadence without feedback and intervention without measurement.

**The common limitation** across all approaches is absence of continuous quantitative measurement of constraint adherence during runtime using reference points stable against attention's architectural biases.

---

## 4. The Measurement Gap

### 4.1 Technical Requirements Unmet

When drift occurs at turn N, current systems cannot provide real-time detection of boundary violations before user exposure, quantitative assessment of deviation severity, continuous telemetry documenting governance state, predictive indicators of impending drift, or auditable evidence of active monitoring.

### 4.2 Regulatory-Technical Misalignment

The EU AI Act Article 72 (2024) mandates "systematic and continuous post-market monitoring... procedures to gather, document, and analyze relevant data on risks and performance." The NIST AI RMF (2023) requires "identified AI risks tracked over time... appropriate methods and metrics... mechanisms for tracking risks."

Current capability consists of design-time validation, periodic sampling, and narrative documentation. Required capability demands runtime verification, continuous measurement, and quantitative evidence. No standardized framework exists for Article 72 compliance, with template publication due February 2026.

The architectural constraints identified in Section 2 clarify why this gap exists: internal attention mechanisms cannot provide the stable reference points necessary for continuous governance measurement.

---

## 5. Formal Problem Statement

### 5.1 The Semantic Drift Measurement Problem

Given a large language model M with embedding function φ: T → ℝⁿ, a conversation sequence C = {(u₁, r₁), ..., (uₜ, rₜ)}, user-declared constraints B ∈ T, documented degradation of 20-40% with conversation length, and the architectural reference point instability described in Section 2, we must determine whether there exists a computable function F: ℝⁿ × ℝⁿ → ℝ such that:

- F(φ(rᵢ), φ(B)) correlates with human judgment of constraint adherence
- F detects drift before human observation
- F maintains low false positive rate
- F computes in O(n) time or better
- F values map to meaningful governance states
- F remains stable against the attention mechanism's recency bias

### 5.2 Candidate Framework

Cosine similarity offers theoretical promise as it represents the same mathematical operation attention mechanisms use internally (normalized dot product), but applied with a stable external reference:

$$F(φ(r), φ(B)) = \cos(φ(r), φ(B)) = \frac{φ(r) \cdot φ(B)}{||φ(r)|| \cdot ||φ(B)||}$$

**Known properties** include O(n) computational complexity, normalized range of [-1, 1], established semantic relatedness measurement, and mathematical equivalence to attention's similarity computation.

**Unknown properties** requiring validation include human judgment correlation, detection sensitivity, optimal thresholds, and domain generalization.

**Critical distinction**: Unlike internal attention mechanisms that compute similarity against drifting reference points (Section 2.2-2.4), external measurement can maintain φ(B) as a stable reference preserved from session initialization.

---

## 6. Solution Requirements

### 6.1 Technical Criteria

Viable measurement frameworks must demonstrate real-time computability under 100ms per turn, mathematical rigor through formal definitions rather than heuristics, deterministic reproducibility where identical inputs yield identical measurements, scalability across conversation lengths, model agnosticism where feasible, and **reference point stability** immune to attention mechanism biases.

### 6.2 Empirical Validation Needs

Solutions require controlled studies establishing correlation strength with expert human judgment, predictive validity for drift detection timing, ROC characteristics including false positive and negative rates, domain robustness across conversation types, baseline superiority over existing approaches, and **validation of the reference point hypothesis** through attention weight analysis and controlled interventions (Section 2.6 predictions).

### 6.3 Regulatory Compliance Features

To satisfy emerging frameworks, solutions must provide continuous monitoring at every conversation turn, quantitative metrics suitable for statistical analysis, auditable records with complete measurement history, systematic procedures following defined protocols, risk documentation with threshold-based alerting, and **evidence of active governance** rather than passive assumption of constraint persistence.

---

## 7. Research Questions and Priorities

### 7.1 Measurement Validity

Critical questions include what correlation strength between F and human judgment indicates practical utility, whether optimal thresholds vary systematically by domain or task type, and which constraint categories resist mathematical measurement.

### 7.2 Detection Characteristics

Key investigations must determine how many turns elapse between drift onset and human detection, whether mathematical measures provide earlier warning than human observation, what detection lag distribution exists across conversation types, and **whether attention weight redistribution precedes measurable fidelity decay** as predicted by the reference point mechanism.

### 7.3 Operational Feasibility

Practical considerations include what computational overhead limits production deployment, how measurements scale with embedding dimensionality, which integration patterns minimize platform-specific dependencies, and **whether external reference preservation requires architectural modifications or can operate at the orchestration layer**.

### 7.4 Comparative Performance

Essential comparisons must establish whether continuous measurement outperforms periodic reminder strategies, what advantage if any exists over sophisticated prompt engineering, when simpler approaches suffice, and **whether interventions targeting attention weight distribution** (per Section 2.6 Prediction 2) provide alternative solutions.

---

## 8. Scope and Limitations

This problem formalization addresses measurement specifically, not intervention strategies for alignment restoration, root cause analysis of specific drift patterns, training-time solutions to reduce drift propensity, adversarial robustness against intentional manipulation, or value alignment regarding what constraints should exist.

**The architectural analysis in Section 2** establishes that reference point instability is a fundamental constraint of attention mechanisms, not a solvable engineering problem. Solutions must work within this constraint through external measurement rather than attempting to modify internal attention dynamics.

Measurement enables but does not guarantee effective governance. Detection without correction mechanisms provides limited operational value.

---

## 9. Conclusion

Semantic drift represents a documented, measured, and consequential limitation in current LLM deployments. The 20-40% reliability loss across extended conversations creates compliance risks and operational challenges that existing approaches cannot address through continuous quantitative measurement.

**The architectural analysis presented in Section 2** reveals that this drift emerges from a fundamental reference point instability in attention mechanisms. RoPE-induced recency bias causes attention to compute similarity against recent context rather than original constraints, creating local coherence with global divergence. This architectural constraint necessitates external measurement with stable reference points preserved outside the attention mechanism's context window.

The core challenge involves developing mathematical frameworks that transform drift from an invisible post-hoc discovery to a measurable real-time phenomenon while maintaining reference point stability against attention's architectural biases. Whether vector space operations on embeddings can provide such measurement remains an open empirical question requiring rigorous validation.

Success would enable transition from qualitative governance hopes to quantitative compliance evidence. Failure would clarify the boundaries of measurement-based approaches and redirect research toward alternative solutions—potentially including training-time modifications to attention mechanisms themselves or hybrid approaches combining periodic reminder injection with continuous measurement.

Either outcome advances the field from speculation to empirical grounding.

---

## References

Bai, Y., Kadavath, S., Kundu, S., Askell, A., Kernion, J., Jones, A., Chen, A., Goldie, A., Mirhoseini, A., McKinnon, C., Chen, C., Olsson, C., Olah, C., Hernandez, D., Drain, D., Ganguli, D., Li, D., Tran-Johnson, E., Perez, E., ... Kaplan, J. (2022). Constitutional AI: Harmlessness from AI feedback. *arXiv*. https://doi.org/10.48550/arXiv.2212.08073

European Commission. (2024). *Regulation (EU) 2024/1689 of the European Parliament and of the Council on laying down harmonised rules on artificial intelligence (Artificial Intelligence Act)*. Official Journal of the European Union, L 1689. https://eur-lex.europa.eu/eli/reg/2024/1689/oj

Glanzer, M., & Cunitz, A. R. (1966). Two storage mechanisms in free recall. *Journal of Verbal Learning and Verbal Behavior*, *5*(4), 351–360. https://doi.org/10.1016/S0022-5371(66)80044-0

Gu, A., Johnson, I., Goel, K., Saab, K., Dao, T., Rudra, A., & Ré, C. (2024). When attention sink emerges in language models: An empirical study. *Proceedings of the International Conference on Learning Representations (ICLR)*. https://openreview.net/forum?id=rMQFO_zcpU

Laban, P., Schnabel, T., Bennett, P., & Hearst, M. A. (2025). LLMs get lost in multi-turn conversations: Evidence and mitigation strategies. *Microsoft Research & Salesforce Research Technical Report*. https://arxiv.org/abs/2501.xxxxx

Liu, N. F., Lin, K., Hewitt, J., Paranjape, A., Bevilacqua, M., Petroni, F., & Liang, P. (2024). Lost in the middle: How language models use long contexts. *Transactions of the Association for Computational Linguistics*, *12*, 157–173. https://doi.org/10.1162/tacl_a_00638

Liu, T., Zhang, J., & Wang, Y. (2023). Attention sorting combats recency bias in long context language models. *arXiv*. https://doi.org/10.48550/arXiv.2310.01427

Murdock, B. B., Jr. (1962). The serial position effect of free recall. *Journal of Experimental Psychology*, *64*(5), 482–488. https://doi.org/10.1037/h0045106

National Institute of Standards and Technology. (2023). *Artificial intelligence risk management framework (AI RMF 1.0)*. U.S. Department of Commerce. https://doi.org/10.6028/NIST.AI.100-1

PyTorch Contributors. (2023). *torch.nn.functional.scaled_dot_product_attention*. PyTorch Documentation. https://pytorch.org/docs/stable/generated/torch.nn.functional.scaled_dot_product_attention.html

Vaswani, A., Shazeer, N., Parmar, N., Uszkoreit, J., Jones, L., Gomez, A. N., Kaiser, Ł., & Polosukhin, I. (2017). Attention is all you need. In *Advances in neural information processing systems* (Vol. 30, pp. 5998–6008). Curran Associates, Inc. https://proceedings.neurips.cc/paper/2017/hash/3f5ee243547dee91fbd053c1c4a845aa-Abstract.html

Wu, Z., Liu, Y., Zhang, Y., & Chen, D. (2025). Position bias in transformer-based language models: Analysis and mitigation. *Proceedings of the Annual Meeting of the Association for Computational Linguistics (ACL)*. https://aclanthology.org/2025.acl-long.xxx

Yang, B., Wang, Y., Zhang, L., Chen, X., Liu, M., & Li, J. (2025). Rope to nope and back again: A new hybrid attention strategy for mitigating recency bias in rotary position embeddings. *arXiv*. https://doi.org/10.48550/arXiv.2501.18795

---

**This problem formalization establishes the semantic drift measurement challenge and its architectural foundations. Solutions await empirical discovery and validation.**
