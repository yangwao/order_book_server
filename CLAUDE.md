# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Test Commands

```bash
cargo build --workspace           # Build all crates
cargo test --workspace            # Run all tests
cargo test -p server              # Test library crate only
cargo test <test_name>            # Run single test by name
cargo fmt                         # Format (max width 120)
cargo clippy --workspace          # Lint check (must pass before commits)

# Run the server
RUST_LOG=info cargo run --release --bin websocket_server -- \
  --address 0.0.0.0 --port 8000 --websocket-compression-level 1
```

## Architecture

Local WebSocket server for Hyperliquid order book data. Ingests events from a non-validating Hyperliquid node and serves `l2book`, `trades`, and `l4book` subscriptions.

**Data flow:**
```
Node event files (~/.hl-node/events/{order_statuses,fills,order_diffs}/)
    ↓
listeners/order_book/ (file watchers + parsers)
    ↓
order_book/ (in-memory state: OrderBook<O>, OrderBooks<O>)
    ↓
servers/websocket_server.rs (axum + yawc)
    ↓
Client WebSocket subscriptions
```

**Workspace structure:**
- `server/` - Core library: `listeners/`, `order_book/`, `servers/`, `types/`
- `binaries/src/bin/` - Executables: `websocket_server.rs`, `example_client.rs`

**Key abstractions:**
- `InnerOrder` trait (`order_book/types.rs`) - Generic contract for order types
- `OrderBook<O>` (`order_book/mod.rs`) - Single coin book (BTreeMap of LinkedLists)
- `OrderBooks<O>` (`order_book/multi_book.rs`) - Multi-coin container
- `OrderBookListener` (`listeners/order_book/mod.rs`) - Event ingestion + state management
- `SubscriptionManager` (`types/subscription.rs`) - Client subscription routing

**Concurrency:** `Arc<Mutex<OrderBookListener>>` for shared state, `tokio::broadcast` for updates.

## Code Conventions

- Rust 2024 edition with strict clippy (all warn-level categories enabled)
- Avoid `unwrap`/`expect` except in `#[cfg(test)]` blocks
- `unsafe` code is warned - avoid unless necessary
- Error type: `Box<dyn std::error::Error + Send + Sync>` (see `prelude.rs`)
- Tests colocated in module files, not separate `tests/` directory
