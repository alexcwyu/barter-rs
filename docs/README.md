# Barter-rs

> **Last Updated**: 2026-04-07T00:00:00Z
> **Git Hash**: `df7ae78`

Barter-rs is an algorithmic trading ecosystem of Rust libraries for building high-performance live-trading, paper-trading, and back-testing systems. It provides a modular workspace of crates covering market data streaming, order execution, instrument modelling, and a fully featured trading engine — all implemented in safe, zero-unsafe Rust.

---

## Key Features

| Feature | Description |
|---|---|
| Multi-Exchange Engine | Execute strategies simultaneously across many exchanges via a single `Engine` |
| Normalised Market Data | Unified `MarketEvent` model across all supported venues via WebSocket streams |
| Live & Mock Execution | Swap real `ExecutionClient` for `MockExchange` without changing strategy code |
| O(1) State Lookups | Cache-friendly indexed data structures (`IndexedInstruments`, `AssetStates`) |
| Pluggable Strategy | Implement `AlgoStrategy`, `RiskManager`, and disconnect hooks independently |
| Audit Stream | Engine emits a typed `AuditStream` for non-hot-path monitoring or UI replication |
| Performance Metrics | Built-in `TradingSummary` with PnL, Sharpe, Sortino, Drawdown, and more |
| Concurrent Backtesting | Utilities for running thousands of backtests concurrently |
| Trading State Control | Toggle algorithmic order generation on/off from any external process |
| Reconnect Handling | Automatic WebSocket reconnection with configurable `OnDisconnectStrategy` |

---

## Quick Start

Add the core crate to your `Cargo.toml`:

```toml
[dependencies]
barter = "0.12"
barter-data = "0.11"
barter-execution = "0.7"
barter-instrument = "0.3"
tokio = { version = "1", features = ["macros", "rt-multi-thread"] }
```

Build a minimal paper-trading system:

```rust
use barter::{
    EngineEvent,
    engine::state::trading::TradingState,
    system::{SystemBuilder, config::SystemConfig, builder::{AuditMode, EngineFeedMode}},
    strategy::DefaultStrategy,
    risk::DefaultRiskManager,
};
use barter_instrument::index::IndexedInstruments;
use barter_data::streams::builder::subscription::SubKind;
use barter::system::builder::init_indexed_multi_exchange_market_stream;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Load instrument + execution config from JSON
    let config: SystemConfig = serde_json::from_str(include_str!("config/system_config.json"))?;
    let instruments = IndexedInstruments::new(config.instruments);

    // Open WebSocket market data streams for all instruments
    let market_stream = init_indexed_multi_exchange_market_stream(
        &instruments,
        &[SubKind::PublicTrades, SubKind::OrderBooksL1],
    ).await?;

    // Build and start system
    let mut system = SystemBuilder::new(barter::system::builder::SystemArgs::new(
        &instruments,
        config.executions,
        barter::engine::clock::LiveClock,
        DefaultStrategy::default(),
        DefaultRiskManager::default(),
        market_stream,
    ))
    .engine_feed_mode(EngineFeedMode::Iterator)
    .audit_mode(AuditMode::Enabled)
    .trading_state(TradingState::Disabled)
    .build::<EngineEvent, _, _>()?
    .init_with_runtime(tokio::runtime::Handle::current())
    .await?;

    system.trading_state(TradingState::Enabled);

    // Run until done, then shut down and print summary
    let (engine, _) = system.shutdown().await?;
    engine.trading_summary_generator(rust_decimal_macros::dec!(0.05))
          .generate(barter::statistic::time::Daily)
          .print_summary();
    Ok(())
}
```

---

## Architecture Summary

Barter-rs is structured as a Cargo workspace with six crates:

```
barter-rs/
├── barter/           # Core Engine, EngineState, Strategy, Risk, Statistics
├── barter-data/      # WebSocket market data streaming (public trades, order books)
├── barter-execution/ # Order execution client interface + MockExchange
├── barter-instrument/ # Exchange, Instrument, Asset data structures
├── barter-integration/ # Low-level REST/WebSocket protocol foundations
└── barter-macro/     # Procedural macros for the ecosystem
```

The `Engine` sits at the centre, consuming a feed of `EngineEvent`s and delegating to pluggable `Strategy` and `RiskManager` implementations. Market data arrives via `barter-data` WebSocket streams; order requests are routed to `barter-execution` clients. All state is held in a single `EngineState` with O(1) indexed lookups.

---

## Documentation Index

| Document | Contents |
|---|---|
| [architecture.md](architecture.md) | System architecture diagram, crate responsibilities, component class diagram |
| [workflow.md](workflow.md) | Processing pipeline sequence, data flow, event lifecycle |
| [state-management.md](state-management.md) | State machines, `EngineState` structure, data models |
| [development.md](development.md) | Setup, project layout, config reference, troubleshooting |

---

## Supported Exchanges (barter-data)

| Exchange | Spot | Perpetual | Futures |
|---|---|---|---|
| Binance | Yes | Yes | Yes |
| Coinbase | Yes | — | — |
| OKX | Yes | Yes | — |
| Bybit | Yes | Yes | — |
| Kraken | Yes | — | — |
| Gate.io | Yes | — | — |
| Bitfinex | Yes | — | — |
| BitMEX | — | Yes | — |

---

## Links

- [Crates.io](https://crates.io/crates/barter)
- [API Documentation](https://docs.rs/barter/latest/barter/)
- [GitHub Repository](https://github.com/barter-rs/barter-rs)
- [Discord Community](https://discord.gg/wE7RqhnQMV)
- [DeepWiki](https://deepwiki.com/barter-rs/barter-rs)

---

**Tags**: `rust`, `algorithmic-trading`, `backtesting`, `paper-trading`, `live-trading`, `market-data`, `websocket`, `crypto`, `hft`, `market-making`, `event-driven`, `tokio`, `order-management`, `strategy`
