# Concurrency

## Concurrency that keeps up with users

Multiple transfers, mints, and burns from the same account can be in flight simultaneously. Each is tracked independently by request id, and a monotonic nonce guarantees results always apply in the correct order. **No queue, no serialisation, no waiting for one transfer to clear before starting the next.**

For how asynchronous private operations settle in general, see [Async private operations](../privacy-on-demand/async-private-operations.md).
