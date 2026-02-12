# k4-md

Market data modules using a generic pipeline architecture.

## Architecture

Each exchange provides `build(config) -> Vec<StreamDef>`. The `GenericMd` engine
in `pipeline.rs` handles SHM creation, channels, dedup tasks, and WS connections.

## Module Registry

`registry::create_md_module(config)` dispatches by `config.exchange`:

| Exchange | Module | WS URL | Data Types |
|----------|--------|--------|------------|
| Binance | `binance::build()` | stream.binance.com, stream-sbe.binance.com, fstream.binance.com | BBO, AggTrade, Trade, Depth5 |
| OKX | `okx::build()` | ws.okx.com:8443 | BBO, Trade, Depth5 |
| Bitget | `bitget::build()` | ws.bitget.com | BBO, Trade, Depth5 |
| Bybit | `bybit::build()` | stream.bybit.com | BBO, Trade, Depth5 |
| UDP | `udp::UdpMd` | N/A (UDP socket) | All types |

## Shared Infrastructure

| File | Purpose |
|------|---------|
| `pipeline.rs` | `StreamDef` descriptor + `GenericMd` engine |
| `dedup_worker.rs` | Generic dedup loop with `ProductShmStores` |
| `ws_helper.rs` | WebSocket stream launchers (text + binary) |
| `json_util.rs` | Shared JSON parsing helpers |

## Exchange-Specific Details

### Binance

- Dual protocol: JSON (aggTrade) + SBE binary (BBO, trade, depth) for spot
- SBE uses Decimal128 encoding
- Separate dedup threads for JSON and SBE streams

### OKX

- Symbol conversion: `BTCUSDT` to `BTC-USDT` (spot) / `BTC-USDT-SWAP` (swap)
- Routes by `arg.channel` field

### Bitget

- Trade messages are batched (array), processed in reverse order
- `instType`: `"SPOT"` or `"USDT-FUTURES"`

### Bybit

- Futures trade IDs are UUIDs, hashed via xxHash64 for dedup
- Depth5 from incremental `orderbook.50` via `OrderBook<50>` state machine
- Routes by `topic` field prefix

### UDP

- Receives from other modules' UdpSender, writes directly to SHM (no dedup)
