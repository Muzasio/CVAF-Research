# Methodology

This document describes the methodology used for both the CVAF detection framework and the CVAF-DS dataset.

---

## Detection methodology

### Signal fusion

The detection layer combines a forensic heuristic rule layer with two statistical anomaly-scoring components.

The resulting signals are combined into a single score and evaluated against an adaptive threshold. The threshold is based on the observed score distribution rather than a single fixed cutoff.

This allows the detection threshold to account for changes in the distribution of observed activity.

### Explainability

Each alert includes a feature-attribution breakdown calculated from the fused detection function.

The attribution is calculated from the combined decision rather than from an individual detection component. This provides information about the signals that contributed to the final alert.

Explainability is included as part of the detection workflow because the framework is intended for forensic analysis, where analysts need to examine the basis of a detection rather than only its final classification.

### Evaluation

The project separates live monitoring from ground-truth-based evaluation.

#### Live monitoring

Live monitoring provides operational detection output while the system is running. This output does not have independently labelled ground truth for every event.

For this reason, precision, recall, and F1 are not calculated from live monitoring output as standalone evaluation results.

#### Offline evaluation

Offline evaluation uses independently recorded ground truth and is the basis for reported precision, recall, and F1 measurements.

The evaluation data is divided into three categories:

* **Matched split:** Contains activity patterns that the heuristic layer was designed to identify. This measures performance on patterns represented in the rule design.
* **Held-out split:** Contains patterns excluded from the rule-development process. This is used as the primary measure of generalization.
* **Evasion split:** Contains modified attack patterns designed to reduce the likelihood of detection. This provides a separate measurement of missed detections under changed activity patterns.

Ground truth is created independently from detector output. The detector's own alerts are therefore not used to define the labels against which it is evaluated.

---

## Dataset collection methodology

### Environment

CVAF-DS is collected using a two-role virtual machine environment consisting of a victim VM and an attacker VM.

The environment uses KVM virtualization with full Ubuntu Linux guests and native Linux auditd collection.

Full virtual machines are used because the research focuses on host-level evidence. Container-based environments share the host kernel and cannot produce some types of guest-level evidence, such as an actual kernel module load inside an independent guest kernel.

### Attack scenario design

Each attack scenario is mapped to a named MITRE ATT&CK technique.

Scenarios are grouped according to their effect on persistent VM state.

Scenarios that do not significantly modify persistent state can be repeated without resetting the environment. Scenarios that modify persistent state, such as persistence mechanisms or state tampering, use a snapshot-and-revert process so that repeated runs begin from the same environment state.

### Ground truth

Ground truth is recorded independently by the process responsible for executing each attack scenario.

The record includes execution timestamps and the outcome of the scenario. Detector output is not used to create or modify the ground-truth labels.

This separation is maintained both during dataset collection and during subsequent detection evaluation.

### Benign activity

Low-privilege background activity runs on the victim VM during collection.

The background activity is interleaved with attack scenarios so that the collected data contains both normal and attack-related activity rather than consisting only of attack events.

### Dataset design references

The design of CVAF-DS is informed by criteria discussed in previous intrusion-detection dataset research, including realism, diversity, documentation, and reproducibility.

These references are used as design guidance. They are not used to claim that CVAF-DS has been formally validated against, or is equivalent to, an established benchmark.

### Cross-dataset comparison

Existing public Linux intrusion-detection datasets are used for separate comparative evaluation.

A format-translation step converts the source datasets into the representation required by the CVAF evaluation workflow. The translated data remains separate from the CVAF-DS ground truth.

### Dataset integrity

The same hash-chaining mechanism used for live evidence acquisition is also applied to the collected dataset files.

This provides an integrity record for the dataset files and allows changes to the recorded files to be detected through the corresponding hash relationships.

### Project-specific collection details

The complete scenario list, virtual-machine configuration, collection procedure, ground-truth structure, and observations from the collection environment are documented in [`dataset.md`](dataset.md).

This document focuses on the general methodology, while `dataset.md` describes the specific implementation and collection environment used by CVAF-DS.

---

## Methodological scope

The methodology does not make the following claims:

* CVAF-DS is equivalent to or formally validated against an established intrusion-detection benchmark.
* Live monitoring output provides standalone precision, recall, or F1 measurements without independent ground truth.
* A statistical anomaly-detection component is an unmodified implementation of its reference algorithm when the implementation uses an approximation.

Specific implementation details and limitations are documented in [`architecture.md`](architecture.md).
