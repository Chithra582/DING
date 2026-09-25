---
name: tree-diff-analyzer
description: Compute differences between working directory, staging area, and historical commit tree snapshots.
---

# Tree Diff Analyzer Skill

## Overview
Performs delta calculations across tree snapshots and current working directory files to report repository status.

## Operations
1. Recursively traverses tree snapshot objects and aligns them with working directory files.
2. Categorizes file states into unmodified, modified, added, deleted, and untracked.
3. Generates unified diffs between different commit trees or between commit and working tree.
4. Feeds status summaries to the CLI and user feedback channels.
