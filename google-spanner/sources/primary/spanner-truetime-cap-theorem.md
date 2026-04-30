# Spanner, TrueTime and the CAP Theorem

Source: https://research.google/pubs/spanner-truetime-and-the-cap-theorem/

## Publication Details
- Author: Eric Brewer
- Organization: Google
- Year: 2017

## Abstract
Spanner is Google's highly available global-scale distributed database. It provides strong consistency for all transactions. This combination of availability and consistency over the wide area is generally considered impossible due to the CAP Theorem. We show how Spanner achieves this combination and why it is consistent with CAP. We also explore the role that TrueTime, Google's globally synchronized clock, plays in consistency for reads and especially for snapshots that enable consistent and repeatable analytics.

## Notes
- Official Google Research publication page.
- Useful for separating “Google production conditions” from the abstract CAP model.
