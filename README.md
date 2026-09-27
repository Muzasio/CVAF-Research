# CVAF Research: Cloud VM Workload Forensics

*An ongoing research project on host-level forensic evidence collection, chain-of-custody preservation, and hybrid anomaly detection for cloud IaaS Linux workloads.*
**Live site:** https://muzasio.github.io/CVAF-Research/

> **Research status:** This repository documents the current CVAF research project, including its architecture, methodology, dataset design, and selected research material. Raw dataset files and private experimental material are maintained separately. The CVAF framework source code is not included in this repository and is not intended for public release. See [Repository scope](#repository-scope) for details.

---

## Table of contents

* [Overview](#overview)
* [CVAF: the framework](#cvaf-the-framework)
* [CVAF-DS: the dataset](#cvaf-ds-the-dataset)
* [Architecture](#architecture)
* [Research objectives](#research-objectives)
* [Methodology](#methodology)
* [Research status](#research-status)
* [Selected results](#selected-results)
* [Repository scope](#repository-scope)
* [Citation](#citation)
* [Academic context](#academic-context)
* [Contact](#contact)

---

## Overview

CVAF (Cloud VM Audit Forensics) focuses on host-level digital forensics in cloud and IaaS environments. The project looks at how host activity can be collected continuously while preserving the integrity of the evidence from the point of acquisition.

Cloud provider logs can provide useful information about activity at the infrastructure and control-plane level, but they do not provide complete visibility into activity taking place inside a guest virtual machine. CVAF focuses on this host-level layer.

The project has two related parts:

* **CVAF:** A seven-phase forensic framework covering evidence acquisition, chain-of-custody hashing, semantic normalization, hybrid anomaly detection, knowledge-graph construction, visualization, and forensic reporting.
* **CVAF-DS:** A controlled dataset collected from full-boot KVM virtual machines and used for framework evaluation and comparison with existing public datasets.

The framework is the main focus of the research. CVAF-DS provides the controlled data and ground truth needed for evaluation.

---

## CVAF: the framework

CVAF is a research prototype. It combines existing techniques and tools into a single host-level forensic workflow.

The framework uses Linux auditd for evidence collection, SHA-256 hash chaining for evidence integrity, statistical anomaly detection, forensic rules, feature attribution, graph-based analysis, and forensic reporting.

### Pipeline

1. **Evidence acquisition**
   Linux auditd is used to collect kernel-level workload activity.

2. **Chain of custody**
   Incoming audit records are incorporated into a SHA-256 hash chain during acquisition.

3. **Semantic normalization**
   Audit records are parsed and converted into a common representation aligned with the project's OCSF-based data model.

4. **Hybrid detection**
   Statistical anomaly scoring is combined with forensic heuristic rules and an adaptive decision threshold.

5. **Explainability**
   Shapley-style feature attribution is used to identify the signals that contributed to an alert.

6. **Knowledge graph**
   Users, processes, files, network endpoints, and their relationships are represented as graph entities.

7. **Visualization and reporting**
   Detection results are presented through graph exploration, timeline analysis, and structured forensic reports.

### Scope

CVAF is not intended to be:

* A new machine-learning algorithm
* A SIEM replacement
* A cloud-provider monitoring service
* A production-scale security platform

---

## CVAF-DS: the dataset

CVAF-DS is a controlled attack dataset collected in a two-VM laboratory environment with separate victim and attacker roles.

The virtual machines use full Ubuntu 22.04 KVM guests with native Linux auditd collection.

Using full virtual machines allows the collection of host-level evidence that is not available in the same form from container-based environments or syscall-only datasets.

Examples include:

* Kernel module activity
* File integrity events
* Cron-based persistence
* Credential file modification
* Privilege escalation activity
* Instance metadata service access

### Design principles

* Attack scenarios are mapped to MITRE ATT&CK techniques.
* Ground truth is recorded independently from detector output using attacker-side execution records and timestamps.
* Benign background activity is collected alongside attack scenarios.
* Evidence collected during dataset generation uses the same acquisition and chain-of-custody mechanisms used by CVAF.
* Dataset design is informed by reliability criteria discussed in previous intrusion-detection dataset research. These criteria are not presented as a certification or validation claim.

### Relationship to existing benchmarks

ADFA-LD and LID-DS are used as separate inputs for cross-dataset comparison.

Format translation is used to convert their event representations into the normalized representation used for CVAF evaluation. Their data and ground truth remain separate from CVAF-DS.

Scenario definitions, collection details, ground-truth procedures, and observations from the collection environment are documented in [`docs/dataset.md`](docs/dataset.md).

---

## Architecture

CVAF and CVAF-DS share the evidence acquisition and chain-of-custody stages. After that point, the workflows have different purposes.

CVAF processes collected evidence through normalization, detection, analysis, and reporting.

CVAF-DS uses controlled attack scenarios and independently recorded ground truth for evaluation.

### System architecture

![CVAF system architecture](docs/diagrams/cvaf-architecture.svg)

*Figure: CVAF framework and CVAF-DS research workflow.*

### CVAF pipeline

![CVAF seven-phase pipeline](docs/diagrams/cvaf-pipeline.svg)

*Figure: Seven phases of the CVAF framework.*

### CVAF-DS collection

![CVAF-DS collection workflow](docs/diagrams/cvaf-dataset-collection.svg)

*Figure: KVM laboratory environment and CVAF-DS evidence collection process.*

Implementation-level diagrams such as internal module structure and class relationships are not included because the framework source code is private.

---

## Research objectives

The project investigates a host-level forensic pipeline for Linux workloads running in IaaS environments.

The main objectives are:

* Collect host-level evidence from Linux virtual machines.
* Preserve evidence integrity from acquisition onward.
* Normalize audit records into a common representation.
* Combine statistical and rule-based detection methods.
* Provide explanations for detection decisions.
* Represent relationships between forensic entities as a graph.
* Evaluate the framework using controlled attack scenarios and independent ground truth.
* Compare the resulting detection workflow with existing public Linux intrusion-detection datasets.

---

## Methodology

Evidence is collected through native Linux auditd on full-boot KVM virtual machines.

Each incoming record is incorporated into a SHA-256 hash chain during acquisition. The records are then parsed and prepared for the detection stages.

The detection layer combines an Extended Isolation Forest variant, a Robust Random Cut Forest approximation, and forensic heuristic rules. These signals are combined using a fixed fusion configuration and adaptive decision threshold.

Shapley-style attribution is used to identify the features that contributed to individual detection decisions.

Attack scenarios are mapped to MITRE ATT&CK techniques and executed in the controlled virtual-machine environment. Ground truth is recorded independently through attacker-side execution records and timestamps.

Detection metrics such as precision, recall, and F1 are calculated against the independent ground truth.

### Technologies and references

`auditd`
`KVM / libvirt`
`MITRE ATT&CK`
`SHA-256`
`Extended Isolation Forest`
`Robust Random Cut Forest`
`SHAP-style attribution`
`OCSF-aligned schema`

---

## Research status

The current project includes the following research components:

| Area                       | Description                                              |
| -------------------------- | -------------------------------------------------------- |
| Virtual machine laboratory | Two-VM KVM environment with victim and attacker roles    |
| Evidence acquisition       | Native Linux auditd collection                           |
| Chain of custody           | SHA-256 hash chaining during acquisition                 |
| Attack scenarios           | MITRE ATT&CK-mapped controlled scenarios                 |
| Ground truth               | Independent attacker-side execution records              |
| Benign activity            | Background workload collected alongside attack scenarios |
| Dataset validation         | Evidence and ground-truth consistency checks             |
| Cross-dataset evaluation   | ADFA-LD and LID-DS comparison workflow                   |
| Framework evaluation       | CVAF evaluation using CVAF-DS                            |
| Dataset publication        | Separate dataset release and DOI record                  |
| Research publications      | CVAF framework and CVAF-DS papers                        |

Quantitative results are reported with their corresponding evaluation methodology and experimental context.

---

### Forensic report

![CVAF forensic report](docs/diagrams/cvaf-forensic-report.png)

*Figure: Example CVAF forensic report output.*

Detection results are presented together with the evaluation methodology used to obtain them.

---

## Repository scope

This repository contains the public documentation for the CVAF research project.

It includes:

* Project documentation
* Architecture
* Methodology
* Dataset documentation
* Public research figures
* Research context and references

The following are kept outside this repository:

* CVAF implementation source code
* Raw audit logs
* Raw dataset files
* Private experimental artifacts
* Unreleased internal evaluation material

The CVAF framework source code is not part of the public repository and is not intended for public release.

The dataset is maintained separately from the framework source code and can be referenced through its formal research release when available.

---

## Citation

A formal citation will be provided with the associated research publication and dataset DOI.

Until those references are available, the project can be referenced as:

> **CVAF Research: Cloud VM Workload Forensics**
> Cloud VM Audit Forensics framework and CVAF-DS research dataset.

---

## Academic context

This project was developed as a Final Year Project for a BS in Cyber Security.

The work is documented here as a research project focused on host-level cloud workload forensics, evidence integrity, and anomaly detection.

---



<sub>This repository contains the public documentation and research material for the CVAF project.</sub>
