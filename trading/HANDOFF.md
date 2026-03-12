# HANDOFF.md — Trading System Context for New Claude Session

**Last updated:** 2026-03-11
**Author:** Claude (session on Yujun's MacBook)

> **How to use this file:** Start a new Claude Code session in `/Users/yujunzou/python/python_repo/trading/` and paste:
> "Read /Users/yujunzou/python/python_repo/trading/HANDOFF.md — this is the full context for the trading research system. Pick up where the last session left off."

---

## 1. What This System Is

An **autonomous AI agent research loop** for crypto trading strategies. Three agents (Sage, Forge, Nova) coordinate via Discord + OpenClaw to:
1. **Propose** strategies (Sage researches)
2. **Backtest** strategies (Forge codes + runs)
3. **Evaluate** results (Sage analyzes)
4. **Promote/reject** and open next round

Primary metric: **Calmar ratio** (CAGR / Max DD). Tier 1 = Calmar > 4.

---

## 2. Key Files — Read These First

### Agent Docs (nova-brain repo)
| File | What It Is |
|---|---|
| `nova-brain/agents/sage/SOUL.md` | Sage identity + core principles |
| `nova-brain/agents/sage/AGENTS.md` | Sage skills, Discord setup, tools, session context rules |
| `nova-brain/agents/sage/MEMORY.md` | Sage long-term knowledge + research status |
| `nova-brain/agents/sage/RESEARCH_LOOP.md` | Round-based protocol (propose → backtest → evaluate) |
| `nova-brain/agents/sage/NIGHTLY_SPRINT.md` | Autonomous overnight sprint protocol |
| `nova-brain/agents/forge/SOUL.md` | Forge identity |
| `nova-brain/agents/forge/AGENTS.md` | Forge skills, backtest conventions, Discord posting |
| `nova-brain/agents/forge/MEMORY.md` | Forge technical memory + all backtest results |

### Scripts (apexnova repo)
| File | What It Does |
|---|---|
| `apexnova/bin/deep_research.py` | Gemini 2.5 Pro + Google Search for strategy research (~40s) |
| `apexnova/bin/download_crypto_data.py` | Pre-cache BTC/ETH daily data for backtests |
| `apexnova/bin/btc_ma_binanceus_trader.py` | Live paper trader (MA120 on BinanceUS, runs as LaunchAgent) |
| `apexnova/bin/ma120_variants_backtest.py` | Forge's crypto backtest template |
| `apexnova/bin/bid_sma_backtest.py` | BidirectionalSMA (Round 4) |
| `apexnova/bin/bid_sma_stop_backtest.py` | BidirectionalSMA + stops (Round 4 Iter 2) |
| `apexnova/bin/rsi_regime_backtest.py` | RSI Regime Filter (Round 1) |
| `apexnova/bin/atr_regime_backtest.py` | ATR Volatility (Round 2) |
| `apexnova/bin/eth_validation_backtest.py` | ETH validation (Round 3) |

### Config
| File | What It Is |
|---|---|
| `~/.openclaw/openclaw.json` | OpenClaw config: agent definitions, Discord bot tokens, model settings |
| `~/.openclaw/gemini_api_key` | Gemini API key for deep_research.py |
| `freqtrade/openclaw_setup/GOLDEN_SHEET.md` | Strategy performance leaderboard (source of truth) |

---

## 3. System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenClaw Platform                         │
│                                                             │
│  sage-fallback-4h cron (isolated session, full tools)       │
│  ┌──────────┐    Discord     ┌──────────┐                  │
│  │   Sage   │ ──ping──────→  │  Forge   │                  │
│  │ (Sonnet) │ ←─results────  │ (Sonnet) │                  │
│  └────┬─────┘                └──────────┘                  │
│       │ pings                                               │
│       ▼                                                     │
│  ┌──────────┐                                              │
│  │   Nova   │  (orchestrator, #all-hands)                  │
│  └──────────┘                                              │
└─────────────────────────────────────────────────────────────┘

Discord Channels:
  #yujun-team (1480051404683870480) — ALL research, results, coordination
  #ma120-signals (1480098496613715982) — live paper trader signals only
  #all-hands (1480262938672500848) — cross-team, Nova pings
  #damien-team (1479681497660395600) — Lex/Kai/Nova (read-only for Sage/Forge)
```

### Agent Models
- Sage: `claude-cli/claude-sonnet-4-6` (fallback: opus-4.6)
- Forge: `claude-cli/claude-sonnet-4-6` (fallback: opus-4.6)

### Session Types (CRITICAL)
| Session Type | How Triggered | Tools Available |
|---|---|---|
| **Isolated** (cron) | `sage-fallback-4h` cron fires | Full tools: web_search, Bash, Edit |
| **Group Chat** (Discord ping) | Someone pings @sage in Discord | Text only — NO tools |

**Rule:** Research work (web_search, deep_research.py, spawning Forge) ONLY in isolated cron sessions.

### Key Paths
| What | Path |
|---|---|
| ApexNova repo | `/Users/yujunzou/python/python_repo/trading/apexnova/` |
| nova-brain repo | `/Users/yujunzou/python/python_repo/trading/nova-brain/` |
| Python (with yfinance, httpx) | `/Users/yujunzou/python/python_repo/trading/freqtrade/.venv_working/bin/python3` |
| Gemini API key | `~/.openclaw/gemini_api_key` |
| Paper trader state | `~/Library/Application Support/apexnova/btc_ma_paper_state.json` |

---

## 4. Current Research State (as of 2026-03-11)

### Research Backlog (from RESEARCH_LOOP.md)
| Round | Strategy | Status | Best 1Y Calmar | Notes |
|---|---|---|---|---|
| 1 | RSI Regime Filter | REJECTED | -0.50 | EMA/SMA switching fails in bear market |
| 2 | ATR Volatility Regime | REJECTED | -0.59 | Same root cause as RSI |
| 3 | ETH Validation | REJECTED | -0.71 | BTC params don't transfer to ETH |
| 4 | BidirectionalSMA | Tier 2 (CLOSED) | 1.02 | +46% alpha vs BTC, but MaxDD 30.87% |
| 5 | WeeklyGatedBidirectional | **PENDING** | — | Forge spawned, awaiting results |

**Systematic finding (2026-03-09):** EMA/SMA switching research line CLOSED after 3 consecutive rejections. All failed in BTC bear market (BTC -18.15% 1Y).

**Best strategy found:** BidirectionalSMA(SMA90) — Calmar 1.02, ROI +31.55%, MaxDD 30.87%, 13 trades. Tier 2 (needs Calmar > 4 for Tier 1).

**Next action:** Check Round 5 (WeeklyGatedBidirectional) results. If no results yet, check if Forge cron ran. Then continue research loop.

### Active Crons
| Cron | Schedule | Status | Notes |
|---|---|---|---|
| `sage-fallback-4h` | every 4h | **error** | Needs investigation — may be a tool/session issue |
| `forge-round5-20260310` | one-shot (every 42d) | idle | Round 5 Forge job, never ran? |

### Known Issues
1. **sage-fallback-4h cron shows "error" status** — last ran ~50 min ago with error. Likely Sage tried to use tools in wrong context or hit a permission issue.
2. **Discord urllib 403** — Python's `urllib` gets blocked by Cloudflare. **Fix:** Always use `httpx` for Discord API calls.
3. **Forge old crons lingering** — forge-round3, forge-round4-iter2 still exist with "every 42d" schedule. Should be cleaned up.

---

## 5. Discord Bot Setup

### Bot Tokens (in `~/.openclaw/openclaw.json`)
```python
import json
cfg = json.load(open('/Users/yujunzou/.openclaw/openclaw.json'))
sage_token = cfg['channels']['discord']['accounts']['sage']['token']
forge_token = cfg['channels']['discord']['accounts']['forge']['token']
```

### Posting to Discord (use httpx, NOT urllib)
```python
import httpx, json
cfg = json.load(open('/Users/yujunzou/.openclaw/openclaw.json'))
token = cfg['channels']['discord']['accounts']['sage']['token']
r = httpx.post(
    'https://discord.com/api/v10/channels/1480051404683870480/messages',
    headers={'Authorization': f'Bot {token}', 'Content-Type': 'application/json'},
    json={'content': 'Your message here'}
)
print(r.status_code)
```

### Team Directory (mention with `<@ID>`)
| Agent | Role | Bot ID |
|---|---|---|
| Nova | Orchestrator | `<@1475686344805318787>` |
| Lex | Stock Quant | `<@1479498561719763074>` |
| Kai | Stock Engineer | `<@1479498272593809500>` |
| Sage | Crypto Quant | `<@1477728131690270948>` |
| Forge | Crypto Engineer | `<@1480084559390441552>` |

---

## 6. Deep Research Tool

Gemini 2.5 Pro + Google Search grounding. Used by Sage for strategy research.

```bash
/Users/yujunzou/python/python_repo/trading/freqtrade/.venv_working/bin/python3 \
  /Users/yujunzou/python/python_repo/trading/apexnova/bin/deep_research.py \
  "Your research question here"
```

- API key: `~/.openclaw/gemini_api_key` (working key: `AIzaSyCQF34UTxOEpwvdTxBBhALaSxQCX1qh1LQ`)
- Takes ~40s, uses real Google Search grounding
- Must answer 6 questions: mechanism, edge size, failure modes, param sensitivity, complementary filters, BTC-specific results

**Claude web_search** is for quick lookups only (1-3 searches).

---

## 7. Backtest Conventions

- **Windows:** 1M, 6M, 1Y — always run all three, never beyond 1Y
- **Data:** Pre-cached via `download_crypto_data.py` → `apexnova/bin/data/crypto/`
- **Metric:** Calmar = CAGR / Max DD. Calmar > 4 = Tier 1.
- **Python:** `/Users/yujunzou/python/python_repo/trading/freqtrade/.venv_working/bin/python3`
- **Step 0:** Always run `download_crypto_data.py` before backtesting to refresh data
- **Parameter rule:** Research derives params (not blind sweeps). Target ≤9 runs.
- **Max 2 tuning iterations** per strategy. If still < Calmar 4, reject.

### Loading cached data in scripts:
```python
import sys
sys.path.insert(0, '/Users/yujunzou/python/python_repo/trading/apexnova')
from bin.download_crypto_data import load_cached
df = load_cached('BTC-USD')
```

---

## 8. Paper Trader (MA120 on BinanceUS)

- Runs as macOS LaunchAgent (KeepAlive=true)
- State persists at `~/Library/Application Support/apexnova/btc_ma_paper_state.json`
- Uses BinanceUS public klines API (no auth needed)
- Posts daily status to `#ma120-signals`, BUY/SELL to all channels
- Script: `apexnova/bin/btc_ma_binanceus_trader.py`

---

## 9. Spawning Forge for Backtest

```bash
openclaw cron add \
  --name "forge-roundN-$(date +%Y%m%d)" \
  --at "now" \
  --session isolated \
  --agent forge \
  --timeout-seconds 1200 \
  --delete-after-run \
  --message "You are Forge. Read SOUL.md and MEMORY.md at /Users/yujunzou/python/python_repo/trading/nova-brain/agents/forge/.

Round N task: [strategy name]
Spec: [exact entry/exit/params]

Steps:
1. Run download_crypto_data.py first
2. Write apexnova/bin/<name>_backtest.py
3. Run 1M / 6M / 1Y windows
4. Post results to #yujun-team (1480051404683870480)
5. Commit script to apexnova main
6. Ping Sage: '<@1477728131690270948> Round N backtest done. Calmar X. Your turn.'"
```

---

## 10. Git Repos

| Repo | Branch | Remote | What |
|---|---|---|---|
| `nova-brain/` | main | `apexnova-vc/nova-brain` | Agent docs, research logs |
| `apexnova/` | main | `apexnova-vc/apexnova` | Backtest scripts, trading code |
| `freqtrade/` | develop | (legacy) | GOLDEN_SHEET.md lives here |

### Commit patterns:
```bash
# nova-brain
cd /Users/yujunzou/python/python_repo/trading/nova-brain
git add -A && git commit -m "🔬 Sage: [summary]" && git push origin main

# apexnova
cd /Users/yujunzou/python/python_repo/trading/apexnova
git add bin/<script>.py && git commit -m "Add <strategy> backtest" && git push origin main

# GOLDEN_SHEET
cd /Users/yujunzou/python/python_repo/trading/freqtrade
git add openclaw_setup/GOLDEN_SHEET.md && git commit -m "Update Golden Sheet" && git push origin develop
```

---

## 11. Forbidden Zones (from experience)

| Filter | Where Forbidden | Why |
|---|---|---|
| High-vol exclusion | Mean reversion | Kills signal at peak edge |
| SMA trend filter | Mean reversion entry | -5% drop breaks SMA, rejects best signals |
| Tight stop-loss | BTC trend following | BTC whipsaws kill positions |
| EMA/SMA switching | All (CLOSED) | 3 consecutive failures in bear market |

---

## 12. Immediate TODO for Next Session

1. **Check sage-fallback-4h cron error** — run `openclaw cron list`, investigate why it errored
2. **Check Round 5 (WeeklyGatedBidirectional) results** — did Forge run? Check `#yujun-team` or Forge MEMORY.md
3. **Clean up old Forge crons** — delete forge-round3, forge-round4-iter2 (completed rounds)
4. **If Round 5 has results:** Evaluate → promote/tune/reject → open Round 6
5. **If Round 5 has NOT run:** Investigate, re-trigger Forge
6. **Continue research loop** — next in backlog is "Discovery Mode" (Round 6) — research new strategy class focused on MaxDD reduction

---

## 13. Lessons Learned

- **urllib breaks on macOS for Discord** — always use `httpx` (Cloudflare blocks urllib)
- **Gemini Interactions API (deep-research-pro-preview)** requires enterprise tier — use `gemini-2.5-pro` with Google Search tool instead
- **yfinance CSV has multi-header** — `load_cached()` uses `header=[0,1,2]`
- **OpenClaw group chat = no tools** — research only works in isolated cron sessions
- **3 consecutive rejections = stop and analyze** — don't keep trying the same approach
- **EMA/SMA switching is dead** — closed after systematic analysis (RSI, ATR, Hurst all failed)
- **BidirectionalSMA best so far** — Calmar 1.02 (Tier 2), need to reduce MaxDD for Tier 1
