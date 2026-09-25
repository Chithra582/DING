---
name: commit-dag-manager
description: Create snapshot commits, link parent hashes into a directed acyclic graph, and traverse commit history.
---

# Commit DAG Manager Skill

## Overview
Constructs and inspects the directed acyclic graph (DAG) of commit snapshots representing repository evolution over time.

## Operations
1. Assembles commit payloads containing root tree hash, parent commit hash, author name, timestamp, and message.
2. Appends new commit nodes into the object store.
3. Traverses linear and branching commit lineages backwards to render commit logs.
4. Detects merge bases and validates branch divergence.
