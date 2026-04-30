# Life of Spanner Reads & Writes

Source: https://docs.cloud.google.com/spanner/docs/whitepapers/life-of-reads-and-writes

## Key points
- Data is split by key range and each split is synchronously replicated via Paxos.
- Single-split writes avoid full 2PC and are therefore the cheapest write path.
- Multi-split read-write transactions add coordination, locking, and 2PC cost.
- Read-only transactions avoid locks and can scale well with replicas.
- Strong reads may need freshness coordination with leaders.
- Stale reads can avoid part of that coordination cost.
- Commit wait is paired with TrueTime to ensure externally consistent ordering.

## Why this source matters
This document adds operational detail to the original paper, especially around the concrete cost difference between single-split and multi-split work.