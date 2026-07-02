# Tollcraft

**Developer tooling that helps Stellar smart contract developers understand and control resource cost — before deployment, not after.**

---

## The Problem

Soroban's fee model charges based on actual metered resource use: CPU instructions, memory, storage reads and writes, bandwidth, and rent. That makes fees fair and predictable at the network level — but it's easy for a developer to write code that's more expensive than it needs to be, and today there's no good way to find that out until the contract is already live and real users are already paying for it.

Tollcraft exists to move that discovery earlier — ideally to the moment the code is written, not the moment it's deployed.

## Approach: Two Tiers, Not One Tool

Cost problems in Soroban contracts split into two categories that require fundamentally different techniques to catch:

- **Cost that's expensive no matter what the input is** — redundant clones, inefficient storage patterns, unnecessary host-function calls. These can, in principle, be caught by analyzing the code itself, without running it.
- **Cost that depends on runtime input** — loop bounds from arguments, the size of a `Vec` or `Bytes`, how much data is already in storage. These *cannot* be known by reading the code alone; they require actually running it and measuring what happens.

Static analysis is good at the first category and structurally blind to the second. That split defines Tollcraft's two active projects:

| Repo | Tier | What it catches | Status |
|---|---|---|---|
| [`soroban-cost-linter`](https://github.com/Tollcraft/soroban-cost-linter) | Static (compile-time) | Input-independent expensive patterns | Research & architecture phase |
| [`soroban-budget-assert`](https://github.com/Tollcraft/soroban-budget-assert) | Runtime (test-time) | Input-dependent cost via actual execution | Active — early-stage |

Neither tool claims to solve the whole problem alone. That's deliberate — a tool that promised to catch all cost issues statically would be overclaiming what static analysis can actually do.

## Current Priority

`soroban-budget-assert`'s first real milestone is measuring how much Soroban's local test-mode budget estimate diverges from real, network-verified cost (via `simulateTransaction` against testnet). That number doesn't exist publicly yet, as far as we've found. Until it's measured, cost thresholds in any tool in this initiative should be treated as directional, not precise.

## Funding & Context

This work is being developed as part of Drips Wave 7. The current sprint is explicitly scoped as research and architecture — mapping proposed tooling to Soroban's real metering and fee model — rather than shipping a finished product. Later phases build on whatever that research finds, not the other way around.

## Repositories

- [`soroban-cost-linter`](https://github.com/Tollcraft/soroban-cost-linter) — static lint plugin, currently defining its first lint specifications and false-positive mitigation strategy.
- [`soroban-budget-assert`](https://github.com/Tollcraft/soroban-budget-assert) — runtime budget-assertion test harness and network-verified cost reporting.

## Contributing

Each repo has its own contributing guide. If you're looking for where to start, check open issues on either repo above — this initiative is actively looking for contributors in cost-model research, lint specification, and system architecture.

## License

*(Add initiative-wide license or licensing policy here, if one applies across repos.)*
