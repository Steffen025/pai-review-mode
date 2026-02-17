# pai-review-mode

**Platform-Agnostic Review Mode for PAI agents — True Dual-Context isolation for processing untrusted code contributions.**

## What is this?

Review Mode is a security framework for AI coding agents (OpenCode, Claude Code, etc.) that need to safely analyze untrusted code — pull requests, external repositories, user-submitted code.

The core problem: When an AI agent reads untrusted code, that code can contain **prompt injection attacks** that hijack the agent. Review Mode prevents this through **True Dual-Context isolation** — the main agent never sees raw untrusted content.

```
┌─────────────────────────────────────────────────────────┐
│                    MAIN AGENT (Trusted)                  │
│  Has access to all tools                                 │
│  NEVER sees untrusted content directly                   │
│  Creates HMAC-signed TypedReferences                     │
└────────────────────┬────────────────────────────────────┘
                     │ Spawns via Task tool
                     ▼
┌─────────────────────────────────────────────────────────┐
│               QUARANTINE AGENT (Untrusted)               │
│  Only Read, Grep, Glob tools allowed (hook-enforced)     │
│  Can ONLY access files via HMAC-verified references      │
│  Returns structured JSON findings                        │
└─────────────────────────────────────────────────────────┘
```

## Key Features

- **True Dual-Context Isolation** — Separate agent contexts for trusted and untrusted content
- **HMAC-SHA256 TypedReferences** — Cryptographically signed file access URIs
- **Hook-Enforced Tool Allowlist** — Only Read, Grep, Glob permitted in quarantine (23 tools blocked)
- **Platform-Agnostic** — Works on OpenCode (`throw Error()`) and Claude Code (`process.exit(2)`)
- **Rate Limiting** — 100 calls/min per agent, max 5 concurrent quarantine agents
- **Audit Logging** — Buffered JSONL security event logging with sensitive data redaction

## Quick Start

```bash
# Clone
git clone https://github.com/Steffen025/pai-review-mode.git
cd pai-review-mode

# Install
bun install

# Test
bun test

# Adversarial attack tests
bun test:adversarial

# Performance benchmarks
bun test:bench

# Coverage
bun test:coverage
```

## Performance

| Metric | Result | Target | Margin |
|--------|--------|--------|--------|
| HMAC generation | 0.02ms avg | <50ms | 2,500x |
| Hook enforcement | 0.01ms p99 | <10ms | 1,000x |
| Quarantine spawn (50 files) | 13ms p95 | <2s | 150x |

## Test Coverage

- **364 total tests** (123 unit + 165 integration + 85 adversarial + 11 benchmark)
- **89.26% line coverage**
- 8 adversarial attack scenarios (AS-001 through AS-008)

## Architecture

See [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) for the full system design, component diagram, data flow, and ADRs.

See [`docs/SECURITY.md`](docs/SECURITY.md) for the threat model and attack scenario documentation.

See [`docs/README.md`](docs/README.md) for detailed usage documentation.

## Project Structure

```
pai-review-mode/
├── src/
│   ├── config/          # Configuration schemas
│   ├── hooks/           # Hook enforcement + platform adapters
│   │   └── adapters/    # OpenCode + Claude Code adapters
│   ├── lib/             # Core: HMAC, TypedReferences, sessions
│   └── quarantine/      # Spawn templates, response parser, timeout
├── tests/
│   ├── adversarial/     # 85 attack scenario tests
│   ├── benchmarks/      # 11 performance tests
│   ├── fixtures/        # Test data
│   ├── integration/     # 165 integration tests
│   ├── platform-compat/ # Cross-platform tests
│   └── unit/            # 123 unit tests
└── docs/
    ├── ARCHITECTURE.md  # System design + ADRs
    ├── README.md        # Detailed usage docs
    └── SECURITY.md      # Threat model
```

## Connection to pai-collab

This project is registered on the [pai-collab](https://github.com/mellanon/pai-collab) blackboard as a standalone tool. pai-collab is the coordination hub for the PAI (Personal AI) ecosystem — Review Mode is one of the tools built within that ecosystem.

- **Blackboard entry:** `projects/pai-review-mode/` on pai-collab
- **Security reviews:** PRs [#90](https://github.com/mellanon/pai-collab/pull/90), [#99](https://github.com/mellanon/pai-collab/pull/99), [#100](https://github.com/mellanon/pai-collab/pull/100), [#101](https://github.com/mellanon/pai-collab/pull/101) (all merged)

## License

MIT — See [LICENSE](LICENSE) for details.

## Security Disclosure

If you discover a security vulnerability, please report it via [GitHub Security Advisories](https://github.com/Steffen025/pai-review-mode/security/advisories).
