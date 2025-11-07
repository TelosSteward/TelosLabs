# Semantic Drift in Large Language Models: Problem Formalization

**TELOS Labs - Theoretical Research**

## Abstract

Large language models exhibit systematic alignment degradation across extended conversations, with empirical studies documenting 20-40% reliability loss as context accumulates. This degradation, termed semantic drift, occurs predictably due to transformer architecture properties: primacy decay, recency bias, and attention redistribution. Current approaches lack continuous quantifiable measurement of governance persistence during runtime, creating a gap between emerging regulatory requirements for "systematic and continuous" monitoring and available technical capabilities. This paper formalizes the semantic drift measurement problem, analyzes current approach limitations, and establishes requirements for viable mathematical frameworks.

## 1. The Semantic Drift Phenomenon

### 1.1 Empirical Evidence

Semantic drift is documented across multiple independent research efforts. Laban et al. (2025) demonstrate that models lose instruction tracking in multi-turn conversations with systematic boundary violations after 20 or more turns. Liu et al. (2024) show middle context loses salience in what they term "Lost in the Middle," with attention decay in the 20-30 turn range. Wu et al. (2025) identify position bias causing primacy effect decay where early token influence diminishes over time. Gu et al. (2024) reveal attention sinks that redistribute focus from constraints, causing governance instructions to lose attention weight.

The aggregate finding across these studies is 20-40% reliability loss in extended dialogues across all major model families.

### 1.2 Architectural Origins

Drift emerges from fundamental transformer properties. Primacy decay causes session-start constraints to exert strong initial influence that progressively weakens. Recency bias leads recent tokens to capture disproportionate attention, overriding earlier instructions. Attention competition forces governance tokens to compete with content tokens for limited capacity. Context window effects create non-uniform attention weighting that causes middle content to lose influence.

These mechanisms mirror documented cognitive phenomena in human memory (Murdock, 1962; Glanzer & Cunitz, 1966), suggesting deep architectural parallels.

### 1.3 Operational Impact

Organizations experience drift through multiple manifestations. Compliance risk emerges when governance boundaries erode silently during extended sessions. User frustration builds as "I already told you not to do that" becomes a recurring refrain requiring repeated re-statement of ignored constraints. Silent degradation occurs gradually rather than suddenly, making real-time detection difficult. Post-hoc discovery means violations are identified only through transcript review after sessions complete.

## 2. Current Approaches: A Comparative Analysis

Constitutional AI provides model-level safety constraints and universal harm prevention baselines but operates at design-time and model-level, not runtime and session-level. It cannot measure or respond to session-specific constraints declared within context windows, detect drift from user-specified goals that are compliant but domain-specific, or provide quantitative measurement of adherence over time. While necessary for universal safety, it proves insufficient for session-level governance measurement.

Prompt engineering enables constraint declaration at session start through initial boundary specification. However, it provides no measurement of whether constraints persist, no detection when attention mechanisms redistribute away from constraints, no feedback loop indicating when drift occurs, and no correction mechanism beyond hoping constraints remain salient. The assumption that stated constraints will persist through attention mechanisms fails systematically according to empirical research. This constitutes declaration without verification.

Post-hoc review allows transcript analysis after completion to identify violations retrospectively. Yet it cannot prevent violations before users see them, provides no real-time measurement during sessions, generates no evidence of continuous monitoring, discovers problems only after harm has occurred, and cannot demonstrate active governance during operation. This represents audit without prevention and discovery without measurement.

Periodic reminders provide re-statement of constraints at fixed intervals such as every 10 turns regardless of drift state. This approach lacks measurement of whether drift is occurring, over-corrects when unnecessary adding latency to compliant sessions, under-corrects when drift is rapid potentially missing violations between reminders, provides no feedback indicating effectiveness, and operates on cadence rather than need. No empirical evidence supports that fixed cadences maintain alignment better than alternatives. This constitutes cadence without feedback and intervention without measurement.

The common limitation across all approaches is absence of continuous quantitative measurement of constraint adherence during runtime.

## 3. The Measurement Gap

### 3.1 Technical Requirements Unmet

When drift occurs at turn N, current systems cannot provide real-time detection of boundary violations before user exposure, quantitative assessment of deviation severity, continuous telemetry documenting governance state, predictive indicators of impending drift, or auditable evidence of active monitoring.

### 3.2 Regulatory-Technical Misalignment

The EU AI Act Article 72 (2024) mandates "systematic and continuous post-market monitoring... procedures to gather, document, and analyze relevant data on risks and performance." The NIST AI RMF (2023) requires "identified AI risks tracked over time... appropriate methods and metrics... mechanisms for tracking risks."

Current capability consists of design-time validation, periodic sampling, and narrative documentation. Required capability demands runtime verification, continuous measurement, and quantitative evidence. No standardized framework exists for Article 72 compliance, with template publication due February 2026.

## 4. Formal Problem Statement

### 4.1 The Semantic Drift Measurement Problem

Given a large language model M with embedding function φ: T → ℝⁿ, a conversation sequence C = {(u₁, r₁), ..., (uₜ, rₜ)}, user-declared constraints B ∈ T, and documented degradation of 20-40% with conversation length, we must determine whether there exists a computable function F: ℝⁿ × ℝⁿ → ℝ such that:

F(φ(rᵢ), φ(B)) correlates with human judgment of constraint adherence. F detects drift before human observation. F maintains low false positive rate. F computes in O(n) time or better. F values map to meaningful governance states.

### 4.2 Candidate Framework

Cosine similarity offers theoretical promise:

```
F(φ(r), φ(B)) = cos(φ(r), φ(B)) = (φ(r) · φ(B)) / (||φ(r)|| · ||φ(B)||)
```

Known properties include O(n) computational complexity, normalized range of [-1, 1], and established semantic relatedness measurement. Unknown properties requiring validation include human judgment correlation, detection sensitivity, optimal thresholds, and domain generalization.

## 5. Solution Requirements

### 5.1 Technical Criteria

Viable measurement frameworks must demonstrate real-time computability under 100ms per turn, mathematical rigor through formal definitions rather than heuristics, deterministic reproducibility where identical inputs yield identical measurements, scalability across conversation lengths, and model agnosticism where feasible.

### 5.2 Empirical Validation Needs

Solutions require controlled studies establishing correlation strength with expert human judgment, predictive validity for drift detection timing, ROC characteristics including false positive and negative rates, domain robustness across conversation types, and baseline superiority over existing approaches.

### 5.3 Regulatory Compliance Features

To satisfy emerging frameworks, solutions must provide continuous monitoring at every conversation turn, quantitative metrics suitable for statistical analysis, auditable records with complete measurement history, systematic procedures following defined protocols, and risk documentation with threshold-based alerting.

## 6. Research Questions and Priorities

### 6.1 Measurement Validity

Critical questions include what correlation strength between F and human judgment indicates practical utility, whether optimal thresholds vary systematically by domain or task type, and which constraint categories resist mathematical measurement.

### 6.2 Detection Characteristics

Key investigations must determine how many turns elapse between drift onset and human detection, whether mathematical measures provide earlier warning than human observation, and what detection lag distribution exists across conversation types.

### 6.3 Operational Feasibility

Practical considerations include what computational overhead limits production deployment, how measurements scale with embedding dimensionality, and which integration patterns minimize platform-specific dependencies.

### 6.4 Comparative Performance

Essential comparisons must establish whether continuous measurement outperforms periodic reminder strategies, what advantage if any exists over sophisticated prompt engineering, and when simpler approaches suffice.

## 7. Scope and Limitations

This problem formalization addresses measurement specifically, not intervention strategies for alignment restoration, root cause analysis of specific drift patterns, training-time solutions to reduce drift propensity, adversarial robustness against intentional manipulation, or value alignment regarding what constraints should exist.

Measurement enables but does not guarantee effective governance. Detection without correction mechanisms provides limited operational value.

## 8. Conclusion

Semantic drift represents a documented, measured, and consequential limitation in current LLM deployments. The 20-40% reliability loss across extended conversations creates compliance risks and operational challenges that existing approaches cannot address through continuous quantitative measurement.

The core challenge involves developing mathematical frameworks that transform drift from an invisible post-hoc discovery to a measurable real-time phenomenon. Whether vector space operations on embeddings can provide such measurement remains an open empirical question requiring rigorous validation.

Success would enable transition from qualitative governance hopes to quantitative compliance evidence. Failure would clarify the boundaries of measurement-based approaches and redirect research toward alternative solutions.

Either outcome advances the field from speculation to empirical grounding.

## References

Bai, Y., et al. (2022). Constitutional AI: Harmlessness from AI Feedback. *arXiv preprint arXiv:2212.08073*.

European Commission. (2024). *Regulation (EU) 2024/1689 on Artificial Intelligence (AI Act)*. Official Journal of the European Union.

Glanzer, M., & Cunitz, A. R. (1966). Two storage mechanisms in free recall. *Journal of Verbal Learning and Verbal Behavior, 5*(4), 351-360.

Gu, A., et al. (2024). When Attention Sink Emerges in Language Models. *Proceedings of ICLR*.

Laban, P., et al. (2025). LLMs Get Lost in Multi-Turn Conversations. *Microsoft Research & Salesforce Research Technical Report*.

Liu, N. F., et al. (2024). Lost in the Middle: How Language Models Use Long Contexts. *Transactions of ACL, 12*, 157-173.

Murdock, B. B. (1962). The serial position effect of free recall. *Journal of Experimental Psychology, 64*(5), 482-488.

National Institute of Standards and Technology. (2023). *AI Risk Management Framework (AI RMF 1.0)*. U.S. Department of Commerce.

Wu, Z., et al. (2025). Position Bias in Transformer-Based Language Models. *Proceedings of ACL*.

*This problem formalization establishes the semantic drift measurement challenge. Solutions await empirical discovery and validation.*