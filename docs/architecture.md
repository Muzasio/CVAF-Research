# Architecture

This document expands on the high-level architecture shown in the main README. It describes the seven-phase CVAF pipeline and its relationship to the CVAF-DS dataset collection process.

**Scope:** This document describes components that have been reviewed and confirmed against the current implementation. Components that exist in the project but have not yet been reviewed are marked **[unreviewed]**. These are listed in the [Open items](#open-items) section.

---

## Pipeline overview

```text
auditd Collection
      |
      v
Chain-of-Custody Hashing
      |
      v
Semantic Normalization
      |
      v
Hybrid Anomaly Detection
      |
      v
Knowledge Graph Construction
      |
      v
Interactive Visualization
      |
      v
Forensic Reporting
```

## Phase 1: Evidence Acquisition

Linux `auditd` is the primary evidence source. A background reader continuously reads the audit log, with `journalctl` available as an alternate source.

Each incoming line is passed to the chain-of-custody layer before downstream processing. This preserves the original arrival order of the records used by the system.

## Phase 2: Chain-of-Custody

Each incoming line is included in a SHA-256 hash chain:

```text
H_i = SHA256(H_{i-1} + line_i)
```

The resulting records are written to a local append-only ledger. An in-memory rolling window is also maintained for faster integrity checks during an active session.

Changes to a previously recorded event can therefore be detected when the corresponding hash relationships no longer match.

Acquisition timing is measured separately from downstream detection. This allows the cost of hashing and ledger writing to be reported independently rather than being combined with later processing stages.

**Current limitation:** integrity verification currently operates on the in-memory rolling window rather than replaying the complete on-disk ledger. Full-ledger replay verification is planned but is not currently implemented.

## Phase 3: Semantic Normalization

Raw audit records are parsed and prepared for the detection stages. The current processing includes timestamp extraction, noise filtering, and feature extraction.

The extracted features include values such as:

* Time of day
* External IP indication
* Event rate

The project is designed to use an OCSF-aligned representation for normalized events.

**[Unreviewed]** The implementation responsible for the OCSF mapping has not yet been reviewed as part of this documentation pass. The description above is limited to normalization steps that have been confirmed.

## Phase 4: Hybrid Anomaly Detection

The detection layer combines several sources of information:

* **Forensic heuristic rules:** Pattern-based rules for known indicators. Rules include severity weights and cooldown periods to reduce repeated alerts for the same activity.
* **Statistical anomaly scoring:** An Extended Isolation Forest variant and a streaming approximation of Robust Random Cut Forest. Both operate on a relatively small feature set.
* **Explainable attribution:** A Shapley-value-style calculation is used to estimate which signals contributed to a detection decision.

These signals are combined using fixed fusion weights and an adaptive threshold. The values are measured configuration parameters rather than being changed for individual test cases.

The processing stages are also timed separately to support throughput measurements for the individual components.

**Current limitation:** the streaming anomaly-forest component is an approximation of the reference algorithm. It uses buffer-based rebuilding rather than a persistent incremental structure, so it is documented as an approximation rather than as a direct implementation of the reference algorithm.

## Phase 5: Knowledge Graph Construction

Detected events are converted into entities such as users, processes, files, and network endpoints. Relationships between these entities are then represented as a graph.

The graph provides a way to examine relationships between events and entities rather than relying only on a flat event list.

**[Unreviewed]** Graph-level statistics and metrics are implemented as a separate component but have not yet been reviewed for this documentation pass.

## Phase 6: Interactive Visualization

The detection state and entity graph are presented through two interfaces:

* A terminal-based operational view for live monitoring
* A 3D force-directed graph for investigating collected events

The visualization supports filtering, path tracing, and inspection of individual entities.

## Phase 7: Forensic Reporting

Session data is compiled into structured machine-readable output and a human-readable forensic report.

The reports include information such as:

* Chain-of-custody status
* Severity distribution
* Detection alerts
* Per-alert explanations

**Current limitation:** live-session reporting does not currently calculate independent false-positive and false-negative counts. These values are therefore not presented as live detection-performance measurements.

Ground-truth-based evaluation is performed separately through the offline evaluation process described in the methodology.

---

## Relationship to CVAF-DS

The CVAF-DS collection process uses the same evidence-acquisition and chain-of-custody mechanisms described in Phases 1 and 2.

Data is collected from real KVM virtual machines in a controlled lab environment using MITRE ATT&CK-mapped attack scenarios and recorded ground truth. Using the same acquisition path allows the dataset to exercise the evidence-collection components used by CVAF rather than relying on a separate collection implementation.

Cross-dataset comparison with existing public Linux intrusion-detection datasets is handled through a format-translation step. Each external dataset is converted into the normalized representation used by CVAF-DS before being processed.

The external datasets are kept as separate evaluation inputs and are not merged with the CVAF-DS ground truth.

The scenario definitions, ground-truth process, and infrastructure observations from the collection process are documented separately in `dataset.md`.

---

## Open items

The following components are present in the project but have not yet been reviewed for this documentation pass:

* OCSF-aligned schema mapping implementation
* Graph-level statistics and metrics computation
* Field-accuracy measurement component
* Docker-based collection orchestration script and its container definition
* Secondary MITRE mapping module. Its relationship to the primary cross-dataset adapter has not yet been confirmed.

This list will be updated as each component is reviewed and the corresponding sections are confirmed or revised.
