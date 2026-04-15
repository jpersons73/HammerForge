[README.md](https://github.com/user-attachments/files/26747919/README.md)
<p align="center">
  <img src="https://img.shields.io/badge/SHA--256-Solo%20Mining-orange?style=for-the-badge" alt="SHA-256 Solo Mining"/>
  <img src="https://img.shields.io/badge/Coins-12-blue?style=for-the-badge" alt="12 Coins"/>
  <img src="https://img.shields.io/badge/Platform-Windows-0078D6?style=for-the-badge&logo=windows" alt="Windows"/>
  <img src="https://img.shields.io/badge/Fees-0%25-brightgreen?style=for-the-badge" alt="Zero Fees"/>
</p>

<h1 align="center">HammerForge</h1>
<h3 align="center">Your Home Node Mining Solution</h3>

<p align="center">
  Fully automated SHA-256 solo mining for <b>12 cryptocurrencies</b> running concurrently on your Windows PC.<br/>
  No pool fees. No third parties. <b>100% of every block reward goes directly to YOUR wallet.</b>
</p>

<p align="center">
  <a href="#quick-start">Quick Start</a> &bull;
  <a href="#supported-coins">Supported Coins</a> &bull;
  <a href="#features">Features</a> &bull;
  <a href="#miner-setup">Miner Setup</a> &bull;
  <a href="#troubleshooting">Troubleshooting</a> &bull;
  <a href="#faq">FAQ</a>
</p>

---

## Overview

HammerForge turns any Windows PC into a **private mining pool** capable of running up to 12 cryptocurrency nodes simultaneously, each with its own Stratum server and web dashboard.

Unlike pool mining, you run the full node, you own the keys, and you keep every satoshi of the block reward.

**Designed for:**
- Home ASIC miners (Bitaxe, MagicMiner, Avalon Nano, etc.)
- NiceHash hashrate rentals
- Small/medium SHA-256 mining operations
- Cryptocurrency enthusiasts supporting decentralization

---

## Supported Coins

| Ticker | Coin | Stratum Port | RPC Port |
|:------:|:-----|:------------:|:--------:|
| **BC2** | BitcoinII | 3333 | 8232 |
| **DGB** | DigiByte (SHA-256) | 3334 | 14022 |
| **BTC** | Bitcoin | 3335 | 8332 |
| **BCH** | Bitcoin Cash | 3336 | 8432 |
| **BSV** | Bitcoin SV | 3337 | 8532 |
| **XEC** | eCash | 3338 | 8632 |
| **FB** | Fractal Bitcoin | 3339 | 8732 |
| **PPC** | Peercoin | 3340 | 9902 |
| **LCC** | Litecoin Cash | 3341 | 62457 |
| **NITO** | NitoCoin | 3342 | 8821 |
| **BCH2** | Bitcoin Cash II | 3343 | 8342 |
| **XRO** | XeroCoin | 3344 | 25169 |

Each coin runs as a **completely independent process** with its own node, wallet, stratum server, and web dashboard. Run any combination simultaneously.

---

## System Requirements

| | Minimum | Recommended |
|:--|:--------|:------------|
| **OS** | Windows 10 (64-bit) | Windows 11 (64-bit) |
| **RAM** | 8 GB | 32 GB (for all 12 coins) |
| **Disk** | 50 GB free (pruned) | SSD with 100+ GB |
| **Network** | Broadband internet | Stable, always-on connection |
| **Runtime** | Node.js v20+ | Auto-installed by installer |

> **Disk Space with Pruning:** BTC uses ~10 GB (pruned from 600+ GB). Other coins typically use 1-5 GB each. HammerForge enables pruning by default.

---

## Quick Start

### Step 1: Download & Extract
Download the latest release and extract the ZIP to any folder (e.g., `C:\HammerForge`).

### Step 2: Install Coin Nodes
Run the installer for each coin you want to mine:
```
Double-click:  INSTALL-BTC-CORE.bat
Double-click:  INSTALL-BCH2-CORE.bat
```

Each installer automatically:
- Downloads the official coin node software
- Configures RPC, pruning, and firewall rules
- Creates a wallet and generates a receiving address
- Launches the node to begin blockchain sync

### Step 3: Wait for Sync
Let each coin node fully synchronize. Check progress in each wallet's status bar.

| Coin | Typical Sync Time |
|:-----|:------------------|
| BTC | 4-12 hours (with snapshot) |
| BCH, BSV | 2-6 hours |
| Small coins (BC2, NITO, etc.) | Minutes to 1 hour |

### Step 4: Install Mining Software
```
Double-click:  Mining\INSTALL-MINING.bat
```
This will:
- Install Node.js v20 (if needed)
- Prompt for wallet addresses for each installed coin
- Optionally configure email notifications
- Create desktop shortcuts for each coin

### Step 5: Start Mining
Double-click any desktop shortcut (e.g., **"BTC HammerForge"**), then point your miner to:
```
stratum+tcp://YOUR_PC_IP:PORT
```

---

## Architecture

```
+-------------------+     +--------------------+     +------------------+
|   ASIC Miner      |     |    HammerForge     |     |    Coin Node     |
|  (Bitaxe, etc.)   | <=> |  Stratum Server    | <=> |   (Full Node)    |
|                   |     |  + Web Dashboard   |     |   + Wallet       |
+-------------------+     +--------------------+     +------------------+
      Stratum V1              Node.js / TS              JSON-RPC
```

**Layer 1 - Coin Full Nodes:** Official coin software validating transactions and providing block templates via `getblocktemplate`.

**Layer 2 - HammerForge Stratum Server:** Implements Stratum V1, distributes jobs to miners, validates shares, submits found blocks, and hosts the monitoring dashboard.

**Layer 3 - Mining Hardware:** Any SHA-256 ASIC miner or NiceHash rental connecting via standard Stratum V1.

### Key Server Components

| File | Purpose |
|:-----|:--------|
| `server/stratum.ts` | Stratum V1 protocol, share validation, block submission |
| `server/rpc.ts` | JSON-RPC communication with coin nodes |
| `server/routes.ts` | REST API and web dashboard |
| `server/storage.ts` | In-memory state and persistent block log |
| `server/email.ts` | SMTP notification system |

---

## Features

### Mining Engine
- Full **Stratum V1** protocol implementation
- SHA-256 double-hash share validation
- **Fast-path block submission** (submit first, verify after)
- **Variable difficulty (Vardiff)** targeting 15-second share intervals
- **NiceHash auto-detection** with 500,000 minimum difficulty
- **AsicBoost / version-rolling** support (`mining.configure`)
- Concurrent mining of all 12 coins on separate ports
- Near-miss share logging (tracks close calls)

### Block Submission Pipeline
When a block-level hash is found, HammerForge uses a zero-delay pipeline:

```
DETECT  -->  BUILD (<1ms)  -->  SUBMIT (immediate)  -->  VERIFY (async)  -->  NOTIFY
```

1. **Detect** - Share hash compared against network target
2. **Build** - Block hex assembled in memory (sub-millisecond)
3. **Submit** - `submitblock` RPC call fired immediately
4. **Verify** - Post-submit chain confirmation runs asynchronously
5. **Notify** - Desktop popup, email, dashboard update

> Submit latency is timed and logged for every block (build time + RPC time in ms).

### Wallet Address Support

| Format | Prefixes |
|:-------|:---------|
| **Bech32** | `bc1q`, `dgb1q`, `nito1q`, `lcc1q`, `xro1q`, `pc1q`, `fb1q`, `firo1q` |
| **Bech32m** | `bc1p`, `dgb1p`, `nito1p`, `lcc1p`, `xro1p`, `pc1p`, `fb1p`, `firo1p` |
| **Base58** | `1`, `3`, `D`, `N`, `P`, `C`, `a`, `L`, `F`, `E`, `M`, `Z` |
| **CashAddr** | `bitcoincash:q`, `bitcoincashii:q`, `ecash:q` |
| **RPC Fallback** | Automatic resolution via `validateaddress` for unknown formats |

Address decoding is verified at startup with a clear log message.

### Web Dashboard
- Real-time hashrate monitoring with interactive charts
- Per-worker statistics (hashrate, shares, difficulty, uptime)
- Network difficulty and block height tracking
- External difficulty comparison (detects wrong-fork scenarios)
- Blocks found log with status tracking
- Responsive design (desktop, tablet, mobile)

### Notifications
- **Windows desktop popup** when a block is found
- **SMTP email alerts** (configurable globally or per-worker)
- Block status tracking: ACCEPTED / ORPHANED / STALE / REJECTED

### Network & Remote Access
- **UPnP** automatic port forwarding
- **Tailscale** integration for secure remote monitoring
- External miner support (mine from anywhere)

---

## Miner Setup

### Bitaxe
1. Open your Bitaxe web interface
2. Go to **Settings**
3. Configure:
   - **Pool URL:** `YOUR_PC_IP`
   - **Pool Port:** `3335` (for BTC, see port table for other coins)
   - **Pool User:** `YOUR_WALLET_ADDRESS.worker1`
   - **Pool Pass:** `x`
4. Save and restart

### NiceHash
1. Log into NiceHash and create a new SHA-256 order
2. Set stratum URL: `stratum+tcp://YOUR_PC_IP:3335`
3. HammerForge auto-detects NiceHash and sets appropriate difficulty

### Generic ASIC
```
URL:   stratum+tcp://YOUR_PC_IP:PORT
User:  WALLET_ADDRESS.WORKER_NAME
Pass:  x
```

### Password Field Options

| Option | Example | Description |
|:-------|:--------|:------------|
| `diff=X` | `diff=50000` | Set custom starting difficulty |
| `notif=EMAIL` | `notif=me@mail.com` | Override notification email for this worker |
| Combined | `diff=100000,notif=me@mail.com` | Both options together |

---

## Configuration

Each coin's configuration is stored in `config-[coin].json`:

```json
{
  "coin": "BTC",
  "walletAddress": "bc1q...",
  "rpcHost": "127.0.0.1",
  "rpcPort": 8332,
  "rpcUser": "btcrpc",
  "rpcPassword": "generated-password",
  "stratumPort": 3335,
  "email": {
    "enabled": false,
    "smtpHost": "",
    "smtpPort": 587,
    "smtpUser": "",
    "smtpPass": "",
    "from": "",
    "to": ""
  }
}
```

### Vardiff Settings

| Parameter | Value |
|:----------|:------|
| Target share time | 15 seconds |
| Retarget interval | 15 seconds |
| Variance | 40% |
| Min difficulty | 1 |
| Max difficulty | 4,000,000,000,000 |
| NiceHash minimum | 500,000 (auto) |

### Coin-Specific Handling

| Coins | Special Handling |
|:------|:-----------------|
| BCH, XEC, BCH2 | CTOR transaction ordering, no SegWit |
| BSV | No SegWit |
| BTC, BC2, DGB, FB, LCC, NITO, XRO | SegWit with witness commitment |
| PPC | Block signature field, transaction timestamps |
| DGB | SHA-256 algo version bits (0x200) |

---

## Troubleshooting

### "Cannot reach [COIN] node" / RPC connection failed
- Ensure the coin node (wallet) is **running and fully synced**
- Check that the RPC port is not blocked by a firewall
- Verify RPC credentials in `config-[coin].json` match the coin's `.conf` file

### Miner connects but shows 0 hashrate
- Wait 30-60 seconds for hashrate calculation to stabilize
- Check that the miner is receiving jobs (look for `mining.notify` in logs)

### Block found but reward not in wallet
- Check block status in dashboard: **ACCEPTED** vs **ORPHANED**
- Coinbase rewards require **~100 confirmations** to mature (this is normal)
- Look for `"Wallet address verified"` in the startup log
- If ORPHANED, another miner's block won the race

### "Could not decode wallet address"
- The address format is not recognized by the local decoder
- Ensure the coin node is running (RPC fallback will be attempted)
- Look for `CRITICAL` messages in the startup log

### Blocks always orphaned
- Check **peer count**: 0 outbound peers = blocks cannot propagate
- Ensure the node is **fully synced** (not in Initial Block Download)
- Add more seed nodes to the coin's `.conf` file
- Check submit latency in logs (should be **under 100ms**)

### UPnP port mapping failed
- Your router may not support UPnP or it may be disabled
- Manually forward the stratum port in your router settings

---

## FAQ

**How much hashrate do I need to find a block?**
> It depends on the coin's network difficulty. For small coins (BC2, NITO, BCH2), even a single Bitaxe (~500 GH/s) can find blocks regularly. For BTC, you need significantly more hashrate or extraordinary luck.

**Is there a pool fee?**
> No. HammerForge is solo mining. You get **100% of every block reward** directly to your wallet. Zero fees.

**Can I mine multiple coins at the same time?**
> Yes. Each coin runs as a separate process on its own port. You can run all 12 simultaneously.

**Does HammerForge work with NiceHash?**
> Yes. NiceHash connections are auto-detected with appropriate settings (minimum difficulty, AsicBoost).

**Do I need to keep the coin wallet open?**
> Yes. The coin node must be running and synced. HammerForge communicates with it via JSON-RPC.

**What happens if my node goes down while mining?**
> HammerForge detects the RPC failure and displays a warning. Mining pauses until the node is back online. No blocks can be lost.

**What is block pruning?**
> Pruning reduces disk usage by only keeping recent blocks. A pruned BTC node uses ~10 GB instead of 600+ GB. Mining works perfectly with pruned nodes.

**Why was my block orphaned?**
> Another miner found a valid block at the same height, and the network chose theirs. This is normal mining competition. HammerForge's fast-path submission minimizes this risk.

**What is Vardiff?**
> Variable Difficulty automatically adjusts mining difficulty to maintain ~1 share every 15 seconds. This optimizes bandwidth and provides smoother hashrate reporting.

**Can I access the dashboard remotely?**
> Yes, via Tailscale (recommended) or by port-forwarding port 5000 through your router.

---

## Uninstalling

Individual coin uninstallers and a master uninstaller are provided:

```
Uninstall\UNINSTALL.bat                    -- Remove everything
[COIN]\UNINSTALL-[COIN]-CORE.bat           -- Remove a specific coin
```

---

## Built With

- **Node.js** & **TypeScript** - Stratum server engine
- **React** & **Vite** - Web dashboard frontend
- **Tailwind CSS** & **Shadcn UI** - Dashboard styling
- **Recharts** - Hashrate visualization

---

<p align="center">
  <b>Mine your own blocks. Run your own node. Keep 100% of the rewards.</b><br/>
  <i>"Your Home Node Solution"</i>
</p>
