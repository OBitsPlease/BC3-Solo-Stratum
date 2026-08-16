# BitcoinIII Solo Stratum

Local and LAN-accessible solo mining for BitcoinIII (BC3), using the chain's
SHA3-256t proof-of-work algorithm.

## Local installation

- BitcoinIII Core: `C:\Program Files\BitcoinIII`
- Node RPC: `127.0.0.1:38337` only
- BitcoinIII peer node: TCP `38338` with public inbound firewall access
- Stratum: `0.0.0.0:3341`, firewall limited to `LocalSubnet`
- Dashboard: `0.0.0.0:8083`, firewall limited to `LocalSubnet`
- Mining payout: selected from the installer's detected local wallet
- Developer payout: locked internally and intentionally omitted from documentation

## Windows installer

Extract `BC3-Stratum-Windows-Installer.zip`, then double-click
`installer\Run Installer.cmd`. The bootstrap:

- detects and reuses an existing BitcoinIII Core installation and data directory;
- detects custom Core `-datadir` and `walletdir` locations;
- requires Core to close normally and creates a timestamped, SHA-256-verified
  wallet backup before changing any configuration;
- never calls `createwallet` or `getnewaddress` when any wallet data already
  exists;
- selects only an existing wallet and owned receiving address;
- downloads and verifies official BitcoinIII Core only when Core is absent;
- installs private portable Node.js and cloudflared runtimes;
- preserves the existing Core configuration in a backup before merging
  loopback-only RPC settings; and
- installs LAN-only firewall rules and a desktop shortcut.

The distribution archive deliberately excludes `config.json`, `.cookie`,
blockchain data, wallet files, dashboard state, and all other local user data.

The coinbase builder pays exactly 1% of `coinbasevalue` to the locked developer
address. The address, expected P2WPKH script, and 1/100 ratio are compiled into
the application and validated before jobs are created.

## Start

Start BitcoinIII Core first, then:

```powershell
.\scripts\start-all.ps1
```

Start the stack and optionally select local miners:

```powershell
.\scripts\start-all.ps1 -Cpu -Gpu
```

Local mining is opt-in. The desktop launcher defaults to no mining and asks
before starting either miner. CPU mining requires a thread count and thermal
cutoff. GPU mining requires a supported watt limit and thermal cutoff; if the
GPU driver rejects the watt limit, the miner does not start.

The dashboard opens at <http://127.0.0.1:8083>. Other LAN devices use the host
computer's current LAN address on TCP port `8083`.

Each Stratum start also creates a free Cloudflare Quick Tunnel for remote,
view-only dashboard access. The random `trycloudflare.com` address changes on
every restart and is written to `MINER-CONNECTION-INFO.txt` before the desktop
launcher opens the guide. Quick Tunnels are a Cloudflare testing service and do
not provide an uptime guarantee. Wallet sends and payout-address changes remain
blocked through the tunnel.

The dashboard includes day/week hashrate history, found-block markers and value,
a live share monitor, connected-worker statistics, and a wallet manager. Wallet
sends and mining payout-address changes are restricted to the host computer.
Hashrate samples and found blocks are retained in `data\dashboard-state.json`.
Changing the payout address validates it with BitcoinIII Core, saves it to the
active configuration, and immediately issues connected miners a fresh job. The
locked 1% developer output is not affected.

## Connect LAN miners

Use:

```text
URL:      stratum+tcp://YOUR_STRATUM_HOST_IP:3341
User:     YOUR_BC3_ADDRESS.worker-name
Password: x
Algorithm: sha3t / SHA3-256t
```

Only miners supporting triple NIST SHA3-256 are compatible.

## Wallet safety

The dashboard displays wallet balances and transactions to LAN viewers. Sending
coins is restricted at the HTTP server to requests originating from loopback on
this computer. A valid BC3 address, positive amount, browser confirmation, and
the exact text `SEND BC3` are required. BitcoinIII Core signs and broadcasts the
transaction.

Keep verified backups of the wallet selected during installation.

## Local miners

- CPU: a locally compiled GPL HashMiner-CPU executable in Ubuntu 22.04 WSL.
- GPU: HashMiner 1.1.1 in an isolated Ubuntu 24.04 WSL distro with a compatible
  CUDA userspace runtime.

The miner launch scripts require explicit safety settings:

```powershell
.\scripts\start-cpu-miner.ps1 -Threads 2 -MaxTemperature 70
.\scripts\start-gpu-miner.ps1 -PowerLimitWatts 50 -MaxTemperature 70
```

The miners are separate programs and are not part of this stratum's source or
license. Review their licenses and fee behavior before redistribution.

## Tests

```powershell
npm test
```

Tests cover:

- SHA3-256t against a live BC3 mainnet header
- locked developer address, script, and exact fee
- coinbase output construction
- target calculations
- live mainnet `getblocktemplate`
- locked developer-output validation
- complete block serialization through node proposal validation
- loopback-only wallet sends

`test\lan-probe.py` verifies Stratum subscription, authorization, job delivery,
dashboard access, and the remote wallet-send guard from a second LAN machine.
