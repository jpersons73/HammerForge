[README.md](https://github.com/user-attachments/files/26265563/README.md)
# HammerForge - SHA-256 Multi-Coin

**Mine 10 SHA-256 coins simultaneously with your own hardware and NiceHash rentals. No pool fees. 100% of block rewards go directly to your wallet.**

HammerForge is a fully automated Windows installer package that sets up a private stratum mining server on your PC. It connects directly to your local full nodes, builds real block templates, and broadcasts your blocks to the network when you find one. You are the pool.

---

## Table of Contents

- [Supported Coins](#supported-coins)
- [How It Works](#how-it-works)
- [Installation Guide](#installation-guide)
  - [Prerequisites](#prerequisites)
  - [Step 1: Download and Extract](#step-1-download-and-extract)
  - [Step 2: Install Tailscale (Optional - Remote Access)](#step-2-install-tailscale-optional---remote-access)
  - [Step 3: Install Wallet(s)](#step-3-install-wallets)
  - [Step 4: Blockchain Pruning](#step-4-blockchain-pruning)
  - [Step 5: Set Up Mining](#step-5-set-up-mining)
  - [Step 6: Start Mining](#step-6-start-mining)
- [Configure Your Miners](#configure-your-miners)
- [Email Notifications](#email-notifications)
  - [Setting Up SMTP During Installation](#setting-up-smtp-during-installation)
  - [Gmail App Password Setup](#gmail-app-password-setup)
  - [Per-Miner Email via Password Field](#per-miner-email-via-password-field)
- [Remote Access with Tailscale](#remote-access-with-tailscale)
  - [Installing Tailscale](#installing-tailscale)
  - [Accessing Your Monitor Remotely](#accessing-your-monitor-remotely)
  - [QR Code for Quick Phone Access](#qr-code-for-quick-phone-access)
- [BTC Fast Sync: UTXO Snapshots](#btc-fast-sync-utxo-snapshots)
  - [What Is a UTXO Snapshot?](#what-is-a-utxo-snapshot)
  - [Step-by-Step: Fast Sync BTC with UTXO Snapshot](#step-by-step-fast-sync-btc-with-utxo-snapshot)
  - [Verifying the Snapshot](#verifying-the-snapshot)
- [Blockchain Pruning Guide](#blockchain-pruning-guide)
  - [What Is Pruning?](#what-is-pruning)
  - [Pruning Sizes by Coin](#pruning-sizes-by-coin)
  - [Enabling Pruning](#enabling-pruning)
  - [Changing Pruning After Installation](#changing-pruning-after-installation)
- [NiceHash Integration](#nicehash-integration)
- [UPnP Auto Port Forwarding](#upnp-auto-port-forwarding)
- [Supported Miners](#supported-miners)
- [Block Found Notification](#block-found-notification)
- [Web Monitoring Dashboard](#web-monitoring-dashboard)
- [Stratum Protocol Support](#stratum-protocol-support)
- [Configuration Files](#configuration-files)
- [Uninstall](#uninstall)
- [Technical Details](#technical-details)
- [Support Development](#support-development)

---

## Supported Coins

| Coin | Ticker | Stratum Port | RPC Port | P2P Port | Node Software |
|------|--------|:------------:|:--------:|:--------:|---------------|
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

All 10 coins run concurrently on separate ports. Each coin has its own process, its own desktop shortcut, and talks to its own full node.

---

## How It Works

### Architecture

```
Your ASIC Miner(s)          NiceHash Rental
       |                          |
       |    stratum+tcp           |    stratum+tcp
       v                          v
+------------------------------------------+
|         HammerForge Stratum Server        |
|    (Node.js - one instance per coin)     |
|                                          |
|  - Builds block templates from your node |
|  - Assigns work to miners               |
|  - Validates shares                      |
|  - Submits blocks when found             |
|  - Variable difficulty (vardiff)         |
|  - UPnP auto port forwarding            |
+------------------------------------------+
       |
       |    JSON-RPC (localhost)
       v
+------------------------------------------+
|         Your Full Node Wallet            |
|    (Bitcoin Core, DigiByte Core, etc.)   |
|                                          |
|  - Fully synced blockchain               |
|  - getblocktemplate for real templates   |
|  - submitblock when you find a block     |
|  - Block reward goes to YOUR wallet      |
+------------------------------------------+
```

### The Mining Flow

1. **Block Template**: The stratum server calls `getblocktemplate` on your local full node to get a real block template containing pending transactions and the current block header data.

2. **Job Construction**: The server constructs a stratum mining job from the template, including the coinbase transaction (which pays the block reward to YOUR wallet address), the merkle branches, and the block header fields.

3. **Work Distribution**: Connected miners receive the job and begin hashing. Each miner gets a unique `extranonce1` so they never duplicate work.

4. **Share Submission**: When a miner finds a hash below the share difficulty target, it submits the share. The server validates the hash by reconstructing the full block header and performing a SHA-256d hash.

5. **Block Found**: If the share hash also meets the network difficulty target, the server immediately assembles the full block (header + coinbase + transactions) and submits it to your node via `submitblock`. A popup notification appears on your screen with the block details.

### Difficulty System

The stratum server runs a sophisticated variable difficulty (vardiff) system that automatically adjusts to any miner size:

| Setting | Value | Purpose |
|---------|-------|---------|
| Default Difficulty | 65,536 | Starting difficulty for home ASIC miners |
| NiceHash Min Difficulty | 500,000 | Minimum for NiceHash rentals (auto-detected) |
| Max Difficulty | 4,000,000,000 | Upper ceiling |
| Target Share Time | 5 seconds | How often a miner should submit a share |
| Retarget Interval | 15 seconds | How often difficulty is recalculated |
| Variance | 40% | Acceptable deviation from target share time |

**Password-based difficulty**: Miners can set a custom starting difficulty by putting `diff=500000` in the password field during stratum authorization. This is useful for pools like letsmine.it-style setups.

**NiceHash auto-detection**: When a miner connects with a user agent containing "NiceHash" or "nhmp", the server automatically sets the minimum difficulty to 500,000 to handle the high hashrate properly.

### Diff1 Target (pdiff)

The server uses the **pdiff** (pool difficulty) format for `DIFF1_TARGET`:

```
0x00000000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
```

This is critical for NiceHash compatibility. Many solo mining pools incorrectly use bdiff (`0x00000000FFFF0000...`), which underreports share difficulty by ~29x. NiceHash sees the mismatch and rejects the pool. Using pdiff ensures accurate difficulty reporting.

---

## Installation Guide

### Prerequisites

- **Operating System**: Windows 10 or 11 (64-bit)
- **Hardware**: At least one SHA-256 miner on your LAN (Bitaxe, MagicMiner, Luckyminer, Avalon Nano, or any SHA-256 ASIC)
- **Internet**: Required for downloading wallet software and syncing blockchains
- **Disk Space**: Depends on how many coins you install and whether you enable pruning (see [Blockchain Pruning Guide](#blockchain-pruning-guide))
- **RAM**: 8 GB minimum recommended if running multiple coin nodes simultaneously

### Step 1: Download and Extract

1. Download the HammerForge ZIP package
2. Right-click the ZIP file and select **Extract All...**
3. Extract to any folder (e.g., your Desktop or `C:\HammerForge`)
4. Open the extracted folder - you should see `START-HERE.html` and the installer batch files

> **Tip**: Open `START-HERE.html` in your browser for an interactive setup guide that walks you through every step.

### Step 2: Install Tailscale (Optional - Remote Access)

If you want to monitor your mining operation from your phone or another computer outside your home network, install Tailscale **before** running the mining installer. The installer will auto-detect Tailscale and configure remote access for you.

See the full [Remote Access with Tailscale](#remote-access-with-tailscale) section below for detailed instructions.

If you skip this step, everything still works - the monitor will just only be accessible on your home LAN. You can always install Tailscale later and re-run the mining installer.

### Step 3: Install Wallet(s)

For each coin you want to mine, right-click the corresponding installer and select **Run as administrator**:

| Installer | Coin | What It Installs |
|-----------|------|------------------|
| `INSTALL-BC2-CORE.bat` | BitcoinII | BitcoinII Core v29.1.0 |
| `INSTALL-DGB-CORE.bat` | DigiByte | DigiByte Core |
| `INSTALL-BTC-CORE.bat` | Bitcoin | Bitcoin Core v28.1 |
| `INSTALL-BCH-CORE.bat` | Bitcoin Cash | Bitcoin Cash Node |
| `INSTALL-BSV-CORE.bat` | Bitcoin SV | Bitcoin SV Node |
| `INSTALL-XEC-CORE.bat` | eCash | Bitcoin ABC |
| `INSTALL-FB-CORE.bat` | Fractal Bitcoin | Fractal Bitcoin |
| `INSTALL-PPC-CORE.bat` | Peercoin | Peercoin |
| `INSTALL-LCC-CORE.bat` | Litecoin Cash | Litecoin Cash |
| `INSTALL-NITO-CORE.bat` | NitoCoin | NitoCoin |

Each wallet installer automatically:

1. Downloads the wallet from the official GitHub release
2. Installs to `C:\Program Files\<CoinName>`
3. Creates the data directory in `%APPDATA%\<CoinName>`
4. **Asks if you want to enable blockchain pruning** (see [Blockchain Pruning Guide](#blockchain-pruning-guide))
5. Configures RPC credentials in the `.conf` file
6. Adds Windows Firewall rules for the P2P and RPC ports
7. Creates a desktop shortcut to launch the wallet

**Important**: After installing each wallet, open it and **let the blockchain sync completely** before mining. Sync time varies:

| Coin | Approximate Sync Time (Pruned) | Approximate Sync Time (Full) |
|------|-------------------------------|------------------------------|
| BC2 | Minutes to hours | Minutes to hours |
| DGB | Several hours | 1-2 days |
| BTC | Several hours | 2-7 days |
| BCH | Several hours | 1-3 days |
| BSV | Several hours | 1-3 days |
| XEC | Several hours | 1-3 days |
| FB | Minutes to hours | Minutes to hours |
| PPC | Minutes to hours | Minutes to hours |
| LCC | Minutes to hours | Minutes to hours |
| NITO | Minutes to hours | Minutes to hours |

### Step 4: Blockchain Pruning

During each wallet installation, you will be asked:

```
Would you like to enable blockchain pruning?
Pruning keeps only the last ~1 month of blocks
Mining works fine with pruning enabled.
Enable pruning? (y/n)
```

**Type `y`** to save disk space. **Type `n`** to keep the full blockchain. See the detailed [Blockchain Pruning Guide](#blockchain-pruning-guide) for more information.

### Step 5: Set Up Mining

After your wallets are installed and syncing, right-click `INSTALL-MINING.bat` and select **Run as administrator**.

The installer walks you through 7 steps:

| Step | What Happens |
|------|-------------|
| **1a/7** | Checks for and installs **Node.js v20** if not already present |
| **1b/7** | Checks for Tailscale (auto-detects if installed) |
| **2/7** | Detects which coin wallets are installed |
| **3/7** | Prompts for your wallet address for each detected coin (press Enter to skip) |
| **4/7** | Email notification setup (optional - see [Email Notifications](#email-notifications)) |
| **5/7** | Configures each wallet's `.conf` file with RPC server settings |
| **6/7** | Installs stratum server dependencies (`npm install`) |
| **7/7** | Creates config files, batch scripts, firewall rules, and desktop shortcuts |

After completion, you will have a desktop shortcut for each coin you configured (e.g., "BC2 HammerForge", "DGB HammerForge").

### Step 6: Start Mining

1. **Make sure your wallet is open and fully synced** for the coin you want to mine
2. **Double-click the desktop shortcut** (e.g., "BC2 HammerForge")
3. A CMD window opens showing the server log and the monitoring dashboard opens in your browser

The CMD window displays:
- The coin name and stratum port
- Your wallet address
- RPC connection status
- UPnP port forwarding status (your public IP for NiceHash)
- Tailscale remote access URL (if installed)
- Connected miners and their hashrates
- Share submissions in real-time

**You can run all 10 coins simultaneously.** Each runs as its own process on its own port.

**Port conflict recovery**: If an old instance is still running when you start a new one, the server automatically detects the conflict, kills the old process, and takes over the port. Your miners will briefly disconnect and then reconnect automatically.

---

## Configure Your Miners

Point your SHA-256 ASIC miners to:

```
URL:      stratum+tcp://YOUR_PC_IP:PORT
Worker:   YOUR_WALLET_ADDRESS.MinerName
Password: x
```

Where `PORT` is the stratum port for the coin you want to mine (see [Supported Coins](#supported-coins) table).

### Examples

**Mining BC2 with a Bitaxe:**
```
URL:      stratum+tcp://192.168.1.100:3333
Worker:   bc1qYourWalletAddress.Bitaxe1
Password: x
```

**Mining DGB with a MagicMiner:**
```
URL:      stratum+tcp://192.168.1.100:3334
Worker:   dgb1qYourWalletAddress.MagicMiner01
Password: x
```

**Mining BTC with NiceHash rental (using your public IP):**
```
URL:      stratum+tcp://YOUR_PUBLIC_IP:3335
Worker:   bc1qYourWalletAddress.NiceHash
Password: x
```

### Password Field Options

The miner's password field supports several options:

| Password | Effect |
|----------|--------|
| `x` | Standard mining, no extras |
| `diff=500000` | Set custom starting difficulty to 500,000 |
| `notif=you@gmail.com` | Enable email notifications for this miner |
| `diff=65536,notif=you@gmail.com` | Custom difficulty + email notifications |

---

## Email Notifications

Get notified by email when you find a block. HammerForge sends a detailed HTML email with the coin, block height, hash, reward, worker name, and timestamp.

### Setting Up SMTP During Installation

During Step 4/7 of `INSTALL-MINING.bat`, you will be asked:

```
Set up email notifications? (y/n)
```

If you choose **y**, the installer presents SMTP presets:

```
SMTP Presets:
  1. Gmail      (smtp.gmail.com)
  2. Outlook    (smtp.office365.com)
  3. Yahoo      (smtp.mail.yahoo.com)
  4. Custom SMTP server
Choose (1-4):
```

After selecting your provider, you enter:

| Prompt | What to Enter |
|--------|--------------|
| **Email address (login)** | Your full email address (e.g., `you@gmail.com`) |
| **App password** | Your app-specific password (NOT your regular email password) |
| **Send notifications to** | Where to receive alerts (press Enter to use the same address) |

The SMTP config is saved in each coin's JSON config file and applies globally. You can also override the notification email per-miner using the password field.

### Gmail App Password Setup

Gmail requires an **App Password** instead of your regular password. Here is how to set one up:

1. Go to [myaccount.google.com](https://myaccount.google.com)
2. Click **Security** in the left sidebar
3. Under "How you sign in to Google", make sure **2-Step Verification** is turned ON
4. Go back to Security, scroll down to **App passwords** (or search for it)
5. Click **App passwords**
6. Select **Mail** as the app and **Windows Computer** as the device
7. Click **Generate**
8. Google shows a 16-character password (e.g., `abcd efgh ijkl mnop`)
9. Copy this password and paste it into the installer when prompted for "App password"
10. You only need to do this once - the password is saved in your config

> **Note**: If you do not see "App passwords" in your Google account, you need to enable 2-Step Verification first.

### Outlook Setup

Use your regular Outlook/Hotmail email address and an app password:

1. Go to [account.microsoft.com/security](https://account.microsoft.com/security)
2. Click **Advanced security options**
3. Under "App passwords", click **Create a new app password**
4. Copy the generated password and use it during installation

### Yahoo Setup

1. Go to [login.yahoo.com/account/security](https://login.yahoo.com/account/security)
2. Click **Generate app password**
3. Select **Other App** and name it "HammerForge"
4. Copy the generated password and use it during installation

### Per-Miner Email via Password Field

You can also set the notification email directly in your miner's password field, which overrides the global config:

```
Password: notif=you@gmail.com
Password: diff=65536,notif=you@gmail.com
```

This is useful if you have multiple miners and want different people to receive block notifications for different miners. The SMTP server settings from the installer are still used for sending - this only controls the "to" address.

### Testing Email

After setting up, you can test the email from the monitoring dashboard API:

```
POST http://localhost:5000/api/test-email
```

Or check email status:

```
GET http://localhost:5000/api/email-status
```

---

## Remote Access with Tailscale

Tailscale lets you securely access your mining monitor from anywhere - your phone, your work PC, or any other device - without opening any ports to the internet. It creates a private encrypted network between your devices.

### Installing Tailscale

**Install Tailscale BEFORE running `INSTALL-MINING.bat`** so the installer auto-detects it:

1. **Download Tailscale** from [tailscale.com/download/windows](https://tailscale.com/download/windows)
2. **Run the installer** and follow the prompts
3. **Sign in** with your Google, Microsoft, or GitHub account (free for personal use)
4. **Wait for Tailscale to connect** - you will see the Tailscale icon in your system tray (bottom-right near the clock)
5. **Verify it is working** by opening a command prompt and running:
   ```
   tailscale ip -4
   ```
   You should see an IP address like `100.x.x.x` - this is your Tailscale IP

6. **Install Tailscale on your phone/other devices**:
   - **iPhone**: Download "Tailscale" from the App Store
   - **Android**: Download "Tailscale" from the Google Play Store
   - **Mac/Linux**: Download from [tailscale.com/download](https://tailscale.com/download)
   - Sign in with the **same account** on all devices

### What the Mining Installer Does with Tailscale

When you run `INSTALL-MINING.bat`, it automatically:

1. Checks if Tailscale is installed and connected
2. If found, displays your Tailscale IP (e.g., `100.64.0.2`)
3. Configures the batch files to detect and display the Tailscale URL when mining starts
4. The start scripts will show: `Remote access: http://100.64.0.2:5000/monitor`

If Tailscale is NOT installed, the installer shows:

```
Tailscale NOT installed.
For remote monitor access (from phone/work), install Tailscale:
  1. Download from https://tailscale.com/download/windows
  2. Install and sign in (free account)
  3. Install Tailscale on your phone too
  4. Access monitor via http://<tailscale-ip>:5000/monitor
Without Tailscale, you can still access the monitor on your home LAN.
```

### Accessing Your Monitor Remotely

Once Tailscale is set up on both your mining PC and your phone/other device:

1. **Start mining** any coin on your PC
2. **Note the Tailscale URL** shown in the CMD window (e.g., `http://100.64.0.2:5000/monitor`)
3. **On your phone/other device**, open a browser and go to that URL
4. You will see the full mining dashboard with live hashrate, workers, shares, and blocks

The connection is:
- **Encrypted** end-to-end (WireGuard protocol)
- **No ports opened** on your router (unlike port forwarding)
- **Works from anywhere** with internet access (coffee shop, work, vacation)
- **Free** for personal use (up to 100 devices)

### QR Code for Quick Phone Access

The monitoring dashboard includes a **QR Code** button in the remote access bar. Click it to generate a QR code of your Tailscale monitor URL. Scan it with your phone camera for instant access - no need to type the URL manually.

The QR code is generated locally in your browser and the URL never leaves your device.

### If You Installed Tailscale After Mining Setup

If you installed Tailscale after already running `INSTALL-MINING.bat`, you have two options:

1. **Re-run `INSTALL-MINING.bat`** - it will detect Tailscale and update your batch files
2. **Access manually** - just find your Tailscale IP (`tailscale ip -4` in command prompt) and go to `http://YOUR_TAILSCALE_IP:5000/monitor`

---

## BTC Fast Sync: UTXO Snapshots

Bitcoin's blockchain is over 600 GB and can take days to sync from scratch. Bitcoin Core v28+ includes a built-in feature called **AssumeUTXO** that lets you load a UTXO (Unspent Transaction Output) snapshot and start mining in under an hour instead of waiting days.

### What Is a UTXO Snapshot?

A UTXO snapshot is a verified checkpoint of every unspent bitcoin at a specific block height. Instead of replaying every transaction since 2009, Bitcoin Core loads this snapshot and immediately has a working view of who owns what. The node is then usable right away for mining and transactions, while it continues validating the full history in the background.

The snapshot for **block height 840,000** is hardcoded into Bitcoin Core v28+, meaning the software already knows the correct hash for this snapshot and will reject any tampered files.

**Key facts:**
- Your node is mining-ready in ~30 minutes instead of days
- Background validation continues automatically after loading
- Once background validation completes, your node is identical to a fully-synced node
- The snapshot hash is hardcoded in Bitcoin Core - tampered files are rejected
- Mining works immediately after loading the snapshot
- This is an official Bitcoin Core feature, not a hack or shortcut

> For a detailed technical explanation, see Jameson Lopp's guide: [Bitcoin Node Sync with UTXO Snapshots](https://blog.lopp.net/bitcoin-node-sync-with-utxo-snapshots/)

### Step-by-Step: Fast Sync BTC with UTXO Snapshot

#### 1. Install Bitcoin Core

Run `INSTALL-BTC-CORE.bat` as administrator. The installer downloads and installs Bitcoin Core v28+.

#### 2. Start Bitcoin Core

Double-click the "Bitcoin Core" desktop shortcut. Let it open and begin its initial sync. It will start downloading blocks - this is normal. You will interrupt it with the snapshot.

#### 3. Download the UTXO Snapshot File

Download the `utxo-840000.dat` file (~12 GB). Available from:

- **GitHub**: [github.com/nicehash/bitcoin-utxo-snapshot](https://github.com/nicehash/bitcoin-utxo-snapshot)
- **Torrent**: Search for `utxo-840000.dat` on popular torrent sites

Save the file somewhere convenient, like `C:\Users\YourName\Downloads\utxo-840000.dat`.

#### 4. Load the Snapshot

While Bitcoin Core is running, open a **Command Prompt** (not PowerShell) and run:

```cmd
"C:\Program Files\Bitcoin\bin\bitcoin-cli.exe" -rpcclienttimeout=0 loadtxoutset "C:\Users\YourName\Downloads\utxo-840000.dat"
```

> **Important**: Use `-rpcclienttimeout=0` because loading takes 10-30 minutes and the default timeout would kill the command prematurely.

> **Note**: Adjust the paths if your Bitcoin Core is installed elsewhere or if you saved the snapshot to a different location.

You will see output like:
```
{
  "coins_loaded": 176000000,
  "tip_hash": "0000000000000000000320283a032748cef8227873ff4872689bf23f1cda83a5",
  "base_height": 840000,
  "path": "C:\\Users\\YourName\\Downloads\\utxo-840000.dat"
}
```

#### 5. Wait for Remaining Sync

After loading the snapshot, Bitcoin Core only needs to sync blocks from height 840,000 to the current tip. This takes minutes to a few hours depending on how far ahead the chain is.

You can check sync progress in the Bitcoin Core GUI - it will show "Syncing Headers" and then "Synchronizing with network..." with a progress bar.

#### 6. Start Mining

Once the node is synced to the current tip, run `INSTALL-MINING.bat` to set up your stratum server and start mining BTC.

### Verifying the Snapshot

Bitcoin Core automatically verifies the snapshot against the hardcoded hash for block 840,000. If the file is corrupted or tampered with, Bitcoin Core will reject it with an error message. You do not need to manually verify anything.

The expected hash for the height 840,000 snapshot is built into the Bitcoin Core source code. This is the same trust model as the rest of Bitcoin Core's `assumevalid` checkpoints that have been used since 2017.

### Using Snapshot with Pruning

UTXO snapshots work with pruning enabled. If you chose pruning during installation, the snapshot will load normally and the node will only keep the most recent blocks after background validation. This is the most disk-space-efficient way to run a BTC node for mining:

- **Snapshot + Pruning**: ~10 GB total (fastest setup, smallest footprint)
- **Snapshot + Full Chain**: Starts mining fast, stores full chain as background sync completes
- **No Snapshot + Pruning**: ~10 GB total but takes days to sync
- **No Snapshot + Full Chain**: 600+ GB, takes days to sync

---

## Blockchain Pruning Guide

### What Is Pruning?

Blockchain pruning tells the wallet software to only keep the most recent block data (approximately the last month of blocks) instead of the entire blockchain history. Older blocks are deleted from disk after they have been validated.

**Key facts about pruning:**

- Mining works perfectly fine with pruning enabled
- Your wallet balance and transactions are NOT affected by pruning
- You can still receive and send coins normally
- The only thing you lose is the ability to look up very old block data locally
- You save hundreds of gigabytes of disk space (especially for BTC, BCH, BSV, XEC)

### Pruning Sizes by Coin

| Coin | Prune Value | Pruned Size | Full Chain Size | Space Saved |
|------|:-----------:|:-----------:|:---------------:|:-----------:|
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

> **Recommendation**: Enable pruning for BTC, BCH, BSV, and XEC unless you have a large drive. These chains are hundreds of gigabytes each.

### Enabling Pruning

Pruning is offered during each wallet installation. When the installer asks:

```
Would you like to enable blockchain pruning?
Pruning keeps only the last ~1 month of blocks (~10 GB)
Mining works fine with pruning enabled.
Enable pruning? (y/n)
```

- Type **`y`** and press Enter to enable pruning
- Type **`n`** and press Enter to keep the full blockchain

### Changing Pruning After Installation

You can enable or disable pruning at any time by editing the coin's config file:

1. **Close the wallet** for that coin if it is running
2. **Find the config file**:

| Coin | Config File Location |
|------|---------------------|
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

3. **Open the file** in Notepad
4. **To enable pruning**, add or change the line:
   ```
   prune=10000
   ```
   (Use the prune value from the table above for your coin)

5. **To disable pruning**, change the line to:
   ```
   prune=0
   ```

6. **Save the file** and restart the wallet

> **Note**: If you disable pruning on a wallet that was previously pruned, it will need to re-download the full blockchain from scratch. This can take several days for larger chains like BTC.

---

## NiceHash Integration

HammerForge is fully compatible with NiceHash SHA-256 hash rentals. Rent hashpower on the NiceHash marketplace and point it at your stratum server to dramatically increase your chances of finding a block.

### How to Set Up NiceHash

1. **Start mining any coin** - The stratum server automatically opens the port on your router via UPnP
2. **Note your public IP** - It is shown in the CMD window: `UPnP: External/public IP is X.X.X.X`
3. **Create a new order on NiceHash**:
   - Algorithm: **SHA256**
   - Pool host: `stratum+tcp://YOUR_PUBLIC_IP:PORT`
   - Worker: `YOUR_WALLET_ADDRESS.NiceHash`
   - Password: `x`

### NiceHash-Specific Features

- **Auto-detection**: Miners with "NiceHash" or "nhmp" in their user agent are automatically given a minimum difficulty of 500,000
- **Version rolling**: Full `mining.configure` support with `version-rolling` extension for AsicBoost
- **Extranonce subscribe**: Supports `mining.extranonce.subscribe` for NiceHash compatibility
- **Rapid vardiff**: Difficulty adjusts within 15 seconds to handle sudden hashrate changes from rental starts/stops

---

## UPnP Auto Port Forwarding

When the stratum server starts, it automatically maps the stratum port on your router using UPnP (Universal Plug and Play). This means:

- **No manual port forwarding** needed in your router settings
- NiceHash and external miners can connect to your server immediately
- The mapping refreshes every hour and has a 2-hour TTL
- When the server stops, the port mapping is automatically cleaned up
- If UPnP is unavailable on your router, the server continues normally with a warning

---

## Supported Miners

| Miner | Type | Notes |
|-------|------|-------|
| MagicMiner | USB ASIC | All models supported |
| Luckyminer LV06 | USB ASIC | SHA-256 mode |
| Avalon Nano 3S | USB ASIC | Low-power home miner |
| Bitaxe | Open-source ASIC | All variants |
| NiceHash SHA-256 | Hashrate rental | Auto-detected, optimized difficulty |
| Any SHA-256 stratum miner | ASIC/Software | Standard stratum v1 protocol |

---

## Block Found Notification

When you find a block, three things happen:

1. **Popup window** appears on your Windows desktop with full block details:

```
===================================
  BLOCK FOUND!
===================================
  Coin:    BC2 (BitcoinII)
  Height:  123456
  Hash:    0000000000000abc...
  Reward:  50.00000000 BC2
  Status:  Submitted to network
  Time:    2025-01-15 14:30:22
===================================
```

2. **Email notification** is sent (if configured) with the same details in a formatted HTML email

3. **Log file entry** is appended to `logs/blocks-found.log` (permanent record of all blocks found)

Blocks are submitted to your local node immediately via `submitblock`. The block reward is paid directly to the wallet address you configured during setup.

---

## Web Monitoring Dashboard

When you start mining any coin, the monitoring dashboard automatically opens in your browser at `http://localhost:5000/monitor`. It provides a live overview of your mining operation:

- **Hashrate chart** - Real-time graph with 30-second interval snapshots (up to 24 hours of history)
- **Best share chart** - Logarithmic bar chart of highest difficulty shares over time (5-minute intervals)
- **Worker table** - All connected miners with 1-minute, 5-minute, and 1-hour hashrate averages
- **Recent shares** - Table of the latest share submissions with difficulty and status
- **Blocks found** - Table of all discovered blocks with height, hash, reward, and acceptance status
- **Network stats** - Current block height, network difficulty, network hashrate, and coin price
- **Best share** - Highest difficulty share submitted in the current session
- **Uptime** - How long the stratum server has been running
- **Donate dropdown** - Support the project with 8 crypto donation addresses (click to copy)

The page auto-refreshes every 5 seconds. A demo mode is available at `/monitor?demo=true` with realistic sample data.

### Auto Browser-Open

On Windows, the monitoring page opens automatically in your default browser as soon as the stratum server finishes starting. No need to manually navigate to the URL.

---

## Stratum Protocol Support

The server implements the stratum v1 protocol with the following methods:

| Method | Support | Notes |
|--------|---------|-------|
| `mining.subscribe` | Full | Returns extranonce1 and extranonce2 size |
| `mining.authorize` | Full | Accepts any wallet.worker format |
| `mining.submit` | Full | SHA-256d validation with block submission |
| `mining.notify` | Full | Real block templates from your node |
| `mining.set_difficulty` | Full | Vardiff with per-miner adjustment |
| `mining.configure` | Full | Version-rolling (AsicBoost) support |
| `mining.suggest_difficulty` | Full | Miners can request starting difficulty |
| `mining.extranonce.subscribe` | Full | NiceHash compatibility |

---

## Configuration Files

Each coin's configuration is stored as a JSON file in the HammerForge directory:

```json
{
  "walletAddress": "bc1qYourAddress...",
  "stratumPort": 3333,
  "rpcHost": "127.0.0.1",
  "rpcPort": 8232,
  "rpcUser": "bc2rpc",
  "rpcPassword": "auto-generated-password",
  "coin": "BC2",
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

These are created automatically by `INSTALL-MINING.bat`. You can edit them manually if needed.

| Config File | Coin |
|------------|------|
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

---

## Uninstall

Right-click `UNINSTALL.bat` and select **Run as administrator** to remove:
- The stratum mining server files
- Desktop shortcuts
- Firewall rules

Your blockchain data and wallets are **never** touched by the uninstaller.

---

## Technical Details

### Coinbase Transaction

The coinbase transaction is constructed with:
- Block height encoded in script (BIP34)
- Coinbase tag identifying the solo miner (e.g., `/BC2Solo/`, `/DGBSolo/`)
- Output paying the full block reward to your configured wallet address
- SegWit commitment (where applicable)

### Share Validation

When a miner submits a share:
1. The server reconstructs the coinbase transaction using `coinb1 + extranonce1 + extranonce2 + coinb2`
2. Builds the merkle root by hashing the coinbase with the merkle branches
3. Constructs the 80-byte block header
4. Performs SHA-256d (double SHA-256) on the header
5. Compares the resulting hash against the share difficulty target
6. If the hash also meets the network target, submits the full block

### Fallback Templates

If a coin's node is not running or not fully synced, the server generates fallback templates so miners can stay connected and keep hashing. However, blocks cannot be found or submitted with fallback templates. The CMD window clearly warns you:

```
*** WARNING: BC2 node STILL not available (RPC port 8232).
    Miners are hashing on FALLBACK templates -- blocks CANNOT be found!
    Sync your node! ***
```

Once your node comes online and syncs, the server automatically switches to real templates and starts finding blocks.

---

## File Structure

```
HammerForge/
  START-HERE.html           - Interactive setup guide (open this first)
  INSTALL-BC2-CORE.bat      - BitcoinII Core wallet installer
  INSTALL-DGB-CORE.bat      - DigiByte Core wallet installer
  INSTALL-BTC-CORE.bat      - Bitcoin Core wallet installer
  INSTALL-BCH-CORE.bat      - Bitcoin Cash Node installer
  INSTALL-BSV-CORE.bat      - Bitcoin SV Node installer
  INSTALL-XEC-CORE.bat      - Bitcoin ABC (eCash) installer
  INSTALL-FB-CORE.bat       - Fractal Bitcoin installer
  INSTALL-PPC-CORE.bat      - Peercoin installer
  INSTALL-LCC-CORE.bat      - Litecoin Cash installer
  INSTALL-NITO-CORE.bat     - NitoCoin installer
  INSTALL-MINING.bat        - Mining setup (Node.js + stratum server)
  UNINSTALL.bat             - Clean uninstaller
  OPEN-MONITOR.bat          - Opens mining monitor in browser
  README.txt                - Quick reference guide
  server/
    stratum.ts              - Stratum protocol implementation
    rpc.ts                  - JSON-RPC client for full nodes
    routes.ts               - Web dashboard API routes
    storage.ts              - In-memory state management
    email.ts                - Email notifications (block found alerts)
    monitor.html            - Mining dashboard (served directly on Windows)
    index.ts                - Server entry point
  shared/
    schema.ts               - TypeScript type definitions
  client/
    src/
      pages/
        dashboard.tsx       - Download page UI
        monitor.tsx         - Mining monitoring dashboard (React version)
```

---

## Support Development

If you find this project useful, consider donating:

| Coin | Address |
|------|---------|
| BCH | `qrqtvrm6vvvzq6w8jcu9alcv599lmsgpfv7a87edaa` |
| BTC | `bc1qczmq77nrrqsxn3uk8p007r9pf07uzq2t96w3g5` |
| DGB | `DNi4NsZKH93QxBWn1RfQYizGtrKVhJcz6q` |
| ETH | `0x996c85e9137a9020C2a3d9b4889aA6a35185d368` |
| LTC | `Lcnv6pvW4PPBfz2LSyf9GytSDvxUWwaiSB` |
| USDC (ERC20) | `0x996c85e9137a9020C2a3d9b4889aA6a35185d368` |
| XEC | `ecash:qrqtvrm6vvvzq6w8jcu9alcv599lmsgpfv8sn4zh` |
| XRP | `rJZxqccCyj93RBLBGqCqzxFgr5bURGJrrm` |

---

## Important: Legal and Security Note

- **No Warranty**: This software is provided "as is" without warranty of any kind. The author is not liable for any damages, lost funds, lost private keys, or any other losses arising from the use of this software.
- **Private Keys**: You are solely responsible for securing your wallet private keys and RPC credentials. Never share your wallet.dat or RPC passwords.
- **Use at Your Own Risk**: Solo mining involves financial risk. Block rewards are not guaranteed. The author makes no guarantees about mining profitability or software reliability.

---

## License

Copyright (c) 2025 James Vernon Persons. All Rights Reserved.

This software and all associated files, scripts, and documentation are the proprietary property of James Vernon Persons. No part of this software may be reproduced, distributed, transmitted, modified, reverse-engineered, or used in any form without the express written permission of the copyright holder.

Unauthorized copying, distribution, or modification of this software is strictly prohibited and may result in legal action.
