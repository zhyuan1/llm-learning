# Spanner: TrueTime and External Consistency

Source: https://docs.cloud.google.com/spanner/docs/true-time-external-consistency

## Overview
TrueTime is a highly available distributed clock that provides time services to applications across all Google servers. It enables applications to generate monotonically increasing timestamps with the guarantee that if timestamp T' finishes being generated before timestamp T starts being generated, the application can compute a timestamp T that is guaranteed to be greater than T'. This guarantee applies across all servers and all timestamps.

Spanner uses TrueTime's properties to assign timestamps to transactions. Each transaction receives a timestamp reflecting when Spanner considers the transaction to have occurred. Because Spanner uses multi-version concurrency control, the timestamp ordering guarantees allow Spanner clients to perform consistent reads across the entire database—even spanning multiple Cloud regions—without blocking writes.

## External Consistency
- Transactions behave as if executed sequentially.
- The serial order matches observable commit order.
- Clients do not observe a later transaction without first observing an earlier completed one.
- Spanner remains close to a single-machine DB semantic model despite multi-datacenter execution.

## Timestamps and MVCC
- Writes create immutable versions stamped with transaction timestamps.
- Snapshot reads at a timestamp return the newest version before that timestamp.
- Proper timestamp assignment is the key bridge from MVCC to externally consistent behavior.

## FAQ Highlights
- External consistency is stronger than serializability.
- Spanner also provides linearizability for the relevant single-object case.
- It does not provide eventual consistency, but does provide stale reads.
