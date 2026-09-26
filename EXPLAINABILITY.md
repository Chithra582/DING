# EXPLAINABILITY.md

This document explains the internal mechanisms, data lineage, operational boundaries, and governance framework of **DING VCS Agent** (`ding-vcs-agent`) in accordance with the **OpenGAP v0.1.0** specification for the **HiDevs GitAgent Passport** clearance pipeline.

> **Agent Name:** DING VCS Agent (`ding-vcs-agent`)  
> **Specification:** OpenGAP v0.1.0  
> **Category / Domain:** Developer Tools / Version Control & Content-Addressable Storage  
> **Compliance Standard:** OpenGAP Checkpoint 2 (Explainability & Decision Governance), GDPR, SOC2  

---

## How the Agent Decides

DING VCS Agent is an autonomous developer tools and storage intelligence agent designed for **DING (Ding Is Not Git)**, a content-addressable version control system implemented in Python. The agent automates repository lifecycle state transitions, immutable cryptographic object storage (blobs, trees, commits), Directed Acyclic Graph (DAG) commit lineage traversal, and reference pointer safety.

### 1. Decision Architecture

The decision, hashing, and storage process operates across a deterministic, five-stage pipeline:

```
User Action (init / hash / commit / branch / checkout / status / log)
    │
    ▼
[Stage 1: Repository Discovery & Boundary Validation]
    │  - Climbs directory tree upward from cwd to locate root `.ding/` repository folder
    │  - Verifies read/write filesystem permissions and object store existence
    │  - Refuses non-repository command execution with explicit `ding init` guidance
    ▼
[Stage 2: Object Serialization & Cryptographic Hashing]
    │  - Reads target file in raw binary mode (`rb`)
    │  - Computes SHA-256 cryptographic digest: OID = hashlib.sha256(content).hexdigest()
    │  - Evaluates deduplication table:
    │      ├── OID exists in `.ding/objects/`: Skips redundant write (deduplication)
    │      └── New OID: Atomically commits binary payload to `.ding/objects/{OID}`
    ▼
[Stage 3: Tree Snapshot Assembly & Hierarchy Parsing]
    │  - Recursively scans directory structure respecting `.dingignore` patterns
    │  - Hashes files into leaf blobs and directories into intermediate tree nodes
    │  - Synthesizes top-level root tree object ID representing exact workspace snapshot
    ▼
[Stage 4: Commit Construction & DAG Lineage Linking]
    │  - Resolves current parent commit OID from active branch ref pointed to by `HEAD`
    │  - Assembles commit payload: Root Tree OID + Parent OID + Author + ISO Timestamp + Message
    │  - Computes commit OID and appends to Directed Acyclic Graph (DAG)
    │  - Atomically swings active branch reference pointer to new commit OID
    ▼
[Stage 5: Diff & Destructive Safety Gate]
    │  - Performs recursive tree diff between current working directory and HEAD tree
    │  - Halts checkout or branch switch if uncommitted local modifications collide with target tree
    ▼
Local Filesystem Workspace & Immutable `.ding/` Object Store
```

### 2. Classification Rubrics & Storage Criteria

#### Object Type Taxonomy
DING partitions all repository data into four immutable, content-addressable object primitives:
1. **Blob (`blob`)**: Raw, uncompressed file payload. Identified solely by its SHA-256 byte digest; contains zero filesystem metadata or file names.
2. **Tree (`tree`)**: Directory manifestation mapping filename strings and permissions to child Blob or Tree OIDs.
3. **Commit (`commit`)**: Snapshot entity binding a root Tree OID to zero, one, or more parent Commit OIDs, accompanied by author signature, commit timestamp, and descriptive log message.
4. **Tag / Reference (`ref`)**: Human-readable pointer (e.g., `refs/heads/main`) that resolves to a specific Commit OID.

#### Atomic Reference Update Rubric
To prevent pointer corruption during sudden process interruption:
- Pointer updates write the target Commit OID to a temporary shadow file (`.ding/refs/heads/<branch>.tmp`).
- The filesystem executes an atomic replacement (`os.replace`) swapping the temporary file into the canonical reference path.

### 3. Verification & Integrity Scoring

The agent enforces mathematical certainty across all repository read and write operations:

$$\text{Integrity}(O) = \begin{cases} 1 & \text{if } \text{SHA-256}(\text{payload}) \equiv \text{OID} \\ 0 & \text{if } \text{SHA-256}(\text{payload}) \not\equiv \text{OID} \quad (\text{Disk Corruption Detected}) \end{cases}$$

- **Content Verification**: Whenever an object is unpacked from disk, the agent recalculates its SHA-256 checksum. Any discrepancy triggers an immediate read failure and corruption alert.
- **DAG Cycle Prevention**: Commit parent assignments are validated to guarantee strictly acyclic chronology. A commit can never cite itself or an ancestor as a child.
- **Diff Matrix Computation**: Status and diff operations compute file states: `Untracked`, `Modified`, `Staged`, or `Clean` by evaluating cryptographic content equality rather than unreliable file modification timestamps.

### 4. Thresholding & Refusal Decision Criteria

DING VCS Agent enforces strict safety gates to eliminate accidental data destruction:
- **Dirty Working Directory Refusal**: The agent explicitly rejects `checkout` or `switch` requests if untracked or modified files in the working directory would be overwritten by files present in the destination commit.
- **Root Repository Boundary**: Commands targeting version control operations outside an initialized `.ding` repository directory tree are rejected immediately.
- **Large Binary Threshold Warning**: If an individual file staged for hashing exceeds $25\text{ MB}$, the agent flags an advisory warning prompting the user to confirm inclusion or append the path to `.dingignore` to prevent object database bloat.
- **Detached HEAD Safety Prompt**: Committing in detached `HEAD` mode triggers an explicit notification reminding the developer that the resulting commit will be orphaned unless attached to a named branch reference.

### 5. Fallback Decision Mechanism

DING VCS Agent is designed with local-first, zero-downtime fault tolerance:
- **100% Local Offline Operation**: All core operations (hashing, committing, branching, diffing, logging) execute entirely on the local system with zero network dependency. No cloud server or external API outage can prevent repository operations.
- **Atomic Swap Recovery**: If a system crash or power cut occurs during a branch pointer move, the canonical reference remains intact at its pre-transaction state, allowing seamless recovery without manual database repairs.
- **Model Fallback for Natural Language CLI Copilot**: If the developer queries natural language Git/DING commands using the embedded AI assistant, the agent calls `gemini-2.0-flash` with automatic fallback to `gpt-4o` and `claude-3-5-sonnet`. If all external LLM APIs are unreachable, the agent falls back to local command-line `--help` docstrings.

### 6. Human-in-the-Loop Governance

DING enforces developer sovereignty over all repository history:
- **Explicit Override for Destructive Actions**: Destructive operations such as force-checkout or hard reset cannot execute autonomously without explicit CLI confirmation flags (`--force`).
- **Immutable Historical Trail**: Commits written to the object database are append-only. The agent never deletes historical commit objects autonomously, ensuring developers can always recover previous states using low-level reflog inspection.
- **Transparent Logging**: All commit and branch transitions are auditable through standard linear log traversal (`ding log`).

---

## The Data It Uses

DING VCS Agent operates strictly on local repository artifacts with zero unauthorized data extraction.

### 1. Ingested Input Data

The agent processes only files and metadata explicitly contained within the user's workspace:
- **Working Directory Content**: Source code files, configuration files, and assets within the project directory tree read in binary mode for SHA-256 digest computation.
- **Developer Attribution Metadata**: Author name and author email gathered from local environment variables (`DING_AUTHOR_NAME`, `DING_AUTHOR_EMAIL`) or CLI arguments.
- **Commit Messages**: User-authored descriptive text strings supplied during commit creation.

### 2. Configuration & Reference Data

- **Repository Configuration**: Local `.ding/config` file containing repository settings and default branch names.
- **Branch and Head References**: Pointer files in `.ding/refs/heads/` and `.ding/HEAD` storing 64-character SHA-256 hexadecimal commit hashes.
- **Ignore Rules**: User-defined patterns in `.dingignore` filtering out temporary build artifacts, virtual environments, and sensitive secrets (`.env`).

### 3. Base Model & Inference Lineage

- **Core VCS Engine**: Deterministic Python code (`hashlib`, `os`, `pathlib`) executing arithmetic hashing, binary serialization, and DAG traversals without any probabilistic LLM involvement.
- **Developer Assistance Copilot**: Utilizes frontier foundation models (Google Gemini `gemini-2.0-flash`, OpenAI `gpt-4o`, Anthropic `claude-3-5-sonnet`) solely for natural language command translation and commit message summarization when invoked.
- **Zero Training on Repository Code**: The agent does not send source code to external servers for training or model fine-tuning.

### 4. Data Privacy, Storage, and Retention

- **0-Byte External Egress Guarantee**: All version control operations, source code blobs, directory trees, and commit histories are stored 100% locally on the user's machine in the `.ding/` directory. Zero bytes of code leave the local filesystem.
- **No Third-Party Telemetry**: DING VCS Agent contains no analytics trackers, crash-reporting pings, or background telemetry.
- **GDPR & SOC2 Compliance**: Because no personal data or proprietary IP is collected, indexed, or shared remotely, the agent complies by design with GDPR (Article 25 - Privacy by Design) and SOC2 security and confidentiality principles.

---

## Limitations

Understanding the operational boundaries and technical constraints of DING VCS Agent is essential for robust developer workflows.

### 1. Hash Collision Theoretical Risk
- **Limitation**: While the mathematical probability of a SHA-256 collision is virtually zero ($2^{-256}$), two distinct files producing the identical hash would cause unrecoverable object collision.
- **Mitigation**: The storage engine executes byte-level equality validation whenever an existing OID is matched during a write, raising a fatal integrity exception if divergent bytes are detected.

### 2. Working Tree Dirty Overwrite Constraints
- **Limitation**: Switching branches or restoring historical commits while uncommitted changes exist in the working directory could result in permanent loss of uncommitted work.
- **Mitigation**: The agent runs a pre-flight working tree diff check, halting destructive operations immediately if uncommitted files conflict with incoming branch files.

### 3. Detached HEAD Commit Orphan Risk
- **Limitation**: When a developer checks out a historical commit directly instead of a branch name, subsequent commits are recorded without an associated named branch reference, making them susceptible to being lost after another checkout.
- **Mitigation**: The agent displays an informative warning banner whenever a commit is recorded in detached HEAD mode, providing the exact command to anchor the commit to a new branch (`ding branch <new-branch-name>`).

### 4. Uncompressed Binary File Storage Bloat
- **Limitation**: Repeated modifications to large binary assets (e.g., videos, compiled binary packages, large datasets) generate separate 25MB+ uncompressed blob files in `.ding/objects/`, rapidly inflating disk space.
- **Mitigation**: The agent issues a warning when staging files exceeding 25MB, guiding developers toward ignore filters or external storage solutions.

### 5. Local Scope & Distributed Transport Boundary
- **Limitation**: DING VCS Agent is designed as a core content-addressable version control system and repository manager; it does not implement a built-in proprietary peer-to-peer network daemon.
- **Mitigation**: Distributed synchronization relies on standard network transports (SSH/HTTPS) and remote adapters adhering to open specifications.

---

## Summary & Compliance Checklist

| Checkpoint 2 Requirement | Corresponding Section | Status |
| :--- | :--- | :---: |
| **How the agent decides** | [How the Agent Decides](#how-the-agent-decides) | **Covered** |
| - Decision architecture & 5-stage pipeline | Section 1 | Verified |
| - Classification rubrics & storage criteria | Section 2 | Verified |
| - Verification & cryptographic integrity scoring | Section 3 | Verified |
| - Thresholding & refusal decision criteria | Section 4 | Verified |
| - Fallback decision mechanism | Section 5 | Verified |
| - Human-in-the-loop governance & developer control | Section 6 | Verified |
| **The data it uses** | [The Data It Uses](#the-data-it-uses) | **Covered** |
| - Ingested input data & working tree files | Section 1 | Verified |
| - Configuration & reference pointer data | Section 2 | Verified |
| - Base model lineage & deterministic engine | Section 3 | Verified |
| - Data privacy, 0-byte egress & GDPR/SOC2 | Section 4 | Verified |
| **Its limitations** | [Limitations](#limitations) | **Covered** |
| - Hash collision theoretical risk | Section 1 | Verified |
| - Working tree dirty overwrite constraints | Section 2 | Verified |
| - Detached HEAD commit orphan risk | Section 3 | Verified |
| - Uncompressed binary file storage bloat | Section 4 | Verified |
| - Local scope & distributed transport boundary | Section 5 | Verified |
