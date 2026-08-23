# Security model

## A security model without a master key

PoD's confidentiality rests on **garbled circuits with non-colluding parties** — a garbler and an evaluator that must both defect to compromise a session.

The distinction that matters is what a worst case costs you. Threshold FHE systems protect a shared decryption key; if enough key-holders collude, **every transaction ever recorded becomes readable retroactively**. PoD has no such key. A compromise is scoped to a session, not to your history.

**Privacy that does not accumulate a single point of catastrophic failure.**

For the underlying primitive, see [Garbled Circuits](../how-coti-works/advanced-topics/garbled-circuits.md).
