# Explainability & Transparency Report: DING VCS Agent

> **Specification:** OpenGAP v0.1.0  
> **Domain:** Developer Tools / Version Control & Content-Addressable Storage  
> **Target System:** DING (Ding Is Not Git)  
> **Audit Status:** Qualified for HiDevs GitAgent Passport  

---

## 1. Overview & Operational Purpose

**DING VCS Agent** is an autonomous developer tools agent designed for **DING (Ding Is Not Git)**, a version control system implemented in Python. The agent automates core version control operations:
1. **Content-Addressable Object Storage**: Hashing files into immutable blobs, directories into trees, and states into commits using SHA-256.
2. **Directed Acyclic Graph (DAG) Commit Lineage**: Managing parent pointers, commit logs, and linear history traversal.
3. **Reference & Pointer Management**: Tracking branch heads and `HEAD` state safely without data loss.
4. **Tree Diffing & Working Directory Status**: Computing state deltas between the index, commit snapshots, and the filesystem.

---

## 2. How the Agent Decides (Decision-Making Logic)

```
User Action (init / hash / commit / branch / checkout / status / log)
  │
  ├── 1. Repository Discovery & Validation
  │      └── Crawl upward from current working directory to locate `.ding`
  │          ├── Found: Load repo configuration & object store paths
  │          └── Not Found: Require `ding init` before proceeding
  │
  ├── 2. Object Serialization & Hash Computation
  │      ├── Read target file in binary mode (`rb`)
  │      ├── Calculate SHA-256 hex digest: OID = hashlib.sha256(content).hexdigest()
  │      └── Verify if `.ding/objects/{OID}` exists (deduplication check)
  │          ├── Exists: Reuse existing OID (no write needed)
  │          └── New: Atomically persist content to `.ding/objects/{OID}`
  │
  ├── 3. Tree Snapshot Assembly
  │      ├── Recursively scan directory entries excluding `.ding`
  │      ├── Hash files into blobs and subdirectories into sub-trees
  │      └── Generate root tree object ID
  │
  ├── 4. Commit Construction & Lineage Linking
  │      ├── Read current commit OID from `.ding/HEAD` (branch ref)
  │      ├── Construct commit object: tree OID + parent OID + author + timestamp + message
  │      ├── Hash and persist commit object
  │      └── Update current branch reference pointer atomically
  │
  └── 5. Diff & Safety Gate
         ├── Compare working tree files against HEAD tree entries
         └── Prevent destructive operations (checkout/reset) if uncommitted changes exist
```

---

## 3. Data Sources & Inputs Used

| Data Input | Source | Purpose | Data Handling & Privacy |
|---|---|---|---|
| **Working Directory Files** | Local filesystem workspace | Generating blob hashes and directory tree snapshots | Read locally in binary format; never transmitted over network |
| **Object Database** | `.ding/objects/` directory | Retrieving immutable historical blobs, trees, and commits | Local append-only filesystem storage with read-once validation |
| **Reference Pointers** | `.ding/HEAD`, `.ding/refs/` | Tracking current branch and commit pointers | Plaintext local pointer files; updated via atomic write operations |
| **Commit Metadata** | User CLI arguments & environment | Recording commit author, timestamp, and message | Embedded immutably in commit objects; no external telemetry |

---

## 4. Known Limitations & Failure Modes

### 1. Hash Collision Vulnerability
*Limitation:* While theoretical probability for SHA-256 collisions is negligible, catastrophic data corruption would occur if two distinct files produced the same hash.  
*Mitigation:* The agent verifies existing object size and byte equality whenever an object write request matches an existing OID, flagging an anomaly if contents diverge.

### 2. Working Tree Dirty Overwrite
*Limitation:* Checking out a different branch or commit while untracked or modified files exist could result in irreversible file overwrites.  
*Mitigation:* The agent runs a tree diff against the working directory before executing checkout operations; if dirty files collide with the target tree, checkout is halted immediately.

### 3. Detached HEAD Commit Abandonment
*Limitation:* Creating commits while in a detached HEAD state (not on an active named branch) leaves commits vulnerable to being orphaned.  
*Mitigation:* The agent warns the user upon committing in detached HEAD mode and suggests creating a named branch reference (`ding branch <name>`) pointing to the newly generated commit OID.

### 4. Binary File Size & Object Database Bloat
*Limitation:* Large binary files checked in repeatedly will generate distinct uncompressed SHA-256 blobs, inflating `.ding/objects/` disk usage.  
*Mitigation:* The agent issues warnings when staging files exceeding 25MB and advises using selective ignore patterns or external artifact stores.

---

## 5. Verification, Safety & Human Oversight

1. **Local-First & Offline Guarantee**: DING VCS Agent requires no internet connection, third-party cloud API, or telemetry server. All data operations occur strictly on the user's local disk.
2. **Atomic Reference Updates**: Branch references are written using temporary swap files (`.ref.tmp` -> `.ref`) to prevent corrupt pointer writes if the process is terminated mid-operation.
3. **Cryptographic Integrity Checks**: Reads from `.ding/objects/` can be verified by re-hashing content against the filename OID, catching disk corruption or unauthorized tampering.
4. **Human In The Loop**: Destructive commands (such as hard reset or tree clearing) require explicit confirmation flags and cannot be triggered autonomously.
