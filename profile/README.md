# Tollcraft

**Developer tooling that helps Stellar smart contract developers understand and control resource cost — before deployment, not after.**

---

## The Problem

Soroban's fee model charges based on actual metered resource use: CPU instructions, memory, storage reads and writes, bandwidth, and rent. That makes fees fair and predictable at the network level — but it's easy for a developer to write code that's more expensive than it needs to be, and today there's no good way to find that out until the contract is already live and real users are already paying for it.

Tollcraft exists to move that discovery earlier — ideally to the moment the code is written, not the moment it's deployed.

## Approach: Two Kinds of Cost Bug, Three Tools

Cost problems in Soroban contracts split into two categories that require fundamentally different techniques to catch:

- **Cost that's expensive no matter what the input is** — redundant clones, inefficient storage patterns, unnecessary host-function calls. These can, in principle, be caught by analyzing the code itself, without running it.
- **Cost that depends on runtime input** — loop bounds from arguments, the size of a `Vec` or `Bytes`, how much data is already in storage. These *cannot* be known by reading the code alone; they require actually running it and measuring what happens.

Static analysis is good at the first category and structurally blind to the second. That split defines the pipeline; the third tool exists because measuring a cost is not the same as explaining it:

| Tier | Repo | Catches | Tool today |
|---|---|---|---|
| 1 — Prevent | [`soroban-cost-linter`](https://github.com/Tollcraft/soroban-cost-linter) | Input-independent expensive patterns, at compile time | `v0.1.1`: a [Dylint](https://github.com/trailofbits/dylint) lint collection over the Rust AST, driven by `cargo cost-lint`, wired into editors and CI |
| 2 — Detect | [`soroban-budget-assert`](https://github.com/Tollcraft/soroban-budget-assert) | Input-dependent cost, at test time, against real network metering | `v0.1.0`: `budget-macros` fails a test the moment cost crosses a pinned limit; `cargo-budget-report` simulates each workspace contract on testnet and publishes the numbers to a cost-over-time dashboard |
| 3 — Diagnose | [`soroban-cost-profiler`](https://github.com/Tollcraft/soroban-cost-profiler) | *Where* the cost went, once tier 2 has failed | A standalone Rust CLI that runs one export of a compiled contract under `soroban-env-host`, folds the event stream into a call tree, and writes collapsed-stack text for speedscope or `flamegraph.pl` |

Neither tool claims to solve the whole problem alone, and no tool here asks you to add instrumentation to your contract source: the linter reads the AST, the assertions are attributes on your own tests, and the profiler consumes a standard `cargo build --profile profiling` artifact.

## What is actually known

The gap tier 2 exists to measure is now measured, and it goes both ways. On the repository's AMM-pool fixture, `do_expensive_work(10_000)` read **901,816 local WASM CPU instructions against 756,678 on testnet** under the size-optimized release profile, and **767,049 local against 832,006 testnet** under Cargo's default release profile — same contract, same input, opposite direction. That is the reason cost thresholds anywhere in this initiative are treated as directional until they are pinned against a specific build profile and a specific protocol, and every published figure is recorded in that repo's `MEASUREMENTS.md` rather than quoted from memory.

What tier 3 has found is the constraint it works under: `wasmi` 2.0's only execution hook is a call hook whose closure receives no program counter, and the engine keeps no instruction hook, so every event the profiler records lands at `pc = 0` and the frames it writes are `wasm[0]`. The DWARF and `name`-section source mapping behind those frames is built, tested against real `wasm32-unknown-unknown` builds, and reachable — it is not yet fed a live address. The profiler says so in its own output instead of printing a table of plausible zeros.

## Documentation

The three tools are documented in one place: **[tollcraft.github.io/docs](https://tollcraft.github.io/docs/)** — per-tool guides, the lint catalog, the measured cost tables, and the profiler's CLI reference.

- Profiler landing page and worked example: [tollcraft.github.io/soroban-cost-profiler](https://tollcraft.github.io/soroban-cost-profiler/)
- Cost-over-time dashboard (testnet-measured, one point per push to `main`): [tollcraft.github.io/soroban-budget-assert/dashboard.html](https://tollcraft.github.io/soroban-budget-assert/dashboard.html)

## Contributing

Each repo runs its own issue tracker; the profiler's `ROADMAP.md` is its source of truth and its issues are titled by phase. Documentation and research contributions are welcome in cost-model research, lint specification, WASM tracing, and source mapping.

- Open issues to pick up: [soroban-cost-linter](https://github.com/Tollcraft/soroban-cost-linter/issues) · [soroban-budget-assert](https://github.com/Tollcraft/soroban-budget-assert/issues) · [soroban-cost-profiler](https://github.com/Tollcraft/soroban-cost-profiler/issues)
- See a repo's `CONTRIBUTING.md` before opening a PR — the profiler's asks for a `ROADMAP.md` checkbox in the same branch and a detailed PR body, not a one-line summary.

## Community

[Discord](https://discord.gg/5aprtMSyR) · [Telegram](https://t.me/+Gflo5jZStw1jMjE0)
