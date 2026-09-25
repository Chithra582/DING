---
name: branch-ref-controller
description: Manage named branch references, manipulate HEAD pointer, and ensure safe branch transitions.
---

# Branch Ref Controller Skill

## Overview
Governs lightweight reference pointers in `.ding/refs/heads/` and the active `.ding/HEAD` pointer.

## Operations
1. Reads and updates branch reference files safely using atomic writes.
2. Resolves symbolic references (e.g. `ref: refs/heads/main`) to commit OIDs.
3. Manages detached HEAD states and transitions between branches.
4. Safeguards against uncommitted changes before switching working directory states.
