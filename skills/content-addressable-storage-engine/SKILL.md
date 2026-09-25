---
name: content-addressable-storage-engine
description: Hash file contents into blobs, serialize directory trees, and manage immutable object storage.
---

# Content-Addressable Storage Engine Skill

## Overview
Manages the foundational storage layer of DING. Every entity is stored as an immutable object keyed by its SHA-256 hash.

## Operations
1. Reads source files in binary mode and calculates SHA-256 hex digests.
2. Writes content to `.ding/objects/{hash}` using atomic file writes.
3. Reconstructs directory trees by recording permissions, entry types, filenames, and child hashes.
4. Verifies object integrity by comparing file payload against its hash identifier.
