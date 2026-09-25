# DING — Ding Is Not Git

[![OpenGAP Spec 0.1.0](https://img.shields.io/badge/OpenGAP-0.1.0-blue.svg)](https://opengitagent.org)
[![GitAgent Passport](https://img.shields.io/badge/GitAgent%20Passport-Ready-brightgreen.svg)](https://app.hidevs.xyz/passport/submit)
[![Category](https://img.shields.io/badge/Category-Developer%20Tools-blueviolet.svg)](https://app.hidevs.xyz/passport/submit)
[![Compliance](https://img.shields.io/badge/Compliance-GDPR%20%7C%20SOC2-orange.svg)](EXPLAINABILITY.md)

_A complete Version Control System built from scratch in Python._

**DING** is an version control system developed as part of **OpenCode by IIITA**.  
The goal of this project is to build a VCS that implements **most of the features** git

DING focuses on teaching core concepts such as content-addressable storage, snapshots, commits, branching, diffs, and logs - all through readable, well-structured Python code.

If you've ever wondered _“What actually happens when I run `git commit`?”_, DING is for you.

---

## Objectives

- Build a **fully functional VCS**, not a mock or toy model
- Keep the implementation **simple, readable, and hackable**
- Encourage contributors to **learn by building**, not just using tools

---

## Features

### Repository Initialization

- Initialize a DING repository inside any directory
- Creates internal metadata and object storage
- Inspired by Git’s `.git` structure

### Object Storage

- Content-addressable storage using hashes
- Stores:
  - Blobs (file contents)
  - Trees (directory snapshots)
  - Commits (project states)

### Commits & History

- Snapshot-based commits
- Commit metadata (message, parent, timestamp)
- Linear history traversal
- Commit logs similar to `git log`

### Branching

- Lightweight branch references
- HEAD pointer management
- Switch between branches safely

### Diff & Status

- Show changes between commits
- Working directory vs last commit
- Track modified, added, and deleted files

---

## Getting Started

Before starting - check out our [CONTRIBUTING.md](https://github.com/opencodeiiita/DING/blob/main/CONTRIBUTING.md) to get a refresher on contribution and general workflow.

---

## Tech Stack

### **Language**

- **Python 3.x**

### **External Dependencies**

_NONE, build from scratch just using std_

---

## Project Structure

```
.
├── setup.py
└── ugit
    ├── base.py
    ├── cli.py
    ├── data.py
    ├── diff.py
    └── remote.py
```

---

## GitAgent Passport Qualification

This repository is fully compliant with the **OpenGAP Spec 0.1.0** standard and qualified for the **HiDevs GitAgent Passport**:

- **Checkpoint 1 (Validate):** Verified OpenGAP spec 0.1.0 compliance via [`agent.yaml`](agent.yaml), [`SOUL.md`](SOUL.md), [`skills/`](skills/), and [`tools/`](tools/).
- **Checkpoint 2 (Explain):** Comprehensive 5-section transparency report in [`EXPLAINABILITY.md`](EXPLAINABILITY.md) detailing content-addressable storage algorithms, commit DAG construction, local-first zero-telemetry guarantee, and failure mode mitigations.
- **Checkpoint 3 (Export):** Cross-framework export compatibility tested across OpenAI SDK, CrewAI, Claude Code, and Lyzr.
- **Target Category:** **`Developer Tools`** (Version Control Systems & Content-Addressable Storage).
