# Barter-rs — Development Guide

## Setup

### Prerequisites

| Requirement | Version | Notes |
|---|---|---|
| Rust toolchain | stable | Pinned via `rust-toolchain.toml` |
| Cargo | bundled with Rust | workspace resolver v3 required |
| Internet access | — | Required at build time for crates.io dependencies |

Barter-rs pins the stable Rust toolchain with the following components:

```toml
# rust-toolchain.toml
[toolchain]
channel = "stable"
components = ["cargo", "clippy", "rust-std", "rustc", "rustfmt"]
```

### Clone and Build

```bash
git clone https://github.com/barter-rs/barter-rs.git
cd barter-rs

# Build all crates in release mode
cargo build --release

# Build a specific crate
cargo build -p barter --release
cargo build -p barter-data --release
```

### Running Tests

```bash
# All crates
cargo test

# Specific crate
cargo test -p barter
cargo test -p barter-data
cargo test -p barter-execution

# Parallel execution (requires cargo-nextest)
cargo nextest run

# With output (useful for integration tests)
cargo test -- --nocapture

# Run a named test
cargo test test_trading_state_update
```

### Code Quality

```bash
# Lint
cargo clippy --all-features -- -D warnings

# Format check
cargo fmt --check

# Format in place
cargo fmt

# Type check without building
cargo check
```

---

## Project Structure

```
barter-rs/
├── Cargo.toml                   # Workspace root — shared dependencies
├── rust-toolchain.toml          # Pinned stable toolchain
├── rustfmt.toml                 # Formatting rules
├── release-plz.toml             # Automated release configuration
│
├── barter/                      # Core trading engine crate (v0.12.x)
│   ├── Cargo.toml
│   ├── src/
│   │   ├── lib.rs               # EngineEvent, Sequence, Timed
│   │   ├── engine/
│   │   │   ├── mod.rs           # Engine struct, Processor impl
│   │   │   ├── run.rs           # sync_run, async_run, sync_run_with_audit
│   │   │   ├── state/
│   │   │   │   ├── mod.rs       # EngineState
│   │   │   │   ├── trading/     # TradingState machine
│   │   │   │   ├── asset/       # AssetState, AssetStates
│   │   │   │   ├── instrument/  # InstrumentState, InstrumentStates
│   │   │   │   ├── order/       # OrderManager
│   │   │   │   ├── position.rs  # Position tracking
│   │   │   │   └── connectivity/ # ConnectivityStates
│   │   │   ├── action/          # CancelOrders, ClosePositions, GenerateAlgoOrders
│   │   │   ├── audit/           # AuditTick, Auditor, StateReplicaManager
│   │   │   ├── clock.rs         # EngineClock, LiveClock, HistoricalClock
│   │   │   ├── command.rs       # Command enum
│   │   │   └── execution_tx.rs  # ExecutionTxMap
│   │   ├── system/
│   │   │   ├── mod.rs           # System, SystemAuxillaryHandles
│   │   │   ├── builder.rs       # SystemBuilder, SystemArgs
│   │   │   └── config.rs        # SystemConfig, InstrumentConfig, ExecutionConfig
│   │   ├── strategy/            # AlgoStrategy, ClosePositionsStrategy interfaces
│   │   ├── risk/                # RiskManager interface + DefaultRiskManager
│   │   ├── statistic/           # TradingSummary, TearSheet, Sharpe, Sortino
│   │   ├── execution/           # ExecutionRequest routing, AccountStreamEvent
│   │   └── backtest/            # Concurrent backtest utilities
│   ├── examples/                # Runnable example binaries
│   └── benches/                 # Criterion benchmarks (backtest throughput)
│
├── barter-data/                 # Market data streaming (v0.11.x)
│   └── src/
│       ├── lib.rs               # MarketStream trait, NoInitialSnapshots
│       ├── exchange/            # Per-exchange Connector impls
│       │   ├── binance/{spot,futures}/
│       │   ├── coinbase/
│       │   ├── okx/
│       │   ├── bybit/
│       │   ├── kraken/
│       │   ├── gateio/
│       │   ├── bitfinex/
│       │   └── bitmex/
│       ├── streams/             # StreamBuilder, DynamicStreams
│       ├── subscription/        # PublicTrades, OrderBooksL1/L2/L3
│       ├── transformer/         # StatelessTransformer, custom order book transformers
│       └── books/               # Local OrderBook maintenance
│
├── barter-execution/            # Order execution (v0.7.x)
│   └── src/
│       ├── lib.rs               # AccountEvent, AccountEventKind
│       ├── client/              # ExecutionClient trait, MockExecutionClient
│       ├── exchange/            # Live exchange execution wrappers
│       ├── order/               # Order, OrderState, OrderRequest types
│       └── trade.rs             # Trade, AssetFees
│
├── barter-instrument/           # Domain types (v0.3.x)
│   └── src/
│       ├── lib.rs               # Keyed, Underlying, Side
│       ├── exchange.rs          # ExchangeId enum (all supported venues)
│       ├── asset/               # Asset, AssetKind, AssetIndex
│       ├── instrument/          # Instrument, InstrumentKind (Spot/Perp/Future/Option)
│       └── index/               # IndexedInstruments, builder
│
├── barter-integration/          # Protocol foundations (v0.10.x)
│   └── src/
│       ├── lib.rs               # Validator, Transformer, Terminal traits
│       ├── protocol/            # StreamParser, WebSocket types
│       ├── channel/             # Tx, UnboundedTx, ChannelTxDroppable
│       ├── stream/              # ExchangeStream, ReconnectingStream
│       └── socket/              # ReconnectingSocket
│
└── barter-macro/                # Procedural macros (v0.2.x)
```

---

## Configuration Reference

### `SystemConfig` (JSON)

The `SystemBuilder` expects a `SystemConfig` deserialisable from JSON. The example config lives at `barter/examples/config/system_config.json`.

```json
{
  "instruments": [
    {
      "exchange": "BinanceSpot",
      "name_exchange": "BTCUSDT",
      "underlying": { "base": "BTC", "quote": "USDT" },
      "quote": { "asset": "USDT" },
      "kind": "Spot",
      "spec": null
    }
  ],
  "executions": [
    {
      "exchange": "BinanceSpot",
      "fees_percent": "0.001",
      "latency_ms": 10
    }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `instruments[].exchange` | `ExchangeId` (string) | Exchange identifier matching `barter-instrument::ExchangeId` enum variant |
| `instruments[].name_exchange` | string | Exchange-native symbol (e.g. `"BTCUSDT"`) |
| `instruments[].underlying` | `{base, quote}` | Asset pair using exchange asset names |
| `instruments[].kind` | `"Spot"` / `"Perpetual"` / `"Future"` / `"Option"` | Instrument type |
| `instruments[].spec` | optional object | Price tick, quantity increment, min order size |
| `executions[].exchange` | `ExchangeId` | Exchange this execution config applies to |
| `executions[].fees_percent` | decimal string | Taker fee rate for mock execution |
| `executions[].latency_ms` | integer | Simulated order round-trip latency |

Source: `barter/src/system/config.rs`

### `SystemBuilder` Options

| Method | Type | Default | Description |
|---|---|---|---|
| `.engine_feed_mode()` | `EngineFeedMode` | `Async` | `Iterator` for backtests (blocking), `Async` for live trading |
| `.audit_mode()` | `AuditMode` | `Disabled` | `Enabled` to receive `AuditTick` stream |
| `.trading_state()` | `TradingState` | `Disabled` | Initial trading state on startup |

Source: `barter/src/system/builder.rs`

### Logging / Tracing

Barter uses `tracing` for structured logging. Initialise with the provided helper:

```rust
use barter::logging::init_logging;
init_logging(); // reads RUST_LOG env var
```

Useful log targets:

| Target | What it logs |
|---|---|
| `barter::engine` | Event processing, TradingState changes |
| `barter_data` | WebSocket connect/disconnect, subscription handshake |
| `barter_execution` | Order submissions, fill confirmations |

---

## Running the Examples

All examples are in `barter/examples/`. They require a `config/system_config.json` in the working directory:

```bash
cd barter-rs/barter

# Paper trading with live market data
cargo run --example engine_sync_with_live_market_data_and_mock_execution_and_audit

# Async engine with historic data
cargo run --example engine_async_with_historic_market_data_and_mock_execution

# Multiple strategies
cargo run --example engine_sync_with_multiple_strategies

# Risk manager order checks
cargo run --example engine_sync_with_risk_manager_open_order_checks

# State replication via audit stream
cargo run --example engine_sync_with_audit_replica_engine_state

# Statistical summary only
cargo run --example statistical_trading_summary

# Concurrent backtests
cargo run --example backtests_concurrent
```

---

## Troubleshooting

### 1. WebSocket connection refused or times out

**Symptoms**: `barter-data` logs `failed to connect` or `SocketError::ConnectTimeout` at startup.

**Causes and fixes**:
- Exchange API may be temporarily unavailable. Check the exchange status page.
- Firewall or VPN blocking outbound WebSocket traffic. Verify outbound port 443 is open.
- Incorrect exchange variant in config (e.g. using `BinanceSpot` when the instrument is on `BinanceFuturesUsd`). Cross-check `ExchangeId` with the instrument kind.

### 2. `IndexedInstruments` panics on `instrument_index_mut`

**Symptoms**: Panic at `barter/src/engine/state/mod.rs` during market event processing with message `index out of bounds`.

**Cause**: A `MarketEvent` was received for an `InstrumentIndex` not registered in `IndexedInstruments`.

**Fix**: Ensure every `Subscription` passed to `StreamBuilder::subscribe()` corresponds to an instrument in `SystemConfig.instruments`. The subscription set and instrument list must be consistent.

### 3. `TradingState::Enabled` but no orders are generated

**Symptoms**: Market events arrive, `TradingState` is `Enabled`, but `AlgoOrders` audit output is always empty.

**Causes and fixes**:
- `DefaultStrategy::generate_algo_orders()` returns no orders by default — it is a no-op placeholder. Implement `AlgoStrategy` with your own logic.
- `RiskManager::check()` is filtering all orders. Add logging inside your `RiskManager` implementation to confirm.
- The `InstrumentFilter` on a prior `CancelOrders` or `ClosePositions` command has put the engine into a safe state. Check command history in the audit stream.

### 4. Backtest runs slower than expected

**Symptoms**: `backtests_concurrent` example takes minutes for simple strategies.

**Causes and fixes**:
- Building in debug mode. Always use `cargo run --release` for performance benchmarking.
- Audit mode enabled (`AuditMode::Enabled`) adds channel send overhead on every event. Use `AuditMode::Disabled` for pure throughput benchmarks.
- `EngineFeedMode::Async` adds Tokio channel overhead. Use `EngineFeedMode::Iterator` for backtests where the input `MarketStream` is a synchronous iterator over historical data.

### 5. Compile error: `the trait bound AlgoStrategy is not satisfied`

**Symptoms**: Rust compiler reports missing trait implementations when calling `SystemBuilder::build()`.

**Cause**: `SystemBuilder` is parameterised on concrete `Strategy` and `Risk` types that must implement all required strategy traits.

**Fix**: Use `DefaultStrategy` and `DefaultRiskManager` as a starting point, or implement all required strategy traits (`AlgoStrategy`, `ClosePositionsStrategy`, `OnDisconnectStrategy`, `OnTradingDisabled`) for your custom type.

---

## Security Considerations

- **No unsafe code**: All crates declare `#![forbid(unsafe_code)]`. Memory safety is guaranteed by the Rust borrow checker.
- **API keys**: Never hardcode exchange API keys. Load them from environment variables or a secrets manager at runtime. The `barter-execution` `ExecutionClient` implementations accept credentials at construction time.
- **TLS**: All WebSocket and REST connections in `barter-integration` use `rustls-tls-webpki-roots` — system CA stores are not required.
- **HMAC signing**: Request signing for authenticated endpoints uses `hmac` + `sha2` from the `RustCrypto` ecosystem. Source: workspace `Cargo.toml` dependencies.
- **Educational use**: Per the project LICENSE and README, barter-rs is provided for educational purposes. Do not deploy to production without thorough independent security and compliance review.

---

## See Also

- [README.md](README.md) — Project overview and quick start
- [architecture.md](architecture.md) — Crate responsibilities and component diagram
- [workflow.md](workflow.md) — Event processing pipeline
- [state-management.md](state-management.md) — `EngineState` internals and data models
