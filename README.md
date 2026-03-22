# HammerForge
Automated Home sha256 Nodes for home mining
================================================================================
  SOLO MINER - SHA-256 MULTI-COIN
================================================================================

Mine 8 SHA-256 coins simultaneously with your own hardware and NiceHash rentals.
No pool fees. 100% of block rewards go directly to your wallet.

HammerForge is a fully automated Windows installer package that sets up a private
stratum mining server on your PC. It connects directly to your local full nodes,
builds real block templates, and broadcasts your blocks to the network when you
find one. You are the pool.


================================================================================
  SUPPORTED COINS
================================================================================

  Coin              Ticker   Stratum   RPC      P2P      Node Software
  -----------------------------------------------------------------------
  BitcoinII         BC2      :3333     :8232    :8338    BitcoinII Core v29.1.0
  DigiByte          DGB      :3334     :14022   :12024   DigiByte Core
  Bitcoin           BTC      :3335     :8332    :8333    Bitcoin Core v28.1
  Bitcoin Cash      BCH      :3336     :8432    :8433    Bitcoin Cash Node
  Bitcoin SV        BSV      :3337     :8532    :8533    Bitcoin SV Node
  eCash             XEC      :3338     :8632    :8633    Bitcoin ABC
  Fractal Bitcoin   FB       :3339     :8732    :8733    Fractal Bitcoin
  Peercoin          PPC      :3340     :9902    :9901    Peercoin

All 8 coins run concurrently on separate ports. Each coin has its own process,
its own desktop shortcut, and talks to its own full node.


================================================================================
  HOW IT WORKS
================================================================================

  Architecture:

  Your ASIC Miner(s)              NiceHash Rental
         |                              |
         |    stratum+tcp               |    stratum+tcp
         v                              v
  +------------------------------------------------+
  |         HammerForge Stratum Server              |
  |    (Node.js - one instance per coin)           |
  |                                                |
  |  - Builds block templates from your node       |
  |  - Assigns work to miners                     |
  |  - Validates shares                            |
  |  - Submits blocks when found                   |
  |  - Variable difficulty (vardiff)               |
  |  - UPnP auto port forwarding                  |
  +------------------------------------------------+
         |
         |    JSON-RPC (localhost)
         v
  +------------------------------------------------+
  |         Your Full Node Wallet                  |
  |    (Bitcoin Core, DigiByte Core, etc.)         |
  |                                                |
  |  - Fully synced blockchain                     |
  |  - getblocktemplate for real templates         |
  |  - submitblock when you find a block           |
  |  - Block reward goes to YOUR wallet            |
  +------------------------------------------------+


  The Mining Flow:

  1. BLOCK TEMPLATE
     The stratum server calls getblocktemplate on your local full node to get a
     real block template containing pending transactions and the current block
     header data.

  2. JOB CONSTRUCTION
     The server constructs a stratum mining job from the template, including the
     coinbase transaction (which pays the block reward to YOUR wallet address),
     the merkle branches, and the block header fields.

  3. WORK DISTRIBUTION
     Connected miners receive the job and begin hashing. Each miner gets a
     unique extranonce1 so they never duplicate work.

  4. SHARE SUBMISSION
     When a miner finds a hash below the share difficulty target, it submits
     the share. The server validates the hash by reconstructing the full block
     header and performing a SHA-256d hash.

  5. BLOCK FOUND!
     If the share hash also meets the network difficulty target, the server
     immediately assembles the full block (header + coinbase + transactions)
     and submits it to your node via submitblock. A popup notification appears
     on your screen with the block details.


================================================================================
  DIFFICULTY SYSTEM
================================================================================

  The stratum server runs a variable difficulty (vardiff) system that
  automatically adjusts to any miner size:

  Setting                    Value            Purpose
  ---------------------------------------------------------------------------
  Default Difficulty         65,536           Starting diff for home ASICs
  NiceHash Min Difficulty    500,000          Minimum for NiceHash rentals
  Max Difficulty             4,000,000,000    Upper ceiling
  Target Share Time          5 seconds        How often a miner should submit
  Retarget Interval          15 seconds       How often diff is recalculated
  Variance                   40%              Acceptable deviation from target

  PASSWORD-BASED DIFFICULTY:
  Miners can set a custom starting difficulty by putting diff=500000 in the
  password field during stratum authorization.

  NICEHASH AUTO-DETECTION:
  When a miner connects with a user agent containing "NiceHash" or "nhmp",
  the server automatically sets the minimum difficulty to 500,000.

  DIFF1 TARGET (pdiff):
  The server uses the pdiff (pool difficulty) format:
  0x00000000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF

  This is critical for NiceHash compatibility. Many solo mining pools
  incorrectly use bdiff (0x00000000FFFF0000...), which underreports share
  difficulty by ~29x. NiceHash sees the mismatch and rejects the pool.
  Using pdiff ensures accurate difficulty reporting.


================================================================================
  INSTALLATION
================================================================================

  REQUIREMENTS:
  - Windows 10 or 11 (64-bit)
  - At least one SHA-256 miner on your LAN (Bitaxe, MagicMiner, Luckyminer,
    Avalon Nano, or any SHA-256 ASIC)
  - Internet connection for downloading wallet software and syncing blockchains

  STEP 1: DOWNLOAD AND EXTRACT
  Download the ZIP package and extract it to any folder (e.g., your Desktop).

  STEP 2: INSTALL WALLET(S)
  For each coin you want to mine, right-click the corresponding installer and
  select "Run as administrator":

    INSTALL-BC2-CORE.bat    Downloads and installs BitcoinII Core v29.1.0
    INSTALL-DGB-CORE.bat    Downloads and installs DigiByte Core
    INSTALL-BTC-CORE.bat    Downloads and installs Bitcoin Core v28.1
    INSTALL-BCH-CORE.bat    Downloads and installs Bitcoin Cash Node
    INSTALL-BSV-CORE.bat    Downloads and installs Bitcoin SV Node
    INSTALL-XEC-CORE.bat    Downloads and installs Bitcoin ABC (eCash)
    INSTALL-FB-CORE.bat     Downloads and installs Fractal Bitcoin
    INSTALL-PPC-CORE.bat    Downloads and installs Peercoin

  Each installer:
    - Downloads the wallet from the official GitHub release
    - Installs to C:\Program Files\<CoinName>
    - Creates the data directory in %APPDATA%\<CoinName>
    - Asks if you want to enable blockchain pruning (keeps ~1 month of blocks)
    - Configures RPC credentials in the .conf file
    - Adds Windows Firewall rules for the P2P and RPC ports
    - Creates a desktop shortcut to launch the wallet

  IMPORTANT: After installing, open each wallet and let the blockchain sync
  fully before mining. This can take hours to days depending on the coin.


  BLOCKCHAIN PRUNING
  ------------------
  Every coin installer asks if you want to enable pruning before downloading.
  Pruning keeps only the most recent ~1 month of block data instead of the
  full blockchain, saving significant disk space. Mining works fine with
  pruning enabled.

    Coin   Pruned    Full Chain
    ----   ------    ----------
    BC2    ~5 GB     Small
    DGB    ~5 GB     ~40+ GB
    BTC    ~10 GB    ~600+ GB
    BCH    ~10 GB    ~200+ GB
    BSV    ~10 GB    ~400+ GB
    XEC    ~10 GB    ~200+ GB
    FB     ~5 GB     Small
    PPC    ~2 GB     Small

  You can change pruning later by editing the prune= line in the coin's
  config file.


  STEP 3: SET UP MINING
  Right-click INSTALL-MINING.bat and select "Run as administrator".

  This installer:
    1. Checks for and installs Node.js v20 if not already present
    2. Prompts you for your wallet address for each coin
       (press Enter to skip any coin you don't want to mine)
    3. Configures each wallet's .conf file with RPC server settings
    4. Creates a JSON config file for each coin
    5. Opens firewall ports 3333-3340 for stratum connections
    6. Creates individual desktop shortcuts for each coin

  STEP 4: START MINING
  Double-click any desktop shortcut to start mining that coin.

  The monitoring dashboard automatically opens in your browser, and a CMD
  window shows the server log with:
    - The coin name and stratum port
    - Your wallet address
    - RPC connection status
    - UPnP port forwarding status
    - Connected miners and their hashrates
    - Share submissions in real-time

  You can run all 8 coins simultaneously. Each runs as its own process
  on its own port.

  PORT CONFLICT RECOVERY:
  If an old instance is still running when you start a new one, the server
  automatically detects the conflict, kills the old process, and takes
  over the port. Your miners will briefly disconnect and reconnect.


================================================================================
  WEB MONITORING DASHBOARD
================================================================================

  When you start mining any coin, the monitoring dashboard automatically opens
  in your browser at http://localhost:5000/monitor. It shows:

    - Hashrate chart (30-second interval snapshots, up to 24 hours)
    - Worker table (1-minute, 5-minute, and 1-hour hashrate averages)
    - Share rate bar chart (accepted/rejected over time)
    - Recent shares table with difficulty and status
    - Blocks found table with height, hash, reward, and acceptance
    - Network stats (block height, difficulty, hashrate, coin price)
    - Best share difficulty achieved
    - Server uptime

  The page auto-refreshes every 5 seconds.


================================================================================
  CONFIGURE YOUR MINERS
================================================================================

  Point your SHA-256 ASIC miners to:

    URL:      stratum+tcp://YOUR_PC_IP:PORT
    Worker:   YOUR_WALLET_ADDRESS.MinerName
    Password: x

  Where PORT is the stratum port for the coin you want to mine:
    BC2 = 3333    DGB = 3334    BTC = 3335    BCH = 3336
    BSV = 3337    XEC = 3338    FB  = 3339    PPC = 3340

  Example (mining BC2 with a Bitaxe):
    URL:      stratum+tcp://192.168.1.100:3333
    Worker:   bc1qYourWalletAddress.Bitaxe1
    Password: x

  EMAIL NOTIFICATIONS (optional):
    Put your email in the miner's Password field to get notified when you
    find a block. The SMTP server must be configured during install (Step 4).

    Password: notif=you@gmail.com
    Password: diff=65536,notif=you@gmail.com    (with custom difficulty)
    Password: x                                  (no notifications)


================================================================================
  NICEHASH INTEGRATION
================================================================================

  HammerForge is fully compatible with NiceHash SHA-256 hash rentals. Rent
  hashpower on the NiceHash marketplace and point it at your stratum server
  to dramatically increase your chances of finding a block.

  HOW TO SET UP NICEHASH:

  1. Start mining any coin. The stratum server automatically opens the port
     on your router via UPnP.

  2. Note your public IP. It's shown in the CMD window:
     "UPnP: External/public IP is X.X.X.X"

  3. Create a new order on NiceHash:
     - Algorithm: SHA256
     - Pool host: stratum+tcp://YOUR_PUBLIC_IP:PORT
     - Worker: YOUR_WALLET_ADDRESS.NiceHash
     - Password: x

  NICEHASH-SPECIFIC FEATURES:
    - Auto-detection: Miners with "NiceHash" or "nhmp" in their user agent
      are automatically given a minimum difficulty of 500,000
    - Version rolling: Full mining.configure support with version-rolling
      extension for AsicBoost
    - Extranonce subscribe: Supports mining.extranonce.subscribe
    - Rapid vardiff: Difficulty adjusts within 15 seconds to handle sudden
      hashrate changes from rental starts/stops


================================================================================
  UPnP AUTO PORT FORWARDING
================================================================================

  When the stratum server starts, it automatically maps the stratum port on
  your router using UPnP (Universal Plug and Play). This means:

    - No manual port forwarding needed in your router settings
    - NiceHash and external miners can connect immediately
    - The mapping refreshes every hour and has a 2-hour TTL
    - When the server stops, the port mapping is automatically cleaned up
    - If UPnP is unavailable, the server continues normally with a warning


================================================================================
  SUPPORTED MINERS
================================================================================

  Miner                        Type              Notes
  ---------------------------------------------------------------------------
  MagicMiner                   USB ASIC          All models supported
  Luckyminer LV06              USB ASIC          SHA-256 mode
  Avalon Nano 3S               USB ASIC          Low-power home miner
  Bitaxe                       Open-source ASIC  All variants
  NiceHash SHA-256             Hashrate rental   Auto-detected, optimized diff
  Any SHA-256 stratum miner    ASIC/Software     Standard stratum v1 protocol


================================================================================
  BLOCK FOUND NOTIFICATION
================================================================================

  When you find a block, a popup window appears with full details:

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

  Blocks are submitted to your local node immediately via submitblock. The
  block reward is paid directly to the wallet address you configured.


================================================================================
  STRATUM PROTOCOL SUPPORT
================================================================================

  The server implements the stratum v1 protocol with the following methods:

  Method                         Support   Notes
  ---------------------------------------------------------------------------
  mining.subscribe               Full      Returns extranonce1 and size
  mining.authorize               Full      Accepts any wallet.worker format
  mining.submit                  Full      SHA-256d validation + block submit
  mining.notify                  Full      Real block templates from your node
  mining.set_difficulty          Full      Vardiff with per-miner adjustment
  mining.configure               Full      Version-rolling (AsicBoost) support
  mining.suggest_difficulty      Full      Miners can request starting diff
  mining.extranonce.subscribe    Full      NiceHash compatibility


================================================================================
  CONFIGURATION FILES
================================================================================

  Each coin's configuration is stored as a JSON file:

    {
      "walletAddress": "bc1qYourAddress...",
      "stratumPort": 3333,
      "rpcHost": "127.0.0.1",
      "rpcPort": 8232,
      "rpcUser": "bc2rpc",
      "rpcPassword": "auto-generated-password",
      "coin": "BC2"
    }

  These are created automatically by INSTALL-MINING.bat. You can edit them
  manually if needed.


================================================================================
  SHARE VALIDATION (TECHNICAL)
================================================================================

  When a miner submits a share:

  1. The server reconstructs the coinbase transaction using:
     coinb1 + extranonce1 + extranonce2 + coinb2

  2. Builds the merkle root by hashing the coinbase with the merkle branches

  3. Constructs the 80-byte block header

  4. Performs SHA-256d (double SHA-256) on the header

  5. Compares the resulting hash against the share difficulty target

  6. If the hash also meets the network target, submits the full block

  The coinbase transaction includes:
    - Block height encoded in script (BIP34)
    - Coinbase tag identifying the solo miner (e.g., /BC2Solo/, /DGBSolo/)
    - Output paying the full block reward to your wallet address
    - SegWit commitment (where applicable)


================================================================================
  FALLBACK TEMPLATES
================================================================================

  If a coin's node is not running or not fully synced, the server generates
  fallback templates so miners can stay connected and keep hashing. However,
  blocks cannot be found or submitted with fallback templates.

  The CMD window clearly warns you:

    *** WARNING: BC2 node STILL not available (RPC port 8232).
        Miners are hashing on FALLBACK templates -- blocks CANNOT be found!
        Sync your node! ***

  Once your node comes online and syncs, the server automatically switches
  to real templates and starts finding blocks.


================================================================================
  UNINSTALL
================================================================================

  Right-click UNINSTALL.bat and select "Run as administrator" to remove:
    - The stratum mining server files
    - Desktop shortcuts
    - Firewall rules

  Your blockchain data and wallets are NEVER touched by the uninstaller.


================================================================================
  FILE STRUCTURE
================================================================================

  HammerForge/
    START-HERE.html           Interactive setup guide (open this first)
    README.txt                This file
    INSTALL-BC2-CORE.bat      BitcoinII Core wallet installer
    INSTALL-DGB-CORE.bat      DigiByte Core wallet installer
    INSTALL-BTC-CORE.bat      Bitcoin Core wallet installer
    INSTALL-BCH-CORE.bat      Bitcoin Cash Node installer
    INSTALL-BSV-CORE.bat      Bitcoin SV Node installer
    INSTALL-XEC-CORE.bat      Bitcoin ABC (eCash) installer
    INSTALL-FB-CORE.bat       Fractal Bitcoin installer
    INSTALL-PPC-CORE.bat      Peercoin installer
    INSTALL-MINING.bat        Mining setup (Node.js + stratum server)
    UNINSTALL.bat             Clean uninstaller
    server/
      stratum.ts              Stratum protocol implementation
      rpc.ts                  JSON-RPC client for full nodes
      routes.ts               Web dashboard API routes
      storage.ts              In-memory state management
      index.ts                Server entry point
    shared/
      schema.ts               TypeScript type definitions
    client/
      src/pages/
        monitor.tsx           Mining monitoring dashboard
        dashboard.tsx         Download page


================================================================================
  SUPPORT DEVELOPMENT
================================================================================

  If you find this project useful, consider donating:

  BCH          qrqtvrm6vvvzq6w8jcu9alcv599lmsgpfv7a87edaa
  BTC          bc1qczmq77nrrqsxn3uk8p007r9pf07uzq2t96w3g5
  DGB          DNi4NsZKH93QxBWn1RfQYizGtrKVhJcz6q
  ETH          0x996c85e9137a9020C2a3d9b4889aA6a35185d368
  LTC          Lcnv6pvW4PPBfz2LSyf9GytSDvxUWwaiSB
  USDC (ERC20) 0x996c85e9137a9020C2a3d9b4889aA6a35185d368
  XEC          ecash:qrqtvrm6vvvzq6w8jcu9alcv599lmsgpfv8sn4zh
  XRP          rJZxqccCyj93RBLBGqCqzxFgr5bURGJrrm


================================================================================
  Solo mining. No pool fees. 100% of block rewards go to you.
================================================================================
