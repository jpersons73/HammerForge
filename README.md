[README.md](https://github.com/user-attachments/files/26747664/README.md)
<p align="center">
  <img src="https://img.shields.io/badge/SHA--256-Solo%20Mining-orange?style=for-the-badge" alt="SHA-256 Solo Mining" />
  <img src="https://img.shields.io/badge/Coins-12-blue?style=for-the-badge" alt="12 Coins" />
  <img src="https://img.shields.io/badge/Platform-Windows-0078D6?style=for-the-badge&logo=windows" alt="Windows" />
  <img src="https://img.shields.io/badge/NiceHash-Compatible-green?style=for-the-badge" alt="NiceHash Compatible" />
</p>

# HammerForge

### Your Home Node Solution

**Mine 12 SHA-256 coins simultaneously. No pool fees. 100% of block rewards go directly to your wallet.**

HammerForge is a fully automated Windows installer that sets up a private stratum mining server on your PC. It connects directly to your local full nodes, builds real block templates, and broadcasts your blocks to the network when you find one. **You are the pool.**

---

## Supported Coins

| Coin | Ticker | Stratum | RPC | P2P | Node Software |
|:-----|:------:|:-------:|:---:|:---:|:--------------|
| BitcoinII | BC2 | 3333 | 8232 | 8338 | BitcoinII Core v29.1.0 |
| DigiByte | DGB | 3334 | 14022 | 12024 | DigiByte Core |
| Bitcoin | BTC | 3335 | 8332 | 8333 | Bitcoin Core v28.1 |
| Bitcoin Cash | BCH | 3336 | 8432 | 8433 | Bitcoin Cash Node |
| Bitcoin SV | BSV | 3337 | 8532 | 8533 | Bitcoin SV Node |
| eCash | XEC | 3338 | 8632 | 8633 | Bitcoin ABC |
| Fractal Bitcoin | FB | 3339 | 8732 | 8733 | Fractal Bitcoin |
| Peercoin | PPC | 3340 | 9902 | 9901 | Peercoin |
| Litecoin Cash | LCC | 3341 | 62457 | 62458 | Litecoin Cash |
| NitoCoin | NITO | 3342 | 8821 | 8820 | NitoCoin |
| Bitcoin Cash II | BCH2 | 3343 | 8342 | 8339 | Bitcoin Cash II |
| XeroCoin | XRO | 3344 | 25169 | 25170 | XeroCoin |

All 12 coins run concurrently on separate ports. Each coin has its own process, its own desktop shortcut, and talks to its own full node.

---

## How It Works

```
Your ASIC Miner(s)            NiceHash Rental
       |                            |
       |  stratum+tcp               |  stratum+tcp
       v                            v
  +-------------------------------------------+
  |      HammerForge Stratum Server           |
  |   (Node.js - one instance per coin)       |
  |                                           |
  |   - Builds block templates from your node |
  |   - Assigns work to miners                |
  |   - Validates shares (SHA-256d)           |
  |   - Submits blocks when found             |
  |   - Variable difficulty (vardiff)         |
  |   - UPnP auto port forwarding            |
  +-------------------------------------------+
              |
              |  JSON-RPC (localhost)
              v
  +-------------------------------------------+
  |      Your Full Node Wallet                |
  |   (Bitcoin Core, DigiByte Core, etc.)     |
  |                                           |
  |   - Fully synced blockchain               |
  |   - getblocktemplate for real templates    |
  |   - submitblock when you find a block     |
  |   - Block reward goes to YOUR wallet      |
  +-------------------------------------------+
```

### Mining Flow

1. **Block Template** - Server calls `getblocktemplate` on your local node for real block data
2. **Job Construction** - Builds a stratum job with the coinbase paying YOUR wallet address
3. **Work Distribution** - Each miner gets a unique `extranonce1` so no duplicate work
4. **Share Submission** - Miner finds a hash below share target, server validates via SHA-256d
5. **Block Found** - If the hash meets network difficulty, the full block is submitted via `submitblock`

### Difficulty System

| Setting | Value | Purpose |
|:--------|:------|:--------|
| Default Difficulty | 65,536 | Starting difficulty for home ASIC miners |
| NiceHash Min Difficulty | 500,000 | Minimum for NiceHash rentals (auto-detected) |
| Max Difficulty | 4,000,000,000 | Upper ceiling |
| Target Share Time | 5 seconds | How often a miner should submit a share |
| Retarget Interval | 15 seconds | How often difficulty is recalculated |
| Variance | 40% | Acceptable deviation from target share time |

- **Password-based difficulty** - Miners can set `diff=500000` in the password field
- **NiceHash auto-detection** - User agents containing "NiceHash" or "nhmp" get minimum 500k difficulty
- **Diff1 Target (pdiff)** - Uses `0x00000000FFFF...` for accurate NiceHash compatibility

---

## Installation Guide

### Prerequisites

- **OS:** Windows 10/11 (64-bit)
- **Hardware:** SHA-256 miner on your LAN (Bitaxe, MagicMiner, Luckyminer, Avalon Nano, or any SHA-256 ASIC)
- **RAM:** 8 GB minimum recommended for multiple coin nodes
- **Disk:** Depends on coins and pruning (see [Pruning Guide](#blockchain-pruning-guide))

### Step 1: Download and Extract

1. Download the HammerForge ZIP package
2. Right-click the ZIP > **Extract All...**
3. Extract to any folder (e.g. Desktop or `C:\HammerForge`)
4. Open the folder and launch `START-HERE.html` for an interactive setup guide

### Step 2: Install Tailscale (Optional)

Install Tailscale **before** the mining installer if you want remote access from your phone or other devices. The installer auto-detects Tailscale and configures everything. See [Remote Access with Tailscale](#remote-access-with-tailscale) for details.

### Step 3: Install Wallet(s)

Right-click each installer and select **Run as administrator**:

| Installer | Coin |
|:----------|:-----|
| `INSTALL-BC2-CORE.bat` | BitcoinII |
| `INSTALL-DGB-CORE.bat` | DigiByte |
| `INSTALL-BTC-CORE.bat` | Bitcoin |
| `INSTALL-BCH-CORE.bat` | Bitcoin Cash |
| `INSTALL-BSV-CORE.bat` | Bitcoin SV |
| `INSTALL-XEC-CORE.bat` | eCash |
| `INSTALL-FB-CORE.bat` | Fractal Bitcoin |
| `INSTALL-PPC-CORE.bat` | Peercoin |
| `INSTALL-LCC-CORE.bat` | Litecoin Cash |
| `INSTALL-NITO-CORE.bat` | NitoCoin |
| `INSTALL-BCH2-CORE.bat` | Bitcoin Cash II |
| `INSTALL-XRO-CORE.bat` | XeroCoin |

Each installer automatically downloads the wallet, creates the data directory, offers blockchain pruning, configures RPC credentials, adds firewall rules, and creates a desktop shortcut. You can choose a custom install location or use the default (`C:\Program Files\<CoinName>`).

**After installing, open each wallet and let the blockchain sync completely before mining.**

### Step 4: Blockchain Pruning

During installation you will be asked to enable pruning. Type **`y`** to save disk space or **`n`** for the full chain. See [Blockchain Pruning Guide](#blockchain-pruning-guide) for details.

### Step 5: Set Up Mining

Right-click `INSTALL-MINING.bat` > **Run as administrator**. The installer walks you through 7 steps:

| Step | What Happens |
|:-----|:-------------|
| 1a/7 | Checks for / installs **Node.js v20** |
| 1b/7 | Checks for Tailscale (auto-detects if installed) |
| 2/7 | Detects which coin wallets are installed |
| 3/7 | Prompts for your wallet address per coin (Enter to skip) |
| 4/7 | Email notification setup (optional) |
| 5/7 | Configures wallet `.conf` files with RPC settings (backup copy saved to coin folder) |
| 6/7 | Installs stratum server dependencies (`npm install`) |
| 7/7 | Creates configs, batch scripts, firewall rules, and desktop shortcuts |

### Step 6: Start Mining

1. Make sure your wallet is open and fully synced
2. Double-click the desktop shortcut (e.g. "BC2 HammerForge")
3. The CMD window shows the server log and the monitor opens in your browser

**You can run all 12 coins simultaneously.** Each runs as its own process on its own port.

> **Port conflict recovery:** If an old instance is still running, the server auto-detects the conflict, kills the old process, and takes over the port. Miners briefly disconnect then reconnect automatically.

---

## Configure Your Miners

Point your SHA-256 ASIC miners to:

```
URL:      stratum+tcp://YOUR_PC_IP:PORT
Worker:   YOUR_WALLET_ADDRESS.MinerName
Password: x
```

Refer to the [Supported Coins](#supported-coins) table for each coin's stratum port.

**Examples:**

```
Mining BC2 with Bitaxe:       stratum+tcp://192.168.1.100:3333
Mining DGB with MagicMiner:   stratum+tcp://192.168.1.100:3334
Mining BTC with NiceHash:     stratum+tcp://YOUR_PUBLIC_IP:3335
```

### Password Field Options

| Password | Effect |
|:---------|:-------|
| `x` | Standard mining |
| `diff=500000` | Set custom starting difficulty |
| `notif=you@gmail.com` | Email notifications for this miner |
| `diff=65536,notif=you@gmail.com` | Both combined |

---

## Supported Miners

| Miner | Type | Notes |
|:------|:-----|:------|
| MagicMiner | USB ASIC | All models |
| Luckyminer LV06 | USB ASIC | SHA-256 mode |
| Avalon Nano 3S | USB ASIC | Low-power home miner |
| Bitaxe | Open-source ASIC | All variants |
| NiceHash SHA-256 | Hashrate rental | Auto-detected, optimized difficulty |
| Any SHA-256 stratum miner | ASIC/Software | Standard stratum v1 protocol |

---

## NiceHash Integration

HammerForge is fully compatible with NiceHash SHA-256 hash rentals.

1. Start mining any coin (UPnP auto-opens the port on your router)
2. Note your public IP from the CMD window: `UPnP: External/public IP is X.X.X.X`
3. Create a NiceHash order: Algorithm **SHA256**, Pool `stratum+tcp://YOUR_PUBLIC_IP:PORT`

**NiceHash features:**
- Auto-detection via user agent ("NiceHash" / "nhmp") with 500k minimum difficulty
- Full `mining.configure` support with version-rolling (AsicBoost)
- `mining.extranonce.subscribe` support
- Rapid vardiff (15-second retarget) for rental start/stop

---

## Web Monitoring Dashboard

The dashboard opens automatically at `http://localhost:5000/monitor` when you start mining:

- **Hashrate chart** - Real-time graph with 30s intervals (up to 24h history)
- **Best share chart** - Logarithmic bar chart of highest difficulty shares (5-min intervals)
- **Worker table** - All miners with 1m, 5m, and 1h hashrate averages
- **Recent shares** - Latest submissions with difficulty and status
- **Blocks found** - Discovered blocks with height, hash, reward, and acceptance
- **Network stats** - Block height, difficulty, hashrate, and coin price
- **Best share & uptime** - Session stats
- **Donate dropdown** - Support addresses (click to copy)

Auto-refreshes every 5 seconds. Demo mode available at `/monitor?demo=true`.

---

## Block Found Notification

When you find a block:

1. **Popup window** on your Windows desktop with full block details
2. **Email notification** (if configured) with formatted HTML details
3. **Log entry** appended to `logs/blocks-found.log`

Blocks are submitted immediately via `submitblock`. The reward goes directly to your configured wallet address.

---

<details>
<summary><strong>Email Notifications Setup</strong></summary>

### Setting Up SMTP During Installation

During Step 4/7 of `INSTALL-MINING.bat`, choose to set up email notifications. The installer offers presets:

| Preset | SMTP Server |
|:-------|:------------|
| Gmail | smtp.gmail.com |
| Outlook | smtp.office365.com |
| Yahoo | smtp.mail.yahoo.com |
| Custom | Your own SMTP server |

You will enter your email address, app-specific password, and notification recipient.

### Gmail App Password

1. Go to [myaccount.google.com](https://myaccount.google.com) > **Security**
2. Enable **2-Step Verification**
3. Go to **App passwords** > Select **Mail** / **Windows Computer**
4. Click **Generate** and copy the 16-character password
5. Paste into the installer when prompted

### Outlook App Password

1. Go to [account.microsoft.com/security](https://account.microsoft.com/security)
2. **Advanced security options** > **Create a new app password**

### Yahoo App Password

1. Go to [login.yahoo.com/account/security](https://login.yahoo.com/account/security)
2. **Generate app password** > Select **Other App**, name it "HammerForge"

### Per-Miner Email Override

Set the notification recipient per miner via the password field:

```
Password: notif=you@gmail.com
Password: diff=65536,notif=you@gmail.com
```

</details>

---

<details>
<summary><strong>Remote Access with Tailscale</strong></summary>

Tailscale creates a private encrypted network between your devices so you can access the mining monitor from anywhere without opening router ports.

### Setup

1. Download from [tailscale.com/download/windows](https://tailscale.com/download/windows)
2. Install and sign in (Google, Microsoft, or GitHub - free for personal use)
3. Verify: run `tailscale ip -4` in a command prompt (should show `100.x.x.x`)
4. Install on your phone (iOS App Store / Google Play) with the same account

### How It Works with HammerForge

When you run `INSTALL-MINING.bat`, it auto-detects Tailscale and configures remote access. When mining starts, the CMD window shows:

```
Remote access: http://100.64.0.2:5000/monitor
```

Open that URL on your phone to see the full live dashboard.

### QR Code

The monitor includes a **QR Code** button in the remote access bar. Scan with your phone for instant access.

### Already Mining Without Tailscale?

Either re-run `INSTALL-MINING.bat` or manually browse to `http://YOUR_TAILSCALE_IP:5000/monitor`.

</details>

---

<details>
<summary><strong>BTC Fast Sync: UTXO Snapshots</strong></summary>

Bitcoin Core v28+ supports **AssumeUTXO** snapshots that let you start mining BTC in under an hour instead of waiting days.

### What Is It?

A UTXO snapshot is a verified checkpoint of every unspent bitcoin at block height 840,000. Bitcoin Core loads it and immediately has a working view of the chain. Background validation continues automatically.

### Steps

1. **Install Bitcoin Core** via `INSTALL-BTC-CORE.bat`
2. **Launch Bitcoin Core** and let it start syncing
3. **Download** `utxo-840000.dat` (~12 GB) from [github.com/nicehash/bitcoin-utxo-snapshot](https://github.com/nicehash/bitcoin-utxo-snapshot)
4. **Load the snapshot** while Bitcoin Core is running:

```cmd
"C:\Program Files\Bitcoin\bin\bitcoin-cli.exe" -rpcclienttimeout=0 loadtxoutset "C:\Users\YourName\Downloads\utxo-840000.dat"
```

5. **Wait for remaining sync** (minutes to hours from block 840,000 to current tip)
6. **Start mining** BTC

### Snapshot + Pruning

| Setup | Disk Usage | Sync Time |
|:------|:-----------|:----------|
| Snapshot + Pruning | ~10 GB | ~30 minutes |
| Snapshot + Full Chain | 600+ GB (grows over time) | ~30 min + background |
| No Snapshot + Pruning | ~10 GB | Days |
| No Snapshot + Full Chain | 600+ GB | Days |

</details>

---

<details>
<summary><strong>Blockchain Pruning Guide</strong></summary>

Pruning keeps only the most recent block data (~1 month) instead of the entire chain history. Mining works perfectly with pruning enabled.

### Pruning Sizes by Coin

| Coin | Prune Value | Pruned Size | Full Chain | Savings |
|:-----|:-----------:|:-----------:|:----------:|:-------:|
| BC2 | 5000 | ~5 GB | Small | Minimal |
| DGB | 5000 | ~5 GB | ~40+ GB | ~35 GB |
| BTC | 10000 | ~10 GB | ~600+ GB | ~590 GB |
| BCH | 10000 | ~10 GB | ~200+ GB | ~190 GB |
| BSV | 10000 | ~10 GB | ~400+ GB | ~390 GB |
| XEC | 10000 | ~10 GB | ~200+ GB | ~190 GB |
| FB | 5000 | ~5 GB | Small | Minimal |
| PPC | 2000 | ~2 GB | Small | Minimal |
| LCC | 2000 | ~2 GB | Small | Minimal |
| NITO | 2000 | ~2 GB | Small | Minimal |
| BCH2 | 2000 | ~2 GB | Small | Minimal |
| XRO | 2000 | ~2 GB | Small | Minimal |

> **Recommendation:** Enable pruning for BTC, BCH, BSV, and XEC unless you have a large drive.

### Config File Locations

| Coin | Config File |
|:-----|:------------|
| BC2 | `%APPDATA%\BitcoinII\bitcoinII.conf` |
| DGB | `%APPDATA%\DigiByte\digibyte.conf` |
| BTC | `%APPDATA%\Bitcoin\bitcoin.conf` |
| BCH | `%APPDATA%\BitcoinCash\bitcoin.conf` |
| BSV | `%APPDATA%\BitcoinSV\bitcoin.conf` |
| XEC | `%APPDATA%\BitcoinABC\bitcoin.conf` |
| FB | `%APPDATA%\FractalBitcoin\bitcoin.conf` |
| PPC | `%APPDATA%\Peercoin\peercoin.conf` |
| LCC | `%APPDATA%\LitecoinCash\litecoincash.conf` |
| NITO | `%APPDATA%\Nito\nito.conf` |
| BCH2 | `%APPDATA%\BitcoinCashII\bitcoincashII.conf` |
| XRO | `%APPDATA%\Xero\xero.conf` |

To change pruning after installation, close the wallet, edit the config file, set `prune=VALUE` (or `prune=0` to disable), save, and restart the wallet.

</details>

---

## UPnP Auto Port Forwarding

The stratum server automatically maps ports on your router via UPnP:

- No manual port forwarding needed
- NiceHash and external miners can connect immediately
- Mappings refresh hourly with a 2-hour TTL
- Auto-cleaned up when the server stops
- Server continues normally if UPnP is unavailable

---

<details>
<summary><strong>Stratum Protocol Details</strong></summary>

### Supported Methods

| Method | Support | Notes |
|:-------|:-------:|:------|
| `mining.subscribe` | Full | Returns extranonce1 and extranonce2 size |
| `mining.authorize` | Full | Accepts any wallet.worker format |
| `mining.submit` | Full | SHA-256d validation with block submission |
| `mining.notify` | Full | Real block templates from your node |
| `mining.set_difficulty` | Full | Vardiff with per-miner adjustment |
| `mining.configure` | Full | Version-rolling (AsicBoost) support |
| `mining.suggest_difficulty` | Full | Miners can request starting difficulty |
| `mining.extranonce.subscribe` | Full | NiceHash compatibility |

### Coinbase Transaction

- Block height encoded in script (BIP34)
- Coinbase tag identifying the solo miner (e.g. `/BC2Solo/`, `/DGBSolo/`)
- Output paying the full block reward to your wallet
- SegWit commitment (where applicable)

### Share Validation

1. Reconstruct coinbase: `coinb1 + extranonce1 + extranonce2 + coinb2`
2. Build merkle root from coinbase hash + merkle branches
3. Construct the 80-byte block header
4. SHA-256d (double SHA-256) the header
5. Compare against share difficulty target
6. If it also meets the network target, submit the full block

### Fallback Templates

If a coin's node is unavailable, the server generates fallback templates so miners stay connected. Blocks cannot be found with fallback templates. The CMD window warns:

```
*** WARNING: BC2 node STILL not available (RPC port 8232).
    Miners are hashing on FALLBACK templates -- blocks CANNOT be found!
    Sync your node! ***
```

Once your node comes online, the server auto-switches to real templates.

</details>

---

<details>
<summary><strong>Configuration Files</strong></summary>

Each coin's config is stored as JSON in the HammerForge directory:

```json
{
  "walletAddress": "bc1qYourAddress...",
  "stratumPort": 3333,
  "rpcHost": "127.0.0.1",
  "rpcPort": 8232,
  "rpcUser": "bc2rpc",
  "rpcPassword": "auto-generated-password",
  "coin": "BC2",
  "dataDir": "C:\\Users\\you\\AppData\\Roaming\\BitcoinII",
  "email": {
    "enabled": true,
    "smtpHost": "smtp.gmail.com",
    "smtpPort": 587,
    "smtpUser": "you@gmail.com",
    "smtpPass": "your-app-password",
    "to": "you@gmail.com",
    "useTls": true
  }
}
```

| Config File | Coin |
|:------------|:-----|
| `config-bc2.json` | BitcoinII |
| `config-dgb.json` | DigiByte |
| `config-btc.json` | Bitcoin |
| `config-bch.json` | Bitcoin Cash |
| `config-bsv.json` | Bitcoin SV |
| `config-xec.json` | eCash |
| `config-fb.json` | Fractal Bitcoin |
| `config-ppc.json` | Peercoin |
| `config-lcc.json` | Litecoin Cash |
| `config-nito.json` | NitoCoin |
| `config-bch2.json` | Bitcoin Cash II |
| `config-xro.json` | XeroCoin |

Created automatically by `INSTALL-MINING.bat`. Edit manually if needed.

### RPC Authentication

The stratum server supports two authentication methods for connecting to your coin node:

1. **Cookie auth** (preferred) - Reads the `.cookie` file from the wallet's data directory. Most modern wallets use this automatically.
2. **User/password auth** - Uses `rpcUser` and `rpcPassword` from the config JSON.

The server tries cookie auth first, then falls back to user/password. If your wallet restarts, the cookie file is regenerated automatically and the server picks up the new credentials.

### Backup Node Configs

During mining setup, a backup copy of each coin's node config file (e.g. `digibyte.conf`, `bitcoin.conf`) is saved to the coin's folder inside the HammerForge directory (e.g. `HammerForge\DGB\digibyte.conf`). If the auto-write to the wallet data directory fails, you can find the correct config in the coin folder and copy it manually.

</details>

---

## Uninstall

Right-click `UNINSTALL.bat` > **Run as administrator**. Removes:

- Stratum mining server files
- Desktop shortcuts
- Firewall rules

Your blockchain data and wallets are **never** touched.

---

## File Structure

```
HammerForge/
  START-HERE.html             Interactive setup guide (open this first)
  INSTALL-BC2-CORE.bat        BitcoinII Core installer
  INSTALL-DGB-CORE.bat        DigiByte Core installer
  INSTALL-BTC-CORE.bat        Bitcoin Core installer
  INSTALL-BCH-CORE.bat        Bitcoin Cash Node installer
  INSTALL-BSV-CORE.bat        Bitcoin SV Node installer
  INSTALL-XEC-CORE.bat        Bitcoin ABC (eCash) installer
  INSTALL-FB-CORE.bat         Fractal Bitcoin installer
  INSTALL-PPC-CORE.bat        Peercoin installer
  INSTALL-LCC-CORE.bat        Litecoin Cash installer
  INSTALL-NITO-CORE.bat       NitoCoin installer
  INSTALL-BCH2-CORE.bat       Bitcoin Cash II installer
  INSTALL-XRO-CORE.bat        XeroCoin installer
  INSTALL-MINING.bat          Mining setup (Node.js + stratum server)
  UNINSTALL.bat               Clean uninstaller
  OPEN-MONITOR.bat            Opens mining monitor in browser
  README.txt                  Quick reference guide
  server/
    stratum.ts                Stratum protocol implementation
    rpc.ts                    JSON-RPC client for full nodes
    routes.ts                 Web dashboard API routes
    storage.ts                In-memory state management
    email.ts                  Email notifications
    monitor.html              Mining dashboard (served on Windows)
    index.ts                  Server entry point
  shared/
    schema.ts                 TypeScript type definitions
  client/src/pages/
    dashboard.tsx             Download page UI
    monitor.tsx               Mining monitor (React version)
```

---

## Support Development

If you find this project useful, consider donating:

| Coin | Address |
|:-----|:--------|
| BTC | `bc1qczmq77nrrqsxn3uk8p007r9pf07uzq2t96w3g5` |
| BCH | `qrqtvrm6vvvzq6w8jcu9alcv599lmsgpfv7a87edaa` |
| DGB | `DNi4NsZKH93QxBWn1RfQYizGtrKVhJcz6q` |
| ETH | `0x996c85e9137a9020C2a3d9b4889aA6a35185d368` |
| LTC | `Lcnv6pvW4PPBfz2LSyf9GytSDvxUWwaiSB` |
| USDC (ERC20) | `0x996c85e9137a9020C2a3d9b4889aA6a35185d368` |
| XEC | `ecash:qrqtvrm6vvvzq6w8jcu9alcv599lmsgpfv8sn4zh` |
| XRP | `rJZxqccCyj93RBLBGqCqzxFgr5bURGJrrm` |

---

## Legal

**No Warranty.** This software is provided "as is" without warranty of any kind. The author is not liable for any damages, lost funds, lost private keys, or any other losses arising from use of this software. Solo mining involves financial risk. Block rewards are not guaranteed.

**Security.** You are solely responsible for securing your wallet private keys and RPC credentials. Never share your wallet.dat or RPC passwords.

---

## License

Copyright (c) 2025 James Vernon Persons. All Rights Reserved.

This software and all associated files, scripts, and documentation are the proprietary property of James Vernon Persons. No part of this software may be reproduced, distributed, transmitted, modified, reverse-engineered, or used in any form without the express written permission of the copyright holder.

Unauthorized copying, distribution, or modification of this software is strictly prohibited and may result in legal action.
