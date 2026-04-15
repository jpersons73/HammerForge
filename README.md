================================================================================
                              H A M M E R F O R G E
                          Your Home Node Mining Solution
================================================================================

                     SHA-256 Solo Mining  |  12 Coins  |  Windows
                  Fully Automated  |  Zero Fees  |  100% YOUR Rewards

     GitHub: https://github.com/jpersons73/HammerForge

================================================================================
  TABLE OF CONTENTS
================================================================================

  1. Overview
  2. Supported Coins
  3. System Requirements
  4. Quick Start Guide
  5. Architecture
  6. Features
  7. Installation Details
  8. Configuration Reference
  9. Web Dashboard & Monitoring
  10. Miner Setup (Bitaxe / NiceHash / ASIC)
  11. Network & Remote Access
  12. Block Submission Pipeline
  13. Troubleshooting
  14. FAQ
  15. License & Credits

================================================================================
  1. OVERVIEW
================================================================================

HammerForge is a fully automated, zero-fee SHA-256 solo mining solution for
Windows. It turns any Windows PC into a private mining pool capable of running
up to 12 cryptocurrency nodes simultaneously, each with its own Stratum server.

Unlike pool mining, HammerForge gives you 100% of every block reward -- no pool
fees, no payout thresholds, no third parties. You run the full node, you own the
keys, you get the coins.

Designed for:
  - Home ASIC miners (Bitaxe, MagicMiner, Avalon Nano, etc.)
  - NiceHash hashrate rentals
  - Small/medium SHA-256 mining operations
  - Cryptocurrency enthusiasts who want to support decentralization

================================================================================
  2. SUPPORTED COINS
================================================================================

HammerForge supports 12 SHA-256 mineable coins, each running on dedicated ports:

  +--------+------------------+---------------+-----------+
  | Ticker | Coin             | Stratum Port  | RPC Port  |
  +--------+------------------+---------------+-----------+
  | BC2    | BitcoinII        | 3333          | 8232      |
  | DGB    | DigiByte (SHA)   | 3334          | 14022     |
  | BTC    | Bitcoin          | 3335          | 8332      |
  | BCH    | Bitcoin Cash     | 3336          | 8432      |
  | BSV    | Bitcoin SV       | 3337          | 8532      |
  | XEC    | eCash            | 3338          | 8632      |
  | FB     | Fractal Bitcoin  | 3339          | 8732      |
  | PPC    | Peercoin         | 3340          | 9902      |
  | LCC    | Litecoin Cash    | 3341          | 62457     |
  | NITO   | NitoCoin         | 3342          | 8821      |
  | BCH2   | Bitcoin Cash II  | 3343          | 8342      |
  | XRO    | XeroCoin         | 3344          | 25169     |
  +--------+------------------+---------------+-----------+

Each coin runs as a completely independent process with its own node, wallet,
stratum server, and web dashboard. You can run any combination of coins
simultaneously.

================================================================================
  3. SYSTEM REQUIREMENTS
================================================================================

  Minimum:
    - Windows 10 or Windows 11 (64-bit)
    - 8 GB RAM (16 GB+ recommended for multiple coins)
    - 50 GB free disk space (with pruning enabled)
    - Internet connection
    - Node.js v20+ (auto-installed by the mining installer)

  Recommended:
    - 32 GB RAM (for running all 12 coins)
    - SSD storage (faster blockchain sync)
    - Stable, always-on internet connection
    - UPnP-capable router (for automatic port forwarding)

  Disk Space (approximate, with pruning):
    - BTC:  ~10 GB (pruned from 600+ GB)
    - BCH:  ~5 GB
    - BSV:  ~5 GB
    - Other coins: 1-5 GB each

  Note: Without pruning, a full BTC node requires 600+ GB. HammerForge enables
  pruning by default during installation to keep disk usage manageable.

================================================================================
  4. QUICK START GUIDE
================================================================================

  Step 1: Download & Extract
  --------------------------
  Download the latest HammerForge release from GitHub and extract the ZIP file
  to any folder (e.g., C:\HammerForge).

  Step 2: Install Coin Nodes
  --------------------------
  Navigate to the extracted folder and run the installer for each coin you want
  to mine. For example:

    > Double-click: INSTALL-BTC-CORE.bat
    > Double-click: INSTALL-BCH2-CORE.bat

  Each installer will:
    - Download the official coin node software
    - Configure the node for solo mining (RPC, pruning, firewall)
    - Create a wallet and generate a receiving address
    - Open the wallet to begin blockchain synchronization

  Step 3: Wait for Sync
  ---------------------
  Let each coin node fully synchronize with the blockchain. This can take
  anywhere from a few minutes (small coins) to several hours (BTC).

  You can check sync progress in each wallet's status bar.

  Step 4: Install Mining Software
  -------------------------------
  Once nodes are synced, run:

    > Double-click: Mining\INSTALL-MINING.bat

  This installer will:
    - Install Node.js v20 (if not already installed)
    - Prompt you for wallet addresses for each installed coin
    - Optionally configure email notifications (SMTP)
    - Set up RPC credentials
    - Create desktop shortcuts for each coin's mining server

  Step 5: Start Mining
  --------------------
  Double-click any desktop shortcut (e.g., "BTC HammerForge") to start that
  coin's stratum server. Then point your miner to:

    stratum+tcp://YOUR_PC_IP:PORT

  Where PORT is the coin's stratum port (see table in Section 2).

================================================================================
  5. ARCHITECTURE
================================================================================

  HammerForge uses a three-layer architecture:

  +-------------------+     +------------------+     +------------------+
  |   ASIC Miner      |     |   HammerForge    |     |   Coin Node      |
  |   (Bitaxe, etc.)  |<--->|   Stratum Server |<--->|   (Full Node)    |
  |                   |     |   + Web Dashboard|     |   + Wallet       |
  +-------------------+     +------------------+     +------------------+
        Stratum V1              Node.js/TS              JSON-RPC

  Layer 1: Coin Full Nodes
  - Official coin software running locally
  - Fully validates all transactions and blocks
  - Provides block templates via getblocktemplate (GBT)
  - Manages wallet and receives block rewards

  Layer 2: HammerForge Stratum Server
  - Implements Stratum V1 mining protocol
  - Receives block templates from the coin node
  - Distributes mining jobs to connected miners
  - Validates submitted shares and checks for blocks
  - Submits found blocks to the coin node
  - Hosts the web monitoring dashboard

  Layer 3: Mining Hardware
  - Any SHA-256 ASIC miner (Bitaxe, MagicMiner, etc.)
  - NiceHash hashrate marketplace
  - Connects via standard Stratum V1 protocol

  Key Server Components:
  - stratum.ts   -- Stratum V1 protocol implementation
  - rpc.ts       -- JSON-RPC communication with coin nodes
  - routes.ts    -- REST API + web dashboard serving
  - storage.ts   -- In-memory state + persistent block log
  - email.ts     -- SMTP notification system

================================================================================
  6. FEATURES
================================================================================

  MINING ENGINE
  -------------
  - Full Stratum V1 protocol implementation
  - SHA-256 double-hash share validation
  - Fast-path block submission (submit first, verify after)
  - Variable difficulty (Vardiff) targeting 15-second share intervals
  - NiceHash auto-detection with 500,000 minimum difficulty
  - AsicBoost / version-rolling support (mining.configure)
  - Concurrent mining of all 12 coins on separate ports
  - Automatic block template polling and job distribution
  - Near-miss share logging (tracks close calls)

  BLOCK SUBMISSION
  ----------------
  - Zero-delay fast-path: block hex is built and submitted immediately
  - Submit latency timing (build time + RPC time in milliseconds)
  - Post-submit verification (confirms block on chain)
  - Orphan detection at 30s and 120s after submission
  - Automatic status updates (ACCEPTED / ORPHANED / STALE)
  - Full block hex logging for forensic analysis

  WALLET ADDRESS SUPPORT
  ----------------------
  - Bech32:    bc1q, dgb1q, nito1q, lcc1q, xro1q, pc1q, fb1q, firo1q
  - Bech32m:   bc1p, dgb1p, nito1p, lcc1p, xro1p, pc1p, fb1p, firo1p
  - Base58:    1..., 3..., D..., N..., P..., C..., a..., L..., F..., E...
  - CashAddr:  bitcoincash:q, bitcoincashii:q, ecash:q
  - RPC fallback: automatic resolution via validateaddress for unknown formats
  - Startup verification: confirms address decodes correctly before mining

  WEB DASHBOARD
  -------------
  - Real-time hashrate monitoring with interactive charts
  - Per-worker statistics (hashrate, shares, difficulty, uptime)
  - Network difficulty and block height tracking
  - External difficulty comparison (detects wrong-fork scenarios)
  - Blocks found log with status tracking
  - Live stratum connection count and activity
  - Responsive design -- works on desktop, tablet, and mobile

  NOTIFICATIONS
  -------------
  - Windows desktop popup when a block is found
  - Visual block hash display with status indicators
  - SMTP email notifications (configurable per-worker)
  - Block status updates (ACCEPTED, ORPHANED, REJECTED)

  NODE MANAGEMENT
  ---------------
  - One-click installers for all 12 coin nodes
  - Automatic blockchain pruning (saves hundreds of GB)
  - Firewall rule configuration
  - RPC credential management
  - Graceful node shutdown support
  - Seed node configuration for smaller coins

  NETWORK & REMOTE ACCESS
  -----------------------
  - UPnP automatic port forwarding
  - Tailscale integration for secure remote monitoring
  - Tunnel URL display in the web dashboard
  - External miner support (mine from anywhere on your network)

================================================================================
  7. INSTALLATION DETAILS
================================================================================

  COIN NODE INSTALLERS
  --------------------
  Each coin has its own installer batch file (e.g., INSTALL-BTC-CORE.bat) that
  performs the following:

    1. Downloads the official coin node binary from the project's website
    2. Extracts and installs to C:\HammerForge\[COIN]\
    3. Creates a coin-specific .conf file with:
       - RPC server enabled (server=1)
       - RPC credentials (unique per coin)
       - Pruning enabled (to save disk space)
       - Listen mode enabled (for network participation)
       - Appropriate seed nodes (for smaller coins)
    4. Configures Windows Firewall rules for the coin's ports
    5. Creates a wallet and generates a receiving address
    6. Launches the node to begin blockchain synchronization

  MINING INSTALLER
  ----------------
  The mining installer (Mining\INSTALL-MINING.bat) performs:

    1. Checks for and installs Node.js v20 (if needed)
    2. Runs npm install to set up the stratum server
    3. Detects which coin nodes are installed
    4. Prompts for wallet addresses for each detected coin
    5. Optionally configures SMTP email settings
    6. Generates config-[coin].json files for each coin
    7. Creates desktop shortcuts for quick access
    8. Sets up the web dashboard

  UNINSTALLER
  -----------
  Individual coin uninstallers and a master uninstaller are provided:

    > Uninstall\UNINSTALL.bat        -- Remove everything
    > [COIN]\UNINSTALL-[COIN]-CORE.bat -- Remove a specific coin

================================================================================
  8. CONFIGURATION REFERENCE
================================================================================

  Each coin's configuration is stored in config-[coin].json:

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

  MINER PASSWORD OPTIONS
  ----------------------
  When connecting your miner, the password field supports special options:

    diff=X          Set a custom starting difficulty
                    Example: diff=50000

    notif=EMAIL     Override the notification email for this worker
                    Example: notif=miner@example.com

  These can be combined with a comma:
    diff=100000,notif=miner@example.com

  VARDIFF SETTINGS
  ----------------
  Variable difficulty automatically adjusts to maintain optimal share rates:

    Target share time:    15 seconds
    Retarget interval:    15 seconds
    Variance:             40%
    Min difficulty:       1
    Max difficulty:       4,000,000,000,000

  For NiceHash connections:
    Minimum difficulty:   500,000 (auto-detected)

================================================================================
  9. WEB DASHBOARD & MONITORING
================================================================================

  Each coin's stratum server hosts a web dashboard on port 5000 (or the next
  available port). Access it at:

    http://YOUR_PC_IP:5000

  Dashboard Sections:

    HASHRATE CHART
    - Real-time hashrate graph updated every few seconds
    - Shows historical hashrate over time
    - Displays current, average, and peak hashrate

    WORKER TABLE
    - Lists all connected miners with:
      - Worker name and IP address
      - Current hashrate
      - Share count (accepted / rejected)
      - Current difficulty
      - Connection uptime
      - User agent (miner software info)

    NETWORK INFO
    - Current block height
    - Network difficulty
    - External difficulty comparison (from hashrate.no)
    - Fork detection warnings
    - Node sync status

    BLOCKS FOUND
    - Complete log of all discovered blocks
    - Block hash, height, reward, and status
    - Worker who found the block
    - Timestamp and difficulty
    - Status tracking: ACCEPTED / ORPHANED / STALE / REJECTED
    - "Clear All" button for resetting the log

    STRATUM ACTIVITY
    - Live connection count
    - Recent stratum protocol messages
    - Share submission rate

================================================================================
  10. MINER SETUP (BITAXE / NICEHASH / ASIC)
================================================================================

  BITAXE SETUP
  -------------
  1. Open your Bitaxe web interface (usually http://192.168.x.x)
  2. Go to Settings
  3. Configure:
     - Pool URL:     YOUR_PC_IP
     - Pool Port:    3335 (for BTC, or the appropriate port)
     - Pool User:    YOUR_WALLET_ADDRESS.worker1
     - Pool Pass:    x (or diff=X for custom difficulty)
  4. Save and restart

  NICEHASH SETUP
  --------------
  1. Log into NiceHash
  2. Create a new order for SHA-256 algorithm
  3. Set the stratum URL:
     - stratum+tcp://YOUR_PC_IP:3335
  4. HammerForge auto-detects NiceHash connections and sets appropriate
     minimum difficulty (500,000)

  GENERIC ASIC SETUP
  ------------------
  Configure your ASIC miner with:
    - URL:      stratum+tcp://YOUR_PC_IP:PORT
    - User:     WALLET_ADDRESS.WORKER_NAME
    - Pass:     x

  MULTIPLE WORKERS
  ----------------
  You can connect multiple miners to the same coin. Use different worker names
  to distinguish them in the dashboard:
    - worker1.bitaxe1
    - worker1.bitaxe2
    - worker1.nicehash

================================================================================
  11. NETWORK & REMOTE ACCESS
================================================================================

  LOCAL NETWORK
  -------------
  By default, HammerForge listens on all network interfaces (0.0.0.0). Any
  device on your local network can connect to the stratum server or view the
  web dashboard.

  UPNP (AUTOMATIC PORT FORWARDING)
  ---------------------------------
  HammerForge automatically attempts to configure your router via UPnP to
  forward the stratum port. This allows external miners (or NiceHash) to
  connect directly to your server.

  If UPnP is not available, you will see a message:
    "UPnP: Could not map port XXXX -- Manual port forwarding may be needed."

  In this case, manually forward the appropriate port in your router settings.

  TAILSCALE (SECURE REMOTE ACCESS)
  --------------------------------
  HammerForge integrates with Tailscale for secure remote monitoring. If
  Tailscale is installed, the dashboard will display your Tailscale IP for
  remote access without exposing ports to the public internet.

  FIREWALL
  --------
  The coin node installers automatically create Windows Firewall rules for:
    - The coin's P2P port (for blockchain network participation)
    - The coin's RPC port (for local communication only)
    - The stratum port (for miner connections)

================================================================================
  12. BLOCK SUBMISSION PIPELINE
================================================================================

  When a miner submits a share that meets the network difficulty target,
  HammerForge uses a fast-path submission pipeline:

    1. DETECT
       - Share hash is compared against network target
       - If hash <= target, it qualifies as a valid block

    2. BUILD (sub-millisecond)
       - Block header hex is already computed from the share
       - Coinbase transaction is assembled with wallet address
       - Transactions are ordered (CTOR for BCH/XEC/BCH2)
       - Full block hex is constructed in memory

    3. SUBMIT (immediate)
       - submitblock RPC call sent to the coin node immediately
       - No pre-submission delays or template re-fetching
       - Submit latency is timed and logged

    4. VERIFY (async, after submission)
       - Post-submit verification confirms block on chain
       - Chain diagnostics: peer count, sync status, best tip
       - Orphan checks scheduled at 30s and 120s

    5. NOTIFY
       - Desktop popup with block details
       - Email notification (if configured)
       - Block recorded in persistent log
       - Dashboard updated in real-time

  This pipeline is optimized for speed. Previous versions performed template
  verification and extensive logging BEFORE submission, adding 100-500ms of
  latency. The current fast-path design submits first and verifies after,
  minimizing the window for competing blocks to be accepted.

  COIN-SPECIFIC HANDLING
  ----------------------
  - BCH / XEC / BCH2:  CTOR transaction ordering, no SegWit
  - BSV:               No SegWit
  - BTC / BC2 / DGB:   SegWit support with witness commitment
  - PPC:               Block signature field appended
  - DGB:               SHA-256 algo version bits (0x200) in block header

================================================================================
  13. TROUBLESHOOTING
================================================================================

  PROBLEM: "Cannot reach [COIN] node" / RPC connection failed
  -----------------------------------------------------------
  - Make sure the coin node (wallet) is running and fully synced
  - Check that the RPC port is not blocked by a firewall
  - Verify RPC credentials in config-[coin].json match the coin's .conf file
  - Try restarting the coin node

  PROBLEM: Miner connects but shows 0 hashrate
  ---------------------------------------------
  - Wait 30-60 seconds for hashrate calculation to stabilize
  - Check that the miner is receiving jobs (look for mining.notify in logs)
  - Verify the miner supports SHA-256 algorithm

  PROBLEM: Block found but reward not in wallet
  ----------------------------------------------
  - Check the block status in the dashboard (ACCEPTED vs ORPHANED)
  - Coinbase rewards require ~100 confirmations to mature
  - Verify the wallet address was correctly decoded at startup
    (look for "Wallet address verified" in the console log)
  - If the block was ORPHANED, another miner's block won the race

  PROBLEM: "Could not decode wallet address"
  ------------------------------------------
  - The wallet address format is not recognized by the local decoder
  - Ensure the coin node is running (HammerForge will try RPC fallback)
  - Check the address format matches the coin (e.g., bc1q for BTC)
  - Look for "CRITICAL" messages in the startup log

  PROBLEM: Blocks found but always orphaned
  ------------------------------------------
  - Check peer count: if 0 outbound peers, blocks cannot propagate
  - Ensure the coin node is fully synced (not in Initial Block Download)
  - Verify your node has good network connectivity
  - Add more seed nodes to the coin's .conf file
  - Check submit latency in logs (should be under 100ms)

  PROBLEM: UPnP port mapping failed
  ----------------------------------
  - Your router may not support UPnP, or it may be disabled
  - Manually forward the stratum port in your router settings
  - Check that another application is not using the same port

  PROBLEM: NiceHash miners keep disconnecting
  --------------------------------------------
  - NiceHash requires minimum difficulty of 500,000 (auto-configured)
  - Ensure your internet connection is stable
  - Check that the stratum port is accessible from the internet

================================================================================
  14. FAQ
================================================================================

  Q: How much hashrate do I need to find a block?
  A: It depends on the coin's network difficulty. For small coins like BC2,
     NITO, or BCH2, even a single Bitaxe (~500 GH/s) can find blocks regularly.
     For BTC, you would need significantly more hashrate or extraordinary luck.

  Q: Is there a pool fee?
  A: No. HammerForge is a solo mining solution. You get 100% of every block
     reward directly to your own wallet. There are no fees of any kind.

  Q: Can I mine multiple coins at the same time?
  A: Yes. Each coin runs as a separate process on its own port. You can run
     all 12 coins simultaneously if your PC has enough resources.

  Q: Does HammerForge work with NiceHash?
  A: Yes. HammerForge auto-detects NiceHash connections and configures
     appropriate settings (minimum difficulty, AsicBoost support).

  Q: Do I need to keep the coin wallet open?
  A: Yes. The coin node (wallet) must be running and fully synced for mining
     to work. HammerForge communicates with it via JSON-RPC.

  Q: What happens if my node goes down while mining?
  A: HammerForge will detect the RPC failure and display a warning. Mining
     will pause (using fallback templates that cannot find real blocks) until
     the node is back online. No blocks can be lost this way.

  Q: Can I access the dashboard remotely?
  A: Yes, via Tailscale (recommended for security) or by port-forwarding
     the web dashboard port (5000) through your router.

  Q: What is block pruning?
  A: Pruning reduces disk usage by only keeping the most recent blocks.
     A pruned BTC node uses ~10 GB instead of 600+ GB. Mining works
     perfectly with pruned nodes.

  Q: Why was my block orphaned?
  A: An orphaned block means another miner found a valid block at the same
     height, and the network chose theirs. This is a normal part of mining
     competition. HammerForge's fast-path submission minimizes this risk.

  Q: What is Vardiff?
  A: Variable Difficulty automatically adjusts the mining difficulty sent to
     your miner to maintain a target share rate (1 share every 15 seconds).
     This optimizes network bandwidth and provides smoother hashrate reporting.

================================================================================
  15. LICENSE & CREDITS
================================================================================

  HammerForge is an open-source project.

  Repository:   https://github.com/jpersons73/HammerForge

  Built with:
    - Node.js & TypeScript
    - React & Vite
    - Tailwind CSS & Shadcn UI
    - Recharts

  Supported coin nodes:
    - Bitcoin Core, Bitcoin Cash Node, Bitcoin SV Node
    - DigiByte Core, eCash (Bitcoin ABC), Fractal Bitcoin
    - Peercoin, Litecoin Cash, NitoCoin, BitcoinII
    - Bitcoin Cash II, XeroCoin

================================================================================

  Thank you for using HammerForge.
  Mine your own blocks. Run your own node. Keep 100% of the rewards.

  "Your Home Node Solution"

================================================================================
