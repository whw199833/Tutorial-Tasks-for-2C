# Fuzzy intent and account onboarding

## Overview

| API | Function | Use Case |
|-----|----------|----------|
| Balances | `assets.getUserAssets` | Route cold-start vs funded users |
| On-ramp | `p2p`, `fiat` | Empty-account deposit / buy guidance |
| Next steps | Guided spot / earn / bots | High-level first actions after snapshot |

## 1. Description

For vague first messages or broad intent (e.g. “what can you do”, “how do I start as a newbie”, “teach me to make money”, “what features do you have”, “hello”).

The agent should:

1. Give a **concise** overview of the main capability areas (trading, earn, trading bots, etc.).
2. **Proactively** pull the user’s **current asset state**.
3. Based on whether funds are sufficient, suggest **next steps**: empty account → guide deposit, P2P/fiat buy, or internal transfer (“cold start”); funded → guide first trade or earn.

## 2. Core skills

- **`assets`**: Required. Balances across spot, funding, earn wallets and total valuation.
- **`p2p` / `fiat`**: Optional. When the account is empty, explain fiat or P2P on-ramps.
- **Mentioned modules**: spot/futures, Simple Earn, Trading Bots.

---

## 3. Plan

### §A Structured pipeline (DAG)

**Step 1: Icebreaker and capability overview**

- Short bullet overview:
  1. **Smart trading and analysis** (spot/futures orders, market snapshot, klines)
  2. **Wealth products** (principal-protected earn, staking-style products)
  3. **Automation** (grid, DCA-style bots)
- Transition: *“To tailor advice, I’ll quickly check your current account balance…”*

**Step 2: Account state check**

- **Must** call `assets` (`getUserAssets`): distribution across wallets, focus `SPOT` and `FUNDING` total USDT-equivalent and non-zero assets.

**Step 3: Conditional guidance**

- **A: Empty or very low balance**  
  - State clearly there is no meaningful tradable balance.  
  - **On-ramp**: crypto deposit address; fiat/P2P options (optional `p2p` for quote context).  
  - Example: *“Your account looks empty. To earn or trade we need funds first—deposit from another wallet, or buy with card/P2P?”*

- **B: Funds idle (e.g. all in Funding, or USDT idle)**  
  - Suggest internal **transfer** to spot/futures for trading, or introduce Simple Earn for idle stablecoins.

- **C: Spot funded with holdings**  
  - Higher-level ideas: grids/DCAs for stables; for BTC/ETH etc., brief market context or mention TP/SL—**still compliant, not guaranteed returns**.

### §B Endpoint quick reference

- **Balances**: `assets` → `getUserAssets`.
- **Buy channels (optional)**: `p2p` → e.g. ads/quote endpoints per SKILL for motivation-only context.

---

## 4. Usage and boundaries

- **Keep Step 1 short**: bullets + light emoji OK—no walls of text.
- **Privacy**: summarize (“~$500 mostly in spot”); skip dust; focus on **next-step** relevance.
- **CTA**: end with a **binary or concrete question** (“Deposit first, or scan today’s market?”)—avoid dead-end statements.
- **Not financial advice**; KYC/fiat edge cases → human support.

For cross-task pre-planning and the full intent index, see [task-upgrade-advice.md](./task-upgrade-advice.md) and the parent [SKILL.md](../SKILL.md).
