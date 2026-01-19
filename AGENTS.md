# Repository Guidelines

## Project Structure & Module Organization

- Workspace root contains shared config (`Cargo.toml`, `rustfmt.toml`) and documentation (`README.md`).
- `server/` is the core library crate. Main modules live under `server/src/` with submodules like `listeners/`, `order_book/`, `servers/`, and `types/`.
- `binaries/` contains runnable entry points in `binaries/src/bin/`:
  - `websocket_server.rs` (primary server binary)
  - `example_client.rs` (client example)
- Tests are colocated with code in module files via `#[test]` blocks (no top-level `tests/` directory).

## Build, Test, and Development Commands

```bash
cargo build --workspace
```
Builds all crates in the workspace.

```bash
cargo test --workspace
```
Runs all unit tests across `server` and `binaries`.

```bash
RUST_LOG=info cargo run --release --bin websocket_server -- --address 0.0.0.0 --port 8000
```
Runs the local WebSocket server with logging enabled.

## Coding Style & Naming Conventions

- Rust 2024 edition; format with `cargo fmt` (see `rustfmt.toml`, max width 120).
- Clippy lints are configured at the workspace level; keep code clippy-clean and avoid `unwrap`/`expect` where possible.
- Follow Rust naming conventions: `snake_case` for modules/functions, `CamelCase` for types, `SCREAMING_SNAKE_CASE` for constants.

## Testing Guidelines

- Prefer module-level unit tests near the code under test.
- Keep tests deterministic and focused on order book behavior and subscription parsing.
- Run targeted tests with `cargo test -p server` when working only on the library crate.

## Commit & Pull Request Guidelines

- Commit history uses short, imperative messages and sometimes a `type:` prefix (e.g., `fix: panic in orderbook listener`).
- Keep commits small and focused; include the primary behavior change in the subject line.
- PRs should include: a concise summary, test commands run, and notes about any API or protocol changes.

## Runtime & Configuration Notes

- The server expects a locally running non-validating Hyperliquid node (see `README.md` for setup details).
- If the node stops emitting events, the server may exit after a short timeout; plan local workflows accordingly.
