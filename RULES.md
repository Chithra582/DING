# Rules: DING VCS Agent

These are immutable operational boundaries and safety constraints for DING VCS Agent.

## MUST ALWAYS
1. **MUST ALWAYS verify repository root prior to mutations**: Confirm `.ding` repository initialization before running any storage or commit operations.
2. **MUST ALWAYS compute deterministic SHA-256 object hashes**: Ensure object keys match file payload hashes before writing to `.ding/objects/`.
3. **MUST ALWAYS guard uncommitted working tree changes**: Abort checkout and branch operations if uncommitted changes would be overwritten.
4. **MUST ALWAYS preserve immutable commit DAG history**: Append new commits with valid parent hashes; never rewrite or truncate commit lineage without explicit force flags.
5. **MUST ALWAYS enforce local-first privacy**: Keep all code blobs, trees, and commit histories stored locally on the user machine with zero external network telemetry.

## MUST NEVER
1. **MUST NEVER overwrite existing objects in the object store**: Content-addressable storage files are strictly write-once, read-many.
2. **MUST NEVER delete untracked files during tree checkouts without explicit confirmation**: Protect user workspace from accidental data destruction.
3. **MUST NEVER allow dangling commit references**: Every branch pointer update must reference a valid, verified commit OID in the object database.
4. **MUST NEVER transmit repository code or hashes over unencrypted or external channels**: Maintain absolute offline security and code privacy.
