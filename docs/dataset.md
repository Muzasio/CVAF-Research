# CVAF-DS: Dataset Collection

This document describes the CVAF-DS collection environment, attack scenarios, ground-truth structure, and operational observations recorded during dataset collection.

The dataset is maintained separately from this public repository. Raw evidence and dataset-scale statistics are not included here.

---

## Environment

* **Host:** KVM/QEMU/libvirt on Linux.
* **Victim VM:** Full-boot Ubuntu 22.04 LTS guest with native Linux auditd collection.
* **Attacker VM:** Isolated VM used to generate attack traffic. Auditd is not installed on the attacker VM because forensic collection is performed on the victim system.
* **Networking:** Both VMs operate on an isolated virtual network and can communicate with each other. SSH key-based automation is used for controlled interaction between the systems.
* **Ground truth:** A canonical ground-truth record is maintained by the attacker-side collection process. Entries are created after an attack action has been executed and the corresponding audit evidence has been confirmed.

---

## Scenario set

Each scenario is associated with a MITRE ATT&CK technique.

| Scenario                       | MITRE ATT&CK technique                                                 | Tactic                        |
| ------------------------------ | ---------------------------------------------------------------------- | ----------------------------- |
| SSH brute-force                | T1110.001: Brute Force, Password Guessing                              | Credential Access             |
| Privilege escalation           | T1548.003: Abuse Elevation Control Mechanism, Sudo                     | Privilege Escalation          |
| Passwd/shadow tampering        | T1098: Account Manipulation                                            | Persistence                   |
| Credential dumping             | T1003.008: OS Credential Dumping                                       | Credential Access             |
| New user / backdoor account    | T1136.001: Create Account, Local Account                               | Persistence                   |
| SSH configuration tampering    | T1098.004: Account Manipulation, SSH Authorized Keys                   | Persistence                   |
| Kernel module load/unload      | T1547.006: Boot or Logon Autostart Execution, Kernel Modules           | Persistence / Defense Evasion |
| Reconnaissance / enumeration   | T1082 / T1046: System Information Discovery / Network Service Scanning | Discovery                     |
| Cloud metadata access          | T1552.005: Unsecured Credentials, Cloud Instance Metadata API          | Credential Access             |
| Log tampering / anti-forensics | T1070.002: Indicator Removal, Clear Linux or Mac System Logs           | Defense Evasion               |

Scenarios are executed repeatedly with variation in timing and parameters. Benign background activity runs during collection so that attack activity is mixed with normal system activity.

---

## Ground truth definition

Ground truth is recorded by the attacker-side automation independently of the CVAF detection output.

Each entry identifies the scenario and repetition and records information including:

* MITRE ATT&CK technique
* Attacker host
* Victim host
* Start and end timestamps
* Reference to the supporting evidence file or files

The ground-truth record is created from the execution process and its timestamps. Detector alerts are not used to create or modify these labels.

This record provides the reference used for subsequent detection evaluation.

---

## Benign activity

Low-privilege background activity runs continuously on the victim system during collection.

The activity includes routine operations such as:

* File access
* System information queries
* Normal process activity
* Occasional legitimate privileged commands

Background activity is collected during the same sessions and environment as the attack scenarios. This keeps normal and attack-related activity within the same collection period rather than producing two separate datasets with different environmental conditions.

---

## Cloud metadata scenario

A real cloud instance metadata endpoint is not available on a standalone KVM virtual machine because such endpoints are normally provided by the cloud infrastructure.

For the cloud metadata scenario, the victim VM was configured with a link-local address and a minimal HTTP responder representing the expected structure of a cloud metadata service.

The scenario is based on MITRE ATT&CK technique T1552.005, Unsecured Credentials: Cloud Instance Metadata API.

This provides a controlled way to exercise metadata-access detection without requiring a live cloud account.

---

## Operational observations

Collection was performed on real virtual machines using the native audit subsystem rather than generated or pre-processed audit logs. Several implementation and infrastructure issues were identified during the collection process.

### Audit-log parsing

The auditd version used in the environment concatenated certain interpreted log fields without a separator. This affected the ability of standard audit-log query tools to filter some record types by event key.

For those records, the collection and analysis workflow used the raw audit log instead of relying on the affected query path.

### Snapshot reverts and persistent automation

Automation that needs to remain available after a VM snapshot revert cannot depend on temporary or ephemeral storage inside the guest.

Scripts required after a revert are therefore stored outside locations that are reset with the snapshot.

### Snapshot operations and audit state

Audit watch rules were observed to stop producing expected evidence after some VM snapshot operations, even when the rules still appeared to be loaded.

The collection procedure accounts for this by restarting the audit subsystem, reloading the rules, allowing the system to settle, and performing a test action before starting an attack scenario.

The test action is used to confirm that the expected audit evidence is actually being generated.

### Network evidence

For some network-related events, the record containing the detection key does not contain the destination address itself. The destination information can appear in a related record.

The normalization and evidence-matching process therefore considers related audit records instead of treating each tagged record as a complete representation of the network operation.

### Remote background processes

Background services started through SSH required proper session detachment to remain active after the interactive session ended.

Process-name-based termination was also unreliable for some privileged services started remotely. Identifying the socket or port used by the service provided a more reliable way to manage those processes.

These observations are included because they affect the reproducibility of the collection environment and the handling of audit evidence during dataset generation.

---

## Dataset material not included here

The following material is maintained separately from this repository:

* Per-scenario run counts
* Total event counts
* Dataset storage size
* Collection duration
* Scenario success and failure statistics
* Detailed validation results
* Raw audit evidence
* Ground-truth data files
* Attack and collection automation scripts

Dataset-scale statistics and validation results are documented with the dataset research publication rather than presented separately from their supporting methodology.

---

For the CVAF framework architecture, see [`architecture.md`](architecture.md).

For the detection and dataset evaluation methodology, see [`methodology.md`](methodology.md).
