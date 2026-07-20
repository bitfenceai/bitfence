# API Reference

Base URL: `https://api.bitfence.ai`

All risk endpoints require authentication via [x402 micropayment](#x402-payment-flow) or API key.

---

## Endpoints

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| `GET` | `/` | Service info (name, version, status) | None |
| `GET` | `/v1/risk/{chain}/{token_address}` | Full risk assessment | Required |
| `POST` | `/v1/risk/contextual` | Position-aware risk assessment | Required |
| `GET` | `/health` | Service health check | None |

---

## GET /

Returns basic service information.

**Request:**

```bash
curl https://api.bitfence.ai/
```

**Response (200):**

```json
{
  "name": "bitfence",
  "version": "0.7.6",
  "status": "operational"
}
```

---

## GET /health

Returns service status and supported chains.

**Request:**

```bash
curl https://api.bitfence.ai/health
```

**Response (200):**

```json
{
  "status": "ok",
  "version": "0.7.6",
  "chains": ["solana", "base", "ethereum", "arbitrum", "bsc"]
}
```

`chains` lists the chains currently enabled on this deployment. A supported-but-disabled chain (currently `hyperevm`, gated behind its release check) is absent from the list and returns `400 unsupported_chain` when queried.

---

## GET /v1/risk/{chain}/{token_address}

Returns a full risk assessment for a token on the specified chain.

### Path parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `chain` | string | Blockchain network: `solana`, `base`, `ethereum`, `arbitrum`, or `bsc` (`hyperevm` once enabled) |
| `token_address` | string | Token mint address (Solana) or `0x` contract address (EVM chains) |

### Request

```bash
# Solana token
curl https://api.bitfence.ai/v1/risk/solana/EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v \
  -H "X-API-Key: your_api_key"

# Base token
curl https://api.bitfence.ai/v1/risk/base/0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 \
  -H "X-API-Key: your_api_key"
```

### Response (200)

```json
{
  "chain": "solana",
  "token_address": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
  "risk_score": 23,
  "risk_level": "LOW",
  "confidence": 0.85,
  "recommendation": "PROCEED",
  "reasoning": "Low-risk token with revoked authorities and adequate liquidity.",
  "flags_triggered": [],
  "cached": false,
  "cache_source": null,
  "analysis_ms": 187,
  "response_ms": 192,
  "version": "0.7.6"
}
```

### Response fields

| Field | Type | Description |
|-------|------|-------------|
| `chain` | string | The chain that was queried |
| `token_address` | string | The token address that was queried |
| `risk_score` | integer | Composite risk score, 0 (safe) to 100 (maximum risk) |
| `risk_level` | string | `LOW`, `MEDIUM`, or `HIGH` |
| `confidence` | float | Confidence in the assessment, 0.0–1.0 |
| `recommendation` | string | `PROCEED`, `REQUIRE_HUMAN_APPROVAL`, or `BLOCK` |
| `reasoning` | string | Human-readable summary of the analysis |
| `flags_triggered` | array of string | Names of any circuit breakers that fired (e.g. `"honeypot"`, `"lp_fully_unlocked_deployer_held"`). Empty when nothing fired. A non-empty array means the score was overridden by a safety floor — treat as `BLOCK`. Thresholds are not exposed. |
| `trusted_token` | string (optional) | Registry display name (e.g. `"Circle (USDC)"`) when the token matches the curated trusted-token registry; the field is omitted otherwise. |
| `degraded` | object (optional) | Present only when a breaker-critical data source was unavailable for this assessment. Contains `missing_sources` (array of source names), `checks_not_run` (array of breaker names that could not reach a conclusion), and `recommendation_capped` (boolean — when `true`, `PROCEED` is withheld and the recommendation is `REQUIRE_HUMAN_APPROVAL` unless a fired breaker independently demands `BLOCK`). Omitted otherwise. |
| `cached` | boolean | Whether this response was served from cache |
| `cache_source` | string or null | `"l1"` (in-memory), `"l2"` (Redis), or `null` (fresh computation) |
| `analysis_ms` | integer | Time spent on analysis in milliseconds |
| `response_ms` | integer | Total request-to-response time in milliseconds |
| `version` | string | API version |

---

## POST /v1/risk/contextual

Returns a risk assessment with additional position-aware context, including estimated slippage, portfolio concentration, and suggested position limits.

### Request body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `chain` | string | Yes | Blockchain network: `solana`, `base`, `ethereum`, `arbitrum`, or `bsc` |
| `token` | string | Yes | Token address |
| `position_size_usd` | float | Yes | Intended position size in USD |
| `agent_portfolio_usd` | float | Yes | Total portfolio value in USD |

### Request

```bash
curl -X POST https://api.bitfence.ai/v1/risk/contextual \
  -H "Content-Type: application/json" \
  -H "X-API-Key: your_api_key" \
  -d '{
    "chain": "solana",
    "token": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
    "position_size_usd": 5000,
    "agent_portfolio_usd": 25000
  }'
```

### Response (200)

The response includes all fields from the standard risk assessment, plus a `context` object:

```json
{
  "chain": "solana",
  "token_address": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
  "risk_score": 23,
  "risk_level": "LOW",
  "confidence": 0.85,
  "recommendation": "PROCEED",
  "reasoning": "Low-risk token with revoked authorities and adequate liquidity.",
  "flags_triggered": [],
  "cached": false,
  "cache_source": null,
  "analysis_ms": 210,
  "response_ms": 215,
  "version": "0.7.6",
  "context": {
    "pool_liquidity_usd": 2400000.0,
    "estimated_slippage_pct": 0.12,
    "effective_cost_usd": 5006.0,
    "max_safe_position_usd": 48000.0,
    "portfolio_concentration_pct": 20.0,
    "mev_exposure": "low",
    "suggested_max_usd": 12500.0
  }
}
```

### Context fields

| Field | Type | Description |
|-------|------|-------------|
| `pool_liquidity_usd` | float or null | Total liquidity available in the primary pool |
| `estimated_slippage_pct` | float or null | Estimated price impact for the given position size |
| `effective_cost_usd` | float or null | Position cost including slippage |
| `max_safe_position_usd` | float or null | Maximum position size that stays within acceptable slippage |
| `portfolio_concentration_pct` | float | Position as a percentage of total portfolio |
| `mev_exposure` | string or null | Estimated MEV risk: `"low"`, `"medium"`, or `"high"` |
| `suggested_max_usd` | float or null | Suggested maximum position size given risk and liquidity |

---

## Error responses

All errors return a JSON body with `error` and `message` fields.

### Unsupported chain (400)

```json
{
  "error": "unsupported_chain",
  "message": "unsupported chain 'tron'. Supported: [\"solana\", \"base\", \"ethereum\", \"arbitrum\", \"bsc\", \"hyperevm\"]"
}
```

A chain that is supported by the binary but disabled on the deployment (kill switch, e.g. `hyperevm` today) also returns `400 unsupported_chain`.

### Invalid address (400)

```json
{
  "error": "invalid_address",
  "message": "Token address appears invalid"
}
```

### Chain unavailable (503)

Returned when the chain is supported but not configured on this deployment, or an upstream data source is unreachable.

```json
{
  "error": "chain_unavailable",
  "message": "Base chain support is not configured"
}
```

### Internal error (500)

```json
{
  "error": "analysis_failed",
  "message": "Risk assessment failed"
}
```

Error-code summary:

| HTTP status | `error` | Cause |
|-------------|---------|-------|
| 400 | `unsupported_chain` | Chain is not one of the supported identifiers, or is disabled on this deployment |
| 400 | `invalid_address` | Address fails chain-specific format validation |
| 400 | `invalid_json` / `invalid_input` | Malformed or out-of-range contextual request body |
| 503 | `chain_unavailable` | Chain supported but adapter unconfigured or upstream unreachable |
| 500 | `analysis_failed` | Unexpected failure in the analysis pipeline |

---

## x402 payment flow

bitfence uses the [x402 protocol](https://www.x402.org/) for permissionless micropayments. No API key or account registration is required.

### Pricing

| Endpoint | Price per query |
|----------|----------------|
| `GET /v1/risk/{chain}/{token_address}` | $0.003 USDC |
| `POST /v1/risk/contextual` | $0.005 USDC |

### How it works

Payment is gated by a self-contained middleware that verifies and settles against the [Coinbase CDP](https://docs.cdp.coinbase.com/x402/) facilitator. The flow:

1. Send a request to any risk endpoint without authentication.
2. The server responds with `402 Payment Required`. The body carries the x402 challenge — an `x402Version`, an `error` string, and an `accepts` array describing the accepted scheme, network, token, amount, and canonical resource — alongside Bazaar discovery metadata (name, description, JSON-Schema, example I/O) for the Coinbase CDP marketplace. The same challenge is also base64-encoded into a `PAYMENT-REQUIRED` response header.

```json
{
  "x402Version": 1,
  "error": "X-PAYMENT header is required",
  "accepts": [
    {
      "scheme": "exact",
      "network": "base",
      "maxAmountRequired": "3000",
      "resource": "https://api.bitfence.ai/v1/risk/{chain}/{token_address}",
      "description": "bitfence token risk assessment",
      "payTo": "0x..."
    }
  ]
}
```

3. Construct a signed payment payload according to the x402 protocol specification and include it in the payment header of a retry request. Both the v1 `X-PAYMENT` header and the v2 `payment-signature` header are accepted:

```bash
curl https://api.bitfence.ai/v1/risk/solana/EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v \
  -H "X-PAYMENT: <base64-encoded-signed-payment-payload>"
```

4. The server verifies the payment with the CDP facilitator (EdDSA JWT bearer auth), runs the handler, and settles on-chain. If settlement fails the response is discarded and a `502` is returned — a paid response is never delivered to a non-paying client.

x402-compatible client libraries (available for TypeScript, Python, and other languages) handle steps 2–4 automatically.

### API key bypass

For high-volume consumers, API keys bypass the x402 payment flow entirely. Include the key in the `X-API-Key` header:

```bash
curl https://api.bitfence.ai/v1/risk/solana/EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v \
  -H "X-API-Key: your_api_key"
```

Contact us to request an API key.

---

## Caching

Responses are cached by `{chain}:{token_address}` (EVM addresses are lowercased before lookup) in a two-tier cache: an in-process moka L1 backed by an optional Redis L2.

TTLs are **risk-score-keyed**, not flat: high-risk scores receive short TTLs so a manipulated upstream reading cannot persist, while low-risk scores on stable tokens are cached longer. Exact durations are not published.

Cache correctness is version-gated. Every cached entry is stamped with the `policy_version` that produced it; an L2 entry stamped with a different version is treated as a miss, so no cached verdict outlives the policy that produced it. Assessments that were `degraded` (a breaker-critical source was dark) are never written to L2.

Cached responses include `"cached": true` and indicate the layer in `cache_source` (`"l1"`, `"l2"`, or `null` on fresh computation). The `analysis_ms` field reflects the original computation time; `response_ms` reflects the actual request duration.

---

## Rate limits

Requests are rate-limited per client IP. The default limit is 10 requests/second with a short burst allowance. Callers exceeding the limit receive `429 Too Many Requests`.
