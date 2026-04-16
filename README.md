# bitfence

**The risk layer agents can't skip.**

bitfence is a pre-transaction token risk oracle for autonomous AI agents
operating on Base and Solana. Before an agent swaps, transfers, or interacts
with a token, bitfence returns a structured risk assessment in under a second:
a composite score, per-category breakdown, confidence value, action
recommendation, and any active circuit-breaker flags.

- **Live on Base and Solana**
- **HTTP API · MCP server · Coinbase AgentKit · Rig (Rust)**
- **$0.003 per query in USDC on Base via x402 — no API keys**

🔗 [bitfence.ai](https://bitfence.ai) · 📄 [Whitepaper](./WHITEPAPER.pdf) · 𝕏 [@bitfenceai](https://x.com/bitfenceai)

---

## Why this exists

Autonomous agents now hold keys, read mandates, and broadcast transactions
without a human in the loop. The agent frameworks shipping today expose swap
and transfer actions with no systematic pre-transaction safety check. The
risk-assessment tooling that exists was built for a human reading a wallet
pop-up — not a machine routing on structured fields.

bitfence fills the gap: a pre-intent token risk layer designed for machine
consumption. Neutral, transparent, agent-native, per-call priced. It is
complementary to existing wallet-grade tools, not competitive with them.

## What it scores

Risk is decomposed into four live categories and assembled into a 0–100
composite with maturity-aware scoring and structured circuit-breaker
overrides.

| Category              | What it covers                                                                 |
| --------------------- | ------------------------------------------------------------------------------ |
| **Contract authority**| Ownership, mint and freeze authority, upgradeable proxies, dangerous opcodes   |
| **Liquidity**         | LP lock and burn status, LP-to-FDV ratio, pool age                             |
| **Holder distribution** | Top-holder share, deployer share, Gini coefficient, holder count             |
| **Trading**           | Honeypot detection, sell and buy taxes                                         |

Two further categories — cross-chain deployer reputation and ML behavioural —
are reserved injection points for future work. The reported confidence value
reflects only the live categories.

The full methodology is in the [whitepaper](./WHITEPAPER.pdf).

## Integration surfaces

The same scoring engine is exposed through four interfaces. Each returns the
same `RiskAssessment` structure.

- **HTTP/JSON API** — for any caller that speaks HTTP
- **MCP server** — for tool-calling LLM agents (Claude Desktop, MCP-enabled IDEs)
- **Coinbase AgentKit** — `bitfenceActionProvider` for AgentKit-based agents
- **Rig (Rust)** — typed in-process tool binding for Rust-native agents

For integration details, contact `info@bitfence.ai`.

## Pricing and access

- **$0.003 per query** in USDC on Base via [x402](https://github.com/coinbase/x402)
- Implemented against the Coinbase CDP facilitator
- No accounts, no API keys, no signups
- Listed on the Coinbase x402 Bazaar

## Open-source posture

Documentation, whitepaper, and integration surfaces are open under Apache
2.0. The scoring engine source remains closed for security reasons:
publishing exact circuit-breaker thresholds, signal weights, and maturity
logic would let adversarial token deployers engineer tokens that sit
immediately below the trigger points, defeating the protection bitfence
provides to agents downstream. This trade-off is discussed in the
[whitepaper](./WHITEPAPER.pdf).

A publishable methodology document — one useful to other Ethereum security
teams without exposing trigger logic to deployers — is in active
development.

## Security

See [SECURITY.md](./SECURITY.md) for vulnerability disclosure.

## License

[Apache License 2.0](./LICENSE)
