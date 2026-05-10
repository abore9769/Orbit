<p align="center">
  <img src="public/Orbit.png" width="120" alt="Orbit logo" />
</p>

<h1 align="center">Orbit</h1>

<p align="center">
  <strong>An ECS framework for building on-chain games on Stellar</strong>
</p>

<p align="center">
  <a href="https://crates.io/crates/orbit-core"><img src="https://img.shields.io/crates/v/orbit-core.svg" alt="crates.io" /></a>
  <a href="https://stellar.org"><img src="https://img.shields.io/badge/Stellar-Soroban-blue?logo=stellar" alt="Stellar" /></a>
  <a href="https://www.rust-lang.org"><img src="https://img.shields.io/badge/Rust-1.70%2B-orange?logo=rust" alt="Rust" /></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-lightgrey" alt="License" /></a>
</p>

Orbit is a Rust framework for building on-chain games on Stellar. It brings an Entity Component System (ECS) architecture to Soroban smart contracts, paired with zero-knowledge tooling, smart account patterns, and a reusable standards layer — all designed to work within blockchain execution constraints.

The repo ships the core library, a catalog of standalone game examples, and research notes. The published crate is intentionally scoped: examples, CI scaffolding, and research docs support development and adoption without bloating the shipped surface.

---

## Status

Orbit `1.0.0` is published on [crates.io](https://crates.io/crates/orbit-core).

| Area | Status |
|---|---|
| ECS runtime and storage | Stable |
| Standards layer | Stable |
| Privacy primitives (`zk::stable`) | Stable |
| Accounts and smart-account patterns | Beta |
| Advanced ZK and confidential abstractions | Experimental |

---

## What's Inside

| Area | What it includes |
|---|---|
| ECS | Typed components, multiple world backends, scheduling, deferred commands, hooks, observers, change tracking |
| Zero-knowledge | Groth16 verification, curve helpers, commitments, Merkle structures, reusable circuits, ECS-integrated proof flows |
| Smart accounts | Session keys, social recovery, multi-device auth, fallback authorization |
| Standards | Ownable, AccessControl, Pausable, execution guards, recovery guards, delayed execution, batch primitives |
| Examples | Standalone game contracts demonstrating concrete patterns |

---

## Installation

```toml
[dependencies]
orbit-core = "1.0.0"
soroban-sdk = "25.1.0"
```

Generate local docs:

```bash
cargo doc --no-deps --all-features
```

---

## Quick Start

```rust
use orbit_core::app::{named_system, GameApp, ScheduleStage};
use orbit_core::SystemConfig;
use soroban_sdk::Env;

let env = Env::default();
let mut app = GameApp::new(&env);

app.add_systems((
    named_system("spawn_player", |world, env| {
        let player = world.spawn_entity();
        world.set_typed(env, player, &Position::new(0, 10));
    })
    .in_stage(ScheduleStage::Startup),
    named_system("tick", |_world, _env| {})
        .with_config(SystemConfig::new().in_stage(ScheduleStage::Update)),
));

app.run(&env).unwrap();
```

---

## Core Concepts

### ECS Runtime

`GameApp` is the recommended entry point for new projects. It provides stage-based execution (`Startup`, `PreUpdate`, `Update`, `PostUpdate`, `Cleanup`), dependency-aware scheduling, deferred commands, and plugin composition.

Two world backends are supported:
- `SimpleWorld` — general-purpose ECS storage, Soroban-first
- `ArchetypeWorld` — archetype-oriented storage for broader query patterns

### Zero-Knowledge

The `zk` module covers Groth16 proof verification, Merkle tree utilities (SHA256 and Poseidon), commit-reveal components, and reusable circuit builders. Stable primitives live under `zk::stable`; advanced tooling is opt-in under `zk::experimental`.

### Smart Accounts

The `accounts` module (aliased as `auth`) provides session keys for scoped gameplay actions, guardian-based social recovery, per-device policies, and graceful fallback from advanced auth to direct authorization.

### Standards

The `standards` module (aliased as `ops`) provides contract-level primitives: ownership, role-based access control, emergency pause, serialized critical sections, timelocked execution queues, and bounded batch operations.

---

## Examples

The `examples/` directory contains standalone game contracts. Each is self-contained, buildable from crates.io, and focused on a specific pattern.

| Category | Examples |
|---|---|
| Arcade | `pong`, `snake`, `flappy_bird`, `space_invaders`, `asteroids`, `arkanoid` |
| Board & strategy | `tic_tac_toe`, `chess`, `battleship`, `checkers`, `reversi` |
| Puzzle | `tetris`, `minesweeper`, `sudoku`, `memory_match` |
| Card & RPG | `trading_card_game`, `shadow_draft_card_game`, `guild_arena` |
| Experimental | `rock_paper_scissors`, `tap_battle`, `proof_of_hunt` |

See [examples/README.md](examples/README.md) for the full catalog.

---

## Repository Layout

```
src/          Core framework
tests/        Integration, edge-case, and stress tests
benches/      Benchmark targets
examples/     Standalone game contracts
research/     Design and exploration notes
```

---

## Development

```bash
cargo fmt --check
cargo clippy --all-targets --all-features -- -D warnings
cargo test
cargo build
```

---

## Compatibility

| Item | Value |
|---|---|
| Rust | 1.70+ |
| Edition | 2021 |
| License | MIT |
| Soroban SDK | 25.1.0 |
| Targets | Soroban-compatible WASM |
