# k4-core

Core crate: types, infrastructure, and WebSocket client.

## Modules

| Module | Purpose |
|--------|---------|
| `types/` | Enums, market data structs (`Bookticker`, `Trade`, etc.), trading structs, symbol utils |
| `config` | JSON config deserialization (`AppConfig`, `ConnectionConfig`) |
| `shm` | `ShmMdStore<T>` — POSIX shared memory ring buffer |
| `udp` | `UdpSender` / `UdpReceiver` — async UDP with rkyv serialization |
| `ws/` | `WsConnection` (auto-reconnect) + `RedundantWsClient` (N-way redundancy) |
| `latency` | `LatencyCollector` — histogram-based (10us bins, p50/p90/p99) |
| `dedup` | `UpdateIdDedup` (monotonic) + `UuidDedup` (hash table for Bybit) |
| `cpu_affinity` | Thread-to-core pinning for low-latency dedup |
| `error` | `K4Error` — domain-specific error types via thiserror |
| `time_util` | `now_us()`, `now_ns()`, `now_ms()`, `monotonic_us()` |
| `logging` | `init_logging()` — tracing + daily file rotation |
