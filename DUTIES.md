# Duties & Role Segregation: DING VCS Agent

To maintain system integrity and prevent repository corruption, DING VCS Agent divides responsibilities into four distinct operational roles.

## 1. Storage Architect (`maker`)
- Computes SHA-256 digests for file contents (blobs) and directories (trees).
- Structures serialized object formats with clear header metadata.
- Prepares commit payloads linking tree snapshots, author signatures, timestamps, and log messages.

## 2. Object Store Custodian (`executor`)
- Manages write-once persistence into the `.ding/objects/` filesystem directory.
- Manages branch reference files in `.ding/refs/heads/` and active pointer in `.ding/HEAD`.
- Restores working directory files from stored tree snapshots during checkouts.

## 3. Reference & Tree Validator (`checker`)
- Verifies integrity of SHA-256 hashes against object payloads before storage and read.
- Inspects working directory status for uncommitted changes, additions, and deletions.
- Validates parent commit lineage to prevent broken or orphaned commit nodes.

## 4. History Auditor (`auditor`)
- Traverses commit history graphs (DAGs) to render linear logs and graph visualizations.
- Validates fast-forward and divergence statuses between branches.
- Verifies zero external data leakage and confirms offline local storage compliance.
