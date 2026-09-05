# bzminer

A high performance NVIDIA, AMD, Intel, Apple Silicon and CPU cryptocurrency miner
and hardware monitor. Runs on Windows, Linux and macOS.

## Algorithms and coins

| `-a` name | coin | dev fee | devices | what it is |
|---|---|---|---|---|
| `cn/gpu` (or `cn`) | Conceal | 1% | NVIDIA · AMD · Intel · Apple · CPU | A privacy chain with on-chain deposits and interest, on CryptoNight-GPU since its 2021 fork. |
| `cn/gpu` (or `cn`) | Ryo | 1% | NVIDIA · AMD · Intel · Apple · CPU | The chain CryptoNight-GPU was written for; mined here through MoneroOcean. |
| `ergo` | Ergo | 1% | NVIDIA · AMD · Intel · Apple | A smart-contract proof-of-work chain built on an extended UTXO model. Autolykos v2 is memory-hard: it builds a ~2 GB table in VRAM before it can mine, so a card needs the room for it. Wallet addresses start with '9'. |
| `etchash`, `etc` | Ethereum Classic | 0.50% | NVIDIA · AMD · Apple | The original Ethereum chain, still proof-of-work; ECIP-1099 halved its epoch, so its DAG is about half ETHW's at a comparable height. |
| `ethash`, `ethw`, `ethereumpow` | EthereumPoW | 0.50% | NVIDIA · AMD · Apple | The proof-of-work fork kept alive after Ethereum moved to proof-of-stake. |
| `rvn` (or `kawpow`) | Ravencoin | 1% | NVIDIA · AMD · Intel · Apple | A Bitcoin-derived chain for issuing and transferring user-created assets. |
| `xna` (or `kawpow`) | Neurai | 1% | NVIDIA · AMD · Intel · Apple | A Ravencoin fork aimed at tying asset tokens to IoT devices and AI data. |
| `neoxa` (or `kawpow`) | Neoxa | 1% | NVIDIA · AMD · Intel · Apple | A Ravencoin fork built around gaming and in-game reward tokens. |
| `meowcoin` (or `kawpow`) | Meowcoin | 1% | NVIDIA · AMD · Intel · Apple | A community-run Ravencoin fork with the same asset layer. |
| `clore` (or `kawpow`) | Clore | 1% | NVIDIA · AMD · Intel · Apple | A Ravencoin fork whose coin pays for time on a GPU-rental marketplace. |
| `pearl` | Pearl | 1% | NVIDIA · AMD · Intel · Apple · CPU | A zk proof-of-work chain: every share is a STARK proof, so it is far heavier per hash than an ordinary algorithm and the hashrate numbers look small. Also mines solo against your own node. Wallet addresses start with 'prl1'. |
| `randomx`, `rx/0`, `xmr`, `monero`, `zeph`, `zephyr`, `sal`, `salvium` | Monero, Zephyr, Salvium | 1% | CPU | A privacy chain that hides sender, receiver and amount by default. RandomX (rx/0) is deliberately CPU-friendly and ASIC-hostile. A Monero address. The worker name is a separate --worker argument, not a suffix on the address. |
| `sha256d` | — | none | NVIDIA · AMD · Intel · CPU | The open-source SDK example: a complete algorithm plugin - CPU and GPU kernels, pool stratum, bench job source. Not worth mining; SHA-256d is ASIC territory. |
| `verus` | VERUS | 1% | CPU | A hybrid proof-of-work / proof-of-stake chain with an identity and currency protocol on top. VerusHash v2.2 is CPU-only by design. A Verus R-address. The worker name is a separate --worker argument, not a suffix on the address. |
| `warthog` | Warthog | 2% | NVIDIA · AMD · Intel · Apple **and** CPU | A chain whose janushash proof-of-work is deliberately split across both processors: the GPU filters sha256t and the CPU runs VerusHash over the same nonces. Neither half finds anything alone. Wallet addresses are 48 hex characters. |
| `xelis` | XELIS | 1% | NVIDIA · AMD · Intel · CPU | A privacy-oriented BlockDAG with encrypted balances and homomorphic transfers. Wallet addresses start with 'xel:'. |

**Warthog needs both a GPU and the CPU.** Neither half finds anything alone, so
disabling either one stops the rig. Every other algorithm mines on whichever of
its listed device types are present.

The five KawPow coins share one algorithm, and **the name you give `-a` is what
picks the chain you mine**: `-a rvn`, `-a xna`, `-a neoxa`, `-a meowcoin` or
`-a clore` names your chain outright, while `-a kawpow` names only the algorithm.
Each coin has its own address format, so the wallet changes with the name.

"Intel" means an Intel Arc or integrated GPU through OpenCL; "Apple" means an
Apple Silicon GPU through Metal, on macOS. Four algorithms have a Metal path —
`ergo`, `kawpow`, `pearl` and `warthog` — and those are the ones that use the GPU
on a Mac. `xelis`, `randomx` and `verus` mine on the CPU there, and so does
`pearl` alongside its GPU work. macOS has had no CUDA since 10.13, and OpenCL on
an Apple GPU is left to Metal, so an algorithm without a Metal kernel simply has
no Mac GPU to use.

The dev fee is declared by each algorithm, printed in the log at startup, and
listed here straight from the shipped binary — so what you read is what you run.
`bzminer --list-algos` prints your own copy's algorithms and fees as JSON.

## Download

Take the archive for your platform from the [latest release][releases], unpack it,
and check the SHA-256 published beside it.

[releases]: ../../releases

```bash
tar -xzf bzminer_<version>_linux.tar.gz && cd bzminer_<version>_linux
```

One archive per platform. Every algorithm, every hardware monitor, all four
console screens and the web dashboard are compiled **into the binary** — there is
nothing else to download and no plugins folder to manage.

| | |
|---|---|
| `bzminer_<ver>_windows.zip` | Windows x64 |
| `bzminer_<ver>_linux.tar.gz` | Linux x64 |
| `bzminer_<ver>_macos.tar.gz` | macOS, Apple Silicon (arm64) |

Inside: the binary, a launcher per algorithm (`start_xelis.sh` / `.bat` and so
on), `start_multi_algo` for mining two at once in one miner (Pearl on the GPUs,
RandomX on the CPU: `-a pearl,randomx --p1 ... --p2 ...`), `start_proxy_*` /
`start_worker_*` for a farm on one pool connection (one box holds the pool
connection with `--proxy_port 4100`, every other rig mines through it with
`-p bzproxy://<proxy>:4100`, and each rig is a light-blue row in the proxy's
tables and dashboard), a fully commented `config.txt`, and `readme.txt`.

There is also a **lite** build and a separate **plugins** archive. The lite binary
has nothing compiled in and loads everything from a `plugins/` folder beside it;
the content is identical, so take the full build unless you specifically want to
replace one plugin without shipping a new miner.

macOS blocks unsigned downloads outright — if it refuses to open, clear the
quarantine flag once, in the unpacked folder:

```bash
xattr -dr com.apple.quarantine .
```

## Monitoring without mining

bzminer can run as a **hardware monitor with no mining at all**: every sensor it can
read, on the console and in the web dashboard, and nothing hashing.

```bash
./bzminer --dmon
```

Then open **<http://127.0.0.1:4014/>**. No wallet, no pool, no algorithm — it
needs none of them. Useful for watching a rig that is busy with something else,
for checking sensors before you commit a card to a job, or simply as a permanent
sensor readout for a machine.

The same thing from `config.txt` (`"dmon": true`), and the same thing implicitly:
a run with no pool configured, or with `"pool": []`, falls into monitoring mode
rather than exiting, and says so in the log.

Which sensors are shown is `--metrics`:

```bash
./bzminer --dmon --metrics temps,powers,clocks
./bzminer --metrics ?            # every group and live metric on this machine
```

A group expands to every sensor of that kind the machine actually reports, so
`temps` on a card with per-die sensors gives you all of them without naming one.
Columns nothing reports are dropped rather than filled with dashes. The groups are
`temps`, `clocks`, `powers`, `fans`, `volts`, `currents`, `pcie`, `utilization`,
`memory`, `timings`, `counters`, `performance` and `all`; any single column name
and any vendor metric key (`pcie.tx`, `temp.vr_core`, …) works too.
[More on the monitoring screens below.](#monitor--the-sensor-table)

**One-shot inspection**, no mining, no dashboard, no screen — each prints and
exits:

```bash
./bzminer --gpu-info       # NVIDIA devices as CUDA sees them
./bzminer --cpu-info       # CPU topology and live metrics
./bzminer --ram-info       # memory modules, DRAM timings and SMBus temperatures
./bzminer --list-metrics   # every sensor this machine exposes
./bzminer --list-columns   # every console-table column name available here
./bzminer --list-algos     # algorithms in this build, with their dev fees
```

And the device page prints itself in full, once, when its output is redirected —
a complete hardware report for one card:

```bash
./bzminer -o device --device-select 1 --device-metrics all > rig.txt
```

## Start mining

Open the launcher for your coin, put your wallet in it, run it. Or go straight to
the command line:

```bash
./bzminer -a xelis -p stratum+ssl://us.vipor.net:5177 -w xel:YOUR_WALLET --worker rig1
```

Then open **<http://127.0.0.1:4014/>** for the dashboard.

The three flags that matter are the same for every algorithm: **`-a`** the
algorithm, **`-p`** the pool URL, **`-w`** your wallet. `--worker` names the rig at
the pool and is optional; `--pass` is the pool password where one is wanted. Note
that `-p` is the **pool**, not the password.

Every section below adds `--worker rig1` and nothing else. Anything from
[Overclocking](#overclocking), [several GPUs](#rigs-with-several-gpus--or-several-cpus)
or [pool failover](#pools-backups-and-failover) can be appended to any of them.

### XELIS

xelhash, on **GPUs and the CPU at once** on the same rig.

```bash
./bzminer -a xelis -p stratum+ssl://us.vipor.net:5177 -w xel:YOUR_WALLET --worker rig1
```

Pools: `stratum+ssl://us.vipor.net:5177` ·
`stratum+ssl://us.xelis.herominers.com:1225`. Solo to your own node with
`ws://127.0.0.1:8080`.

### Ergo

Autolykos v2, **GPU only**, and memory-hard: it builds a table of about 2 GB in
VRAM before it can mine, so a card needs the room for it and there is a pause at
startup — and again whenever the table is rebuilt — while it is filled. Cards with
less VRAM than the table needs are reported and skipped rather than quietly
producing nothing.

```bash
./bzminer -a ergo -p stratum+tcp://pool.us.woolypooly.com:3100 -w 9YOUR_ERGO_WALLET --worker rig1
```

On an **Apple Silicon** Mac the table comes out of unified memory, sized against
what Metal says it may use rather than against installed RAM. 2 GB fits an 8 GB
M2 with room to spare — but it is the machine's memory, so leave the Mac something
to live in.

Pools: `stratum+tcp://pool.us.woolypooly.com:3100`. MoneroOcean mines Autolykos2 and
pays in XMR — point `-w` at a Monero address and pin the algorithm in the
password:

```bash
./bzminer -a ergo -p stratum+tcp://gulf.moneroocean.stream:10128 \
  -w 4YOUR_MONERO_WALLET --pass "rig1~autolykos2"
```

### Ethash and etchash — Ethereum Classic, EthereumPoW

**NVIDIA, AMD and Apple Silicon**, and **memory-hard**: both build a DAG in VRAM
before they can mine — several gigabytes of it, growing with the chain — so there
is a pause at startup while it is generated, and again each time the chain crosses
an epoch. A card without the room is reported and skipped rather than quietly
producing nothing.

**The `-a` name picks the chain, and it is not interchangeable.** Ethash and
etchash are the same algorithm, but ECIP-1099 halved Ethereum Classic's epoch, so
the same block height means a *different DAG* on each. Point `-a ethash` at an
Ethereum Classic pool and the miner builds the wrong DAG — a bigger one — and
every share it finds is worthless. Use `etchash` (or `etc`) for Ethereum Classic,
and `ethash` (or `ethw`, `ethereumpow`) for EthereumPoW.

```bash
# Ethereum Classic
./bzminer -a etchash -p stratum+tcp://etc.2miners.com:1010 -w 0xYOUR_ETC_WALLET --worker rig1
```

Because ETC's epoch is halved, its DAG is about **half** EthereumPoW's at a
comparable height — so a card that can no longer hold an ETHW DAG will usually
still mine ETC.

Pools: `stratum+tcp://etc.2miners.com:1010` ·
`stratum+ssl://etc.2miners.com:11010`. MoneroOcean takes etchash and pays in XMR
— point `-w` at a Monero address and pin the algorithm in the password, or it
will serve you whatever it happens to be mining:

```bash
./bzminer -a etchash -p stratum+tcp://gulf.moneroocean.stream:10128 \
  -w 4YOUR_MONERO_WALLET --pass "rig1~etchash"
```

EthereumPoW pools have thinned out a long way. `stratum+tcp://ethw.f2pool.com:6688`
was the one still answering when this release was built — and F2Pool signs you in
with your **account name**, not a wallet address:

```bash
# EthereumPoW
./bzminer -a ethash -p stratum+tcp://ethw.f2pool.com:6688 -w YOUR_F2POOL_ACCOUNT --worker rig1
```

### KawPow — Ravencoin, Neurai, Neoxa, Meowcoin, Clore

**GPU only.** Name the coin, not `kawpow`: the name is what selects the chain you
mine, and each coin has its own address format.

```bash
# Ravencoin
./bzminer -a rvn -p stratum+ssl://pool.us.woolypooly.com:55555 -w RYOUR_RVN_WALLET --worker rig1

# Neurai (rplant carries it on the ASIA host only)
./bzminer -a xna -p stratum+ssl://stratum-asia.rplant.xyz:17029 -w NYOUR_XNA_WALLET --worker rig1

# Neoxa
./bzminer -a neoxa -p stratum+ssl://stratum-eu.rplant.xyz:17069 -w GYOUR_NEOXA_WALLET --worker rig1

# Meowcoin
./bzminer -a meowcoin -p stratum+ssl://stratum-eu.rplant.xyz:17120 -w MYOUR_MEWC_WALLET --worker rig1
```

MoneroOcean takes any KawPow coin and pays in XMR:

```bash
./bzminer -a kawpow -p stratum+tcp://gulf.moneroocean.stream:10128 \
  -w 4YOUR_MONERO_WALLET --pass "rig1~kawpow"
```

No CUDA toolkit, NVRTC, ROCm compiler or Metal source is needed on the rig.
NVIDIA specializes KawPow's three-block random program through the driver's PTX
JIT with precompiled cubins as a fallback; AMD uses offline-native gfx code
objects and Apple a portable offline metallib, neither of which compiles anything
at runtime.

### Pearl

A zk proof-of-work: every share is a STARK proof, so it is far heavier per hash
than an ordinary algorithm and the hashrate numbers look small by comparison.
That is normal — judge it by accepted shares.

```bash
./bzminer -a pearl -p stratum+tcp://us.pearl.herominers.com:1200 -w prl1YOUR_WALLET --worker rig1
```

Pools: `stratum+tcp://us.pearl.herominers.com:1200` ·
`stratum+tcp://prl-us.kryptex.network:7048` ·
`stratum+tcp://pearl-cpu-eu1.luckypool.io:3370`.

Pearl also mines **solo** — point `-p` at your own node's RPC URL instead of a
pool.

### RandomX — Monero

**CPU only.** `-a randomx`, `-a rx/0`, `-a xmr` and `-a monero` all select it.

```bash
./bzminer -a randomx -p stratum+ssl://gulf.moneroocean.stream:20128 -w YOUR_MONERO_WALLET --worker rig1
```

Pools: `stratum+ssl://gulf.moneroocean.stream:20128` ·
`stratum+ssl://xmr.2miners.com:12222` · `stratum+ssl://monero.herominers.com:1118`.

RandomX needs **2080 MB for its dataset plus 2 MB per mining thread**, and drops
to a slower low-memory mode if that does not fit. It uses huge pages where the
system allows them, and the startup line says which it got. More threads is not
always faster: each one wants 2 MB of cache, so past roughly *L3 size ÷ 2 MB* they
compete — see [`--cpu_threads`](#how-much-of-the-cpu-mines). A CPU without AES
instructions runs it roughly four times slower, and bzminer says so.

### VERUS

VerusHash v2.2, **CPU only**. No GPU is used, so a rig can mine this on the
processor while the cards do something else.

```bash
./bzminer -a verus -p stratum+ssl://bzdev.vipor.net:5140 -w YOUR_VERUS_ADDRESS.rig1
```

The wallet is a Verus R-address, optionally with `.worker` appended — that form
carries the worker name, so `--worker` is not needed.

### Warthog

Needs a GPU **and** the CPU together. The GPU filters sha256t and the CPU runs
VerusHash over the same nonces; neither half finds anything alone, so disabling
either one stops the rig. The first two minutes show `calibrating verus` while it
works out how hard to drive the filter — a near-zero rate during that is expected,
not a fault.

```bash
./bzminer -a warthog -p stratum+ssl://us.vipor.net:5120 -w YOUR_48_HEX_WALLET --worker rig1
```

Pools: `stratum+ssl://us.vipor.net:5120` · `stratum+tcp://warthogunited.com:2001` ·
`stratum+tcp://pool.us.woolypooly.com:3140` · `stratum+tcp://us.acc-pool.pw:12000`.
Solo against your own node, either way it exposes work — `http://127.0.0.1:3000`
for RPC, or `stratum+tcp://127.0.0.1:3456` if you started the node with
`--stratum 0.0.0.0:3456`.

Warthog's CPU stage needs hardware AES and carry-less multiply: any x86-64 from
2013 on, or Apple Silicon. On anything else the miner says so and refuses rather
than pretending.

It runs on **Apple Silicon** with both halves on the one chip: the GPU filters
sha256t through Metal and the CPU runs VerusHash through the ARMv8 Crypto
Extensions.

It also plans the whole rig at once, because its two halves feed each other: it
groups CPU workers by shared L3 cache and measures each GPU's PCIe bandwidth so it
knows an x16 slot from an x1 riser, then sizes the candidate flow per card.
`--no-bandwidth-test` skips that measurement if you would rather save the second
per GPU at startup.

### SHA-256d

The open-source example that ships with the plugin SDK. It mines, but SHA-256d is
ASIC territory — it is there to be read and copied, not to earn.

```bash
./bzminer -a sha256d -p stratum+tcp://your-pool:3333 -w YOUR_WALLET
./bzminer --bench -a sha256d      # what start_sha256d does: no pool, just prove it runs
```

### Updating on a mining OS

Both scripts fetch **v100.13**, the version on this page, so they can be
pasted as they are. To move a rig to a later release, change the version at the
top of the script.

**MMPOS** — put this in *miner profile → advanced → "Initiate command prior to
miner launch"*. It downloads once; on every later launch the `if` sees the archive
already in `/tmp` and exits immediately, so it costs nothing per restart.

```bash
export version="v100.13"
if [ -f "/tmp/bzminer_${version}_linux.tar.gz" ]; then
exit 0
else
cd /tmp; wget https://github.com/bzminer/bzminer/releases/download/${version}/bzminer_${version}_linux.tar.gz; tar -xvf bzminer_${version}_linux.tar.gz; sudo cp -adpR bzminer_${version}_linux/bzminer /opt/mmp/miners/bzminer/
fi
```

**Hive OS** — run it over SSH or in the Hive web shell. You do **not** need to
know which bzminer version is installed: Hive keeps one folder per version under
`/hive/miners/bzminer/` and the flight sheet decides which one runs, so this
replaces the binary in *every* bzminer folder it finds and whichever one your
flight sheet points at gets the new build.

```bash
version=v100.13
cd /tmp && wget -q https://github.com/bzminer/bzminer/releases/download/${version}/bzminer_${version}_linux.tar.gz && tar -xf bzminer_${version}_linux.tar.gz || { echo "download failed"; exit 1; }
miner stop
n=0; for d in /hive/miners/bzminer/*/; do [ -d "$d" ] && cp -f "bzminer_${version}_linux/bzminer" "$d" && n=$((n+1)); done
[ "$n" -gt 0 ] && echo "updated $n bzminer folder(s)" || echo "no /hive/miners/bzminer/<version>/ found - install bzminer from the flight sheet first"
miner start
```

It prints how many folders it updated. `0` means bzminer has never been installed
from a flight sheet on that rig, so there is nothing to replace yet.

Both mining OSes rewrite the pool block from their own algorithm list, so an
algorithm bzminer gained after that front-end shipped cannot be picked in their
UI. `--force_algo <name>` (or `force_algo` in `config.txt`) overrides it whatever
wrote it — that is what it is for.

## Console output modes

Four screens, all compiled in, all showing the same run. Press **`o`** to cycle
through them, or start on one with **`-o <name>`** (`--output` in full).

| screen | what it shows |
|---|---|
| [`tui`](#tui--the-dashboard) | boxed dashboard: devices, shares, pool state, scrollable log pane. **The default whenever there is a terminal** |
| [`log`](#log--the-scrolling-log) | plain scrolling log, with the device table reprinted every 30s. **The default when output is redirected** |
| [`monitor`](#monitor--the-sensor-table) | one dense sensors table, `nvidia-smi dmon` style, redrawn in place |
| [`device`](#device--the-hardware-inspector) | full-screen hardware inspector: one page per device, every sensor it publishes |

Three ways to choose one, all naming the same screen:

```bash
./bzminer -o monitor ...        # start on it
```
```json
{ "output": "monitor" }
```
```
c  →  output monitor            # switch live, from the console command line
```

`o` cycles from wherever you are, `d` jumps straight to the device page, and `Esc`
comes back from it.

**If you pipe or redirect bzminer's output it defaults to `log` on purpose** — a
full-screen dashboard sent to a file is just a file full of cursor codes, so that
is what mining OSes get without asking. An explicit `-o tui` is still obeyed;
the fallback only applies when you have not named a screen. `--color` /
`--no-color` overrides the terminal detection for colour specifically.

Every screen is a plugin. In the **full** build all four are compiled in and there
is no way to remove one from the `o` cycle; in the **lite** build they are files in
`plugins/`, so deleting `out_monitor.so` removes `monitor` from the rig entirely.
The web dashboard is the same plugin story and *does* have a switch —
`http_enabled=false` — because it is a listening socket rather than a screen.

### Refresh rates

How often each screen redraws is separate from how often the engine *publishes*
data. The web dashboard and the metric history want that fast; a human reading a
table does not.

```bash
--interval 1000              # the underlying data rate for every screen and the web UI
--tui-interval 400           # 'tui': repaint the dashboard every 400ms
--log-table-interval 30000   # 'log': reprint the device table every 30s (0 = every snapshot)
--device-interval 1000       # 'device': that page's own clock (unset = --interval)
--hashrate-window 30         # seconds the reported hashrate averages over
```

`--hashrate-window` is a different kind of thing from the four above it: they
decide how often a number is *drawn*, it decides what the number *is*. A GPU's
hash counter advances one batch at a time, so an un-averaged rate reads 0, 0, 0,
BURST, 0 — the window is what makes it a rate. Raise it for a calmer number on a
bursty rig, lower it to see an intensity or thread change take effect sooner. The
`avg` figure is unaffected; that is always since start.

`i` / `I` steps the refresh rate up and down live, through round numbers.

### `tui` — the dashboard

The default screen, and the one that shows what the program is doing: a hardware
box, an algorithm box per pool, and a log pane underneath.

Two tables, two column settings, because they answer different questions:

```bash
--device-columns free,total,core,mem,fan,power,temp
     # the HARDWARE box. Same vocabulary as --metrics: single names, sensor
     # groups, or any vendor metric key this rig emits.

--mining-columns id,cfg,tbs,shares,eff,poolhr,hr,status,poolinfo
     # the ALGORITHM box. Its own closed vocabulary:
     #   id name cfg shares accepted rejected pending stale errors
     #   eff poolhr hr avghr power temp fan core mem tbs status poolinfo
```

`bzminer --list-columns` prints every name all three tables accept on *your*
machine, vendor keys included.

Worth knowing about four of them:

- **`cfg`** says how each device is *configured*, as opposed to what it is doing:
  `i64` is a GPU at intensity 64, `i0` a GPU on auto, `t16` a CPU mining on 16
  threads. The quickest way to confirm a setting actually took.
- **`tbs`** is the *measured* average time between shares found — what a device
  delivers, as opposed to the `est. tbs` its difficulty predicts.
- **`poolhr`** is the pool's credited rate as a percentage of the miner's own:
  **100%** means the pool is crediting exactly what your rig reports, below means
  it is finding fewer shares than its hashrate implies, above means luck has run
  your way. It is the number to watch if a miner looks fast but pays badly. Expect
  it to swing over minutes and settle over hours — it is computed from accepted
  shares, so a short run says very little.
- **`poolinfo`** is the pool's own column: height, difficulty, estimated time
  between shares and latency, one per row. Warthog leaves it out by default
  because its status column carries the rig summary and wants the width; name it
  explicitly to get it there too. Drop it and those four pack into `status`
  instead; drop `status` as well and the per-device status text goes with them.

**The log pane scrolls.** Mouse wheel, `Up`/`Down`, `PageUp`/`PageDown`, `Home`
and `End`. While you are scrolled up the pane holds still — new lines keep
arriving behind it rather than dragging the text out from under you — and the bar
says `held N up  [End] live` so a frozen pane is never mistaken for a stalled
miner. `End` returns to following the newest line.

**Selecting and copying text keeps working too.** Scrolling and selecting are
separate actions, and where the terminal can serve both bzminer never takes the
mouse: it asks the terminal to turn wheel ticks into key presses instead. That
covers every Linux and macOS terminal, and on Windows every modern one — Windows
Terminal, VS Code, ConEmu. Drag out a log line to paste into a bug report while the
wheel still scrolls the pane. The old Windows console window (`conhost`, still the
default on Windows 10) is the exception: it cannot translate wheel ticks and hands
a program the wheel only with its own quick-edit selection switched off, so there
the two really are exclusive — and the wheel wins. Run bzminer under Windows
Terminal and have both.

In `--dmon` the tui drops the mining boxes and shows the sensors table instead, so
it stays the screen that shows what the program is doing.

### `log` — the scrolling log

A plain scrolling log: one coloured line per event, with a compact device and pool
table reprinted every 30 seconds (`--log-table-interval`, `0` for every snapshot).
Nothing repaints in place, so it is what redirection, `journald`, and mining-OS
log windows want — and it is what they get automatically.

Every line carries its level in that level's colour: `[error]` red, `[warn]`
yellow, `[debug]` light blue, `[network]` purple, `[trace]` dark grey. Info is
untagged on purpose — it is what a normal run is made of.

```bash
--log-level info          # network | debug | info (default) | warn | error
-v, -vv, -v2 …            # one step more verbose each
-q, -qq, -q2 …            # one step quieter each
```

`+` and `-` move the level live, on any screen. `--log-level network` prints the
raw stratum traffic, which is the fastest way to settle an argument with a pool
about what was actually sent; `--log-wire-max <n>` elides fields longer than *n*
characters in those lines so a Pearl share's ~140 KB proof does not bury the
frames either side of it (`0` = raw).

There is **always a file log**, whichever screen you are watching, and it holds the
same lines *and* the periodic tables:

```bash
--logfile rig1.log            # name/path (default bzminer.worker.log)
--logfile-level debug         # pin the FILE's level; empty = follow the screen, live
--logfile-mode timestamp      # append (default) | overwrite | timestamp
```

`--log-level info --logfile-level debug` is the useful pair: a readable screen and
a file worth sending to support.

`--internal-log` additionally shows developer lines — kernel tile shapes, operand
layouts, per-share prover timings. They are for whoever wrote the kernel, not
whoever is running it, so they are off at every user level, and this is the only
way to reach `trace`.

### `monitor` — the sensor table

The sensors, dense: one in-place table, no boxes, in the spirit of `nvidia-smi
dmon`, with a small log pane at the bottom. It is available while mining as well
as in `--dmon`.

```
   #  device                      used     free    util  memutil    core     mem    temp  memtemp     hot
   0  AMD Radeon RX 9070 XT       4.1G    11.9G      8%       3%      75      96     52C        -     55C
   1  AMD Ryzen Threadripper P       -        -      0%        -    2501       -       -        -       -
   2  NVIDIA RTX 6000 Ada Gene    1.2G    46.8G      0%       0%     210     405     37C      41C       -
  total power 26W
```

`-` means the metric is not available — CPU temperatures without an elevated
driver, for instance. Columns are `--metrics`:

```bash
./bzminer --dmon --metrics temps,powers,clocks
./bzminer --dmon --metrics all
./bzminer --metrics ?                          # everything selectable on this machine
```

Groups (`temps`, `clocks`, `powers`, `fans`, `volts`, `currents`, `pcie`,
`memory`, `timings`, `utilization`, `counters`, `performance`, `all`) expand to
every sensor of that kind the machine actually reports; single names
(`memused`, `memfree`, `memtotal`, `util`, `memutil`, `core`, `mem`, `temp`,
`memtemp`, `hotspot`, `power`, `fan`) and raw vendor keys (`pcie.tx`,
`temp.vr_core`) work too. Unset gives you
`memused,memfree,util,memutil,core,mem,temp,memtemp,hotspot`.

Every tick, each device's selected metrics also go through the log system, so they
scroll in the log pane, land in the log file, and survive a switch to another
screen:

```
[metrics] dev 2 (c1:0) RTX 6000 Ada Generation: used=1.2G free=46.8G util=0% core=210 mem=405 temp=37C
```

`dev N` is the row number — a position, not an identity, since the list is ordered
by sensor-backend priority and a CPU can sit between two GPUs. `(c1:0)` is the PCI
id, which does not move, and is what the `oc` command takes.

This screen can also write **its own** file, separate from the miner log, and only
while it is the screen being viewed:

```bash
--monitor-logfile sensors.txt
--monitor-logfile-mode timestamp     # append (default) | overwrite | timestamp
```

One block per sample. A rolling appended miner log can sit beside a timestamped
file per monitoring session, which is why the two have separate settings.

### `device` — the hardware inspector

A full-screen page answering *what is this device*, for every device in the
machine — GPUs that are mining, GPUs that are not, and the CPU. A selector table
on top, a scrollable detail page below: identity, firmware, board, PCIe link,
memory type and lockable clocks, DRAM timings, and every sensor the driver
publishes.

Press **`d`** from any screen to get here, `Esc` to go back.

```
--------------------------------------------- devices ----------------------------------------------
|  # | pci   | device                                   | arch         | memory          | link    |
|  0 | 29:0  | GeForce RTX 4070 Ti SUPER                | Ada Lovelace | 15.99 GiB       | Gen4 x16|
| ▸1 | 26:0  | Radeon RX 6800 XT                        | gfx1030      | 15.98 GiB GDDR6 | Gen4 x16|
|  2 | 255:0 | AMD Ryzen 5 3600 6-Core Processor        | 6c/12t       | 31.93 GiB DDR4  | --      |
---- 11:45:22 ------------------------------------------------------------------ [<-/->] device ----
```

Its own keys, on top of the shared ones:

| key | |
|---|---|
| `←` / `→` | previous / next device, wrapping — **and aims the tuning keys at it** |
| `↑` / `↓` / wheel | scroll the detail page |
| `PgUp` / `PgDn` / `Home` / `End` | scroll by a screenful, or to the ends |
| `a` | compact ↔ **every** available sensor |
| `/` | filter rows by substring (Enter keeps it, Esc clears) |
| `Esc` | back to `tui` |

`[a]` is opt-in because NVIDIA publishes well over a hundred readings on a modern
card. Sensors are bucketed by category in the same taxonomy and order the web
dashboard uses, so a card's page and its browser card group identically.

Three flags point it at a device with nobody at the keyboard:

```bash
--device-select 1        # open on that row (and aim the tuning keys at it)
--device-metrics all     # start with every sensor shown
--device-interval 2000   # this page's refresh, in ms
```

It refreshes on its own timer and does **not** wait for mining snapshots —
everything on it comes from the hardware inventory — so a monitoring-only run, a
run between pools, and a run whose algorithm publishes nothing all keep it ticking.

**The selection is the overclock target.** Moving the caret with `←`/`→` aims
`g`/`G`, `l`/`L`, `m`/`M`, `k`/`K` and `w`/`W` at the card you are looking at, and
`n` moves the caret as well as the target — the two cannot disagree. Selecting the
CPU leaves the aim on the last card, since there is nothing on a CPU to tune.

Also useful here: a **memory-clock lock is only accepted at one of a card's
discrete states**, so the memory group lists them — `405, 810, 5001, 10251,
[10501]` — with the state the card is currently running in brackets. "Why is
memory at 810" is the question that sends someone to this page.

## The web dashboard

Served at **<http://127.0.0.1:4014/>**, and it is the same data the console shows:
per-device hashrate, shares, pool state, every sensor, and the overclock controls.

```bash
--set http_port=8080            # different port
--set http_address=0.0.0.0      # reachable from the rest of the LAN
--set http_enabled=false        # off
```

It binds to localhost by default. `0.0.0.0` exposes it to anything that can reach
the machine — **there is no password on it**, so put it behind something you trust.
In `--dmon` it is forced on, because in that mode it is the point.

Two tabs, with a poll-interval selector (0.5s / 1s / 2s / 5s / off):

- **Monitoring** — a card per GPU (usage, core/mem/hotspot temp, power, fan,
  clocks vs. rated, PCIe gen×width and TX/RX, GDDR timings), grouped advanced
  sections for fan RPM, rail power/current/voltage, engine utilization, PCIe
  errors and performance limits; an lstopo-style CPU topology map with
  Utilization/Temperature and Boxes/Tree toggles; per-DIMM RAM cards with a
  temperature graph; and storage volumes. **Click any metric to chart it over
  10m / 1h / 1d.** "Show all metrics" reveals every vendor sensor as a graphable
  tile; histories are collected even while the tiles are hidden.
- **Mining** — per-device hashrate, accepted/rejected/pending shares, temps and
  power, and the pool/rig summary, pushed live over a WebSocket.

Overclocking from the dashboard goes through the same implementation as the
console and the command line, so the three cannot disagree.

JSON, for scripting or your own dashboard:

| endpoint | |
|---|---|
| `/api/snapshot` | everything: devices, pools, hashrate, shares |
| `/api/stream` | the same, pushed as it changes (WebSocket) |
| `/api/gpus`, `/api/metrics`, `/api/topology`, `/api/ram`, `/api/storage` | hardware detail |
| `/api/oc` | read and apply overclocks |
| `/status` | the shape other mining dashboards expect |
| `/hive_status` | HiveOS |

## Hotkeys

The same on every screen. `h` prints this list *in the miner*, with the current
setting beside each key that has one, and `h` again closes it.

| key | |
|---|---|
| `h` | this list (again to close) |
| `c` | open the command line |
| `o` | next screen |
| `d` | device detail page |
| `u` | **pool details: url, wallet, worker, password** (again to close) |
| `p` | pause / resume mining |
| `q` | quit |
| `+` / `-` | more / less log detail, live |
| `i` / `I` | refresh faster / slower |
| `t` / `T` | CPU mining threads, up / down |
| `n` | aim the tuning keys at the next card |
| `g` / `G` | core clock offset, ±15 MHz |
| `l` / `L` | core clock lock, ±15 MHz |
| `m` / `M` | memory clock offset, ±50 MHz |
| `k` / `K` | memory clock lock, through the card's own steps |
| `w` / `W` | board power cap, ±5% of the card's range |

**`u` is the one to know when a pool is crediting you nothing.** It prints every
configured pool in failover order with its url, algorithm, wallet, worker name and
password, each on its own line — read from the *config*, not from the snapshot,
because the snapshot only carries the joined `wallet.worker` login, in which a
wallet typo and a worker typo look identical. The password is only shown for a
pool that actually set one, and it never leaves the console: the web host does not
answer that field, so nothing here reaches `/status`.

**The tuning keys move one card, not the rig.** `n` picks which, and its row is
marked `>` in the device tables and `▸` on the device page. A lock and an offset
are separate settings and a card can carry both, which is why each has its own key.
Every step starts from what the card is actually set to, read back per device, so
a rig started with an offset in `config.txt` does not jump on the first press.

### The command line

Press `c`, type, press enter. `Esc` cancels.

| command | what it does |
|---|---|
| `help` | list the commands |
| `pause` / `resume` | stop and restart mining without exiting |
| `oc ...` | overclock — [see below](#live-from-the-console) |
| `intensity <n>` | mining intensity on every device; `0` is auto |
| `level <name>` | log level: `network`, `debug`, `info`, `warn`, `error` |
| `output <name>` | switch screen by name |
| `quit` | exit |

## Overclocking

Three routes — a start script or `config.txt`, the console, or the web dashboard —
all going through **one implementation**, so they cannot disagree with each other.
Every refusal tells you *why* ("needs root/administrator", "the driver would not
expose the clock-offset API for this device") instead of failing quietly.

### What each device type can do

| device | power | core clock | memory clock | fan |
|---|---|---|---|---|
| **NVIDIA** | yes | offset **and** lock | offset **and** lock | duty, temperature target, or automatic |
| **AMD** | — | — | — | duty, temperature target, or automatic |
| **Intel** | — | — | — | — |
| **CPU** | — | — | — | — |

NVIDIA is the full set, through NVML and NvAPI. On AMD only the fan is settable —
ask for a clock or a power limit on a Radeon and bzminer says so by name rather
than reporting a card that is not there. The fan curve below works on both, because
it is computed by bzminer and handed to the driver as a duty. Intel cards report
their fan but expose no way to drive it, and everything else about them is
read-only. There is no CPU overclocking; use
[`--cpu_threads` / `--cpu_affinity`](#how-much-of-the-cpu-mines) to decide how
much of the processor mines instead.

On **Linux every knob is a privileged write** — run as root, or nothing applies.
On AMD, fan control additionally needs the amdgpu overdrive gate, which bzminer
turns on at startup when it is run as root. bzminer restores whatever it changed
when it exits.

### From the command line (or `config.txt`)

```
--oc-power-limit 160             board power cap, watts
--oc-core-clock-offset 150       core offset, MHz
--oc-memory-clock-offset 1000    memory offset, MHz
--oc-lock-core-clock 1600        pin the core clock; 0 unlocks
--oc-lock-memory-clock 810       pin the memory clock; 0 unlocks
--oc-fan-speed 60                fan duty, or a temperature target (below)
--oc-reset                       back to driver defaults
```

Each takes one value for every card, or **one per card**: `--oc-power-limit
160,180` sets 160 W on the first and 180 W on the second. A short list leaves the
remaining cards untouched.

A worked example — mine xelis with the cards capped at 160 W, a mild core offset,
memory pushed harder, and the fans held at a temperature target:

```bash
./bzminer -a xelis -p stratum+ssl://us.vipor.net:5177 -w xel:YOUR_WALLET \
  --oc-power-limit 160 --oc-core-clock-offset 150 --oc-memory-clock-offset 1000 \
  --oc-fan-speed "t:60[25-75] tm:80[50-100]"
```

The same per card, in `config.txt`, where each entry is addressed by its
enumeration index:

```json
{
  "oc": { "power_limit": "160", "fan_speed": "t:60[25-75]" },
  "devices": [
    { "index": 0, "core_clock_offset": "150",  "memory_clock_offset": "1000" },
    { "index": 1, "core_clock_offset": "-100", "power_limit": "130" },
    { "index": 2, "lock_core_clock": "1600" }
  ]
}
```

A field left out of a `devices[]` entry falls back to the global `oc` block, so
the above caps all three cards at 160 W and gives each its own clocks — except
card 1, which asks for 130 W and gets it.

The underscore spellings from bzminer 1.x (`--oc_power_limit`, `--oc_fan_speed`,
`--oc_core_clock_offset`, …) are all accepted, so an old start script needs no
editing.

### Fan control

Takes a fixed duty, or a temperature target with a range to move inside:

```
--oc-fan-speed 60                            just run at 60%
--oc-fan-speed "t:60[25-75]"                 hold the CORE at 60C, using 25-75% fan
--oc-fan-speed "th:70[30-90]"                the same, aimed at the HOTSPOT
--oc-fan-speed "tm:80[50-100]"               the same, aimed at MEMORY temperature
--oc-fan-speed "t:60[25-75] tm:80[50-100]"   both — whichever wants more fan wins
```

That last form is the one worth knowing: a card can sit at a perfectly comfortable
core temperature while its memory cooks, and the second clause is what catches it.
The fan moves gradually toward the target rather than jumping, and stays inside
the range you gave.

A card keeps a fan duty or a locked clock until something clears it, so a run that
is **killed** rather than closed leaves them behind. `bzminer --oc-reset` on its
own undoes them.

### Live, from the console

Press `c`, then:

```
oc                                  what is there, and what each card is set to
oc 150 500                          +150 core, +500 memory on every card
oc 0 core=+150 mem=+500             card 0 only
oc all fan=70                       every card
oc 1 power=300 clock=2600           card 1: 300 W limit, 2600 MHz locked core
oc 2 mem=1200 fan=85                card 2: +1200 memory, fans at 85%
oc 3 memclock=810                   card 3: memory locked to 810 MHz
oc 0 fan=auto                       hand the fan back to the driver
oc reset [dev|all]                  undo it
```

The settings are `core`, `mem`, `power`, `clock`, `memclock` and `fan`. `core` and
`mem` are **offsets** in MHz; `power` is watts; `clock` and `memclock` lock the
core and the memory to an absolute MHz, and `0` unlocks; `fan` is a duty
percentage or `auto`. A device is named by its number, by its PCI address as the
tables print it (`oc 29:0 fan=70`), or by `all`.

A memory lock is only honoured at one of the clocks the board publishes as
lockable — ask for anything else and the driver quietly runs the nearest one it
does support, so bzminer reads the clock back and tells you what the card is
actually doing. The [device page](#device--the-hardware-inspector) lists a card's
lockable clocks, and so does `--gpu-info`.

`fan=auto` is not `fan=0`. Zero means *stop the fan*.

### With the hotkeys

For nudging by feel while watching what a number does. They move **one card** —
the one `n` (or the device page's `←`/`→`) is aimed at, marked `>` in the tables:

| key | step |
|---|---|
| `g` / `G` | core clock **offset**, ±15 MHz |
| `l` / `L` | core clock **lock**, ±15 MHz |
| `m` / `M` | memory clock **offset**, ±50 MHz |
| `k` / `K` | memory clock **lock**, to the card's next / previous supported state |
| `w` / `W` | board power cap, ±5% of the card's range |

An offset and a lock are separate settings on the same domain, which is why each
gets its own key: a locked card can still carry an offset, and `l`/`L` is the only
way to *set* a lock from the keyboard. Each press starts from what the card is
actually running, read back per device, and the result line says which knob moved
and where it landed.

### What to reach for, per algorithm

Starting points, not measured claims — every card is different, and the honest
test is [`--benchmark`](#benchmarking-and-mining-without-a-pool) at a fixed
difficulty, comparing accepted shares.

| algorithm | bound by | what usually pays |
|---|---|---|
| `ergo` | memory bandwidth | memory offset first, then trim the power limit — the core has slack |
| `ethash`, `etchash` | memory bandwidth, almost entirely | memory offset first, then take the power limit down hard — every hash is 8 KB of DAG reads, so the core spends most of its time waiting and the clock buys very little |
| `kawpow` | memory, with a real core component | memory offset, and a core **lock** rather than a big offset: KawPow's power draw is spiky and boost wanders |
| `xelis` | memory | memory offset, power limit down; the core buys little |
| `pearl` | compute | core clock, and **headroom**: the prover is heavy, so do not starve it on power |
| `warthog` | both halves at once | leave the GPU near stock and give the CPU room — the verus stage is what the tuner balances against, and a card pushed until it throttles drags the whole pair down |
| `randomx`, `verus` | CPU cache and memory | nothing here applies; see [`--cpu_threads`](#how-much-of-the-cpu-mines) |

Two rules that hold everywhere: **cap power before you raise clocks** — an
efficiency gain is usually a watt saved, not a megahertz added — and **change one
thing at a time**, because `poolhr` takes hours to mean anything and two changes
at once cannot be told apart.

## Safety limits

Off by default. Turn on what you want and bzminer will park a card that misbehaves
and bring it back when it recovers — **only the offending card stops, the rest of
the rig keeps mining**.

```bash
--set safety.max_temp_c=85      # pause a GPU over 85 C
--set safety.max_power_w=600    # ...or over 600 W board power
--set safety.sustain_s=15       # only after 15s over the limit, not on a spike
--set safety.resume_margin=5    # resume once it is 5 back under
--set safety.resume_s=10        # ...and has stayed there 10s
```

Set the temperature limit first. It works on every card, and hardware in trouble —
a failing 12VHPWR connector included — gets hot.

There are 12VHPWR current limits too (`safety.max_current_a`, and per-pin
`safety.max_pin_current_a` / `safety.max_pin_imbalance_a`), but they need sensors
only some boards carry. `config.txt` says exactly which.

## Benchmarking and mining without a pool

Two ways to make the miner work without pointing it at anyone, answering two
different questions. Neither needs a wallet, an internet connection or an account.

**`--bench`** — mine a job made up locally. No sockets at all. This answers *"does
this machine mine this algorithm"*: it exercises the kernels and the CPU path and
nothing else.

```bash
./bzminer --bench -a ergo         # also: xelis, pearl, warthog, verus, randomx, kawpow, sha256d
```

**`--benchmark [difficulty]`** — the measurement tool. bzminer starts a **real
stratum server inside itself**, on a local port, then connects to it over TCP like
any other pool. The whole client path runs: subscribe, authorize, `mining.notify`,
share submit, the pool's verdict, the a/r/s counters and the pool-hashrate column.
It is the honest way to measure a card, because a share only counts once the server
has accepted it.

```bash
./bzminer --benchmark -a warthog              # default difficulty 1000
./bzminer --benchmark 25000000000000 -a pearl # fixed difficulty 25T
```

**Pick the difficulty deliberately.** Too low and a fast rig floods the local
server with submissions and measures the plumbing instead of the kernel; too high
and you will wait a long time for enough shares to mean anything. As a rule aim for
a share every few seconds: Pearl on a big GPU wants `25000000000000` (25T), xelis
wants something in the thousands.

**The share count is what to trust.** `shares × difficulty ÷ seconds` is a rate
the miner cannot flatter — compare it against the displayed hashrate, and if the
two disagree for long, one of them is lying.

Each benchmark job is held for ten minutes by default so that a memory-hard kernel
is not thrown away mid-batch every time the job refreshes, which would make
fixed-difficulty share accounting read low. `--benchmark-job-interval <ms>` changes
that; set it to something short, say `4000`, if what you actually want to exercise
is how the miner copes with jobs going stale.

Both modes honour everything else: `--nvidia` to test one vendor, `--oc-*` to
measure at a given power limit, `-o log` for a scrolling log instead of the
dashboard. Some algorithms register a benchmark screen of their own (xelis does),
and `--benchmark` opens it by default; `-o tui` overrides that.

For measuring a *narrower* CPU path than the hardware has — to find out what an
instruction set is actually worth on this rig — there are
`--disable_sse`, `--disable_avx2`, `--disable_avx512`, `--disable_vaes`,
`--disable_amx` and `--disable_huge_pages`.

## Mining through a SOCKS5 proxy

One flag, and it covers **every outbound connection** bzminer makes: your pools,
the dev-fee pools, and plugin downloads alike.

```bash
./bzminer -a xelis -p stratum+ssl://us.vipor.net:5177 -w xel:YOUR_WALLET \
  --proxy socks5://127.0.0.1:1080
```

With authentication (RFC 1929 user/pass):

```bash
--proxy socks5://user:password@proxy.example.net:1080
```

In `config.txt` it is `network.proxy`:

```json
{ "network": { "proxy": "socks5://user:password@proxy.example.net:1080" } }
```

The proxy is applied **before** the optional TLS wrap, so it covers plaintext
`stratum+tcp://` and `stratum+ssl://` identically, and the *proxy* resolves the
pool hostname — no DNS lookup for the pool leaves this machine. Empty (the
default) is a direct connection.

It does not touch the web dashboard, which is a listening socket rather than an
outbound one; use `http_address` for that.

## Pools, backups and failover

List as many pools as you like. The **first is primary and the rest are backups**:
bzminer connects to the primary, and on a disconnect rotates through the others
until one answers, then keeps mining there.

**It comes home on its own.** A failed primary is re-probed every five minutes on
a second, independent connection that does the real subscribe and authorize — so a
pool whose TCP is up but whose stratum is broken is never switched to — and when
it verifies, that already-live connection is promoted in place. No reconnect, no
gap, and the backup keeps mining right up to the switch.

```bash
./bzminer -a xelis \
  -p stratum+ssl://main-pool:5177   -w xel:YOUR_WALLET --worker rig1 \
  -p stratum+ssl://backup-pool:5177 -w xel:YOUR_WALLET --worker rig1
```

Each `-p` (or `--url`) starts a new pool, and the flags after it belong to that
pool. Anything you put *before* the first one applies to all of them, so a single
`-a xelis` covers the whole list.

The same thing in `config.txt`:

```json
{
  "pools": [
    { "url": "stratum+ssl://main-pool:5177",   "wallet": "xel:YOUR_WALLET", "worker": "rig1", "algo": "xelis" },
    { "url": "stratum+ssl://backup-pool:5177", "wallet": "xel:YOUR_WALLET", "worker": "rig1", "algo": "xelis" }
  ]
}
```

Backups must be the **same algorithm** as the primary; one that is not is skipped
with a warning saying so. `"pool": 0` picks one entry out of the list, `"pool":
[0, 2]` runs a chosen subset, `"pool": []` is monitoring mode, and omitting it
gives you all of them.

Other pool options: `stratum+ssl://` for TLS (add `--ssl-verify` to check the
certificate chain and hostname), `--pass <password>` where a pool wants one — note
`-p` is the pool URL, not the password — and `--proxy` for
[SOCKS5](#mining-through-a-socks5-proxy).

Press **`u`** in the console at any time to see every configured pool with its
wallet, worker and password, in failover order.

## Mining several algorithms at once

**One bzminer process mines every algorithm you give it**, each on its own
devices. `-a` takes a list, and a flag's *position in that list* is the number it
carries:

```bash
./bzminer -a pearl,randomx --p1 <pool> --w1 <wallet> --p2 <pool> --w2 <wallet>
```

| flag | algorithm |
|---|---|
| `--p1` `--w1` `--worker1` `--pass1` `--devices1` `--cpu_threads1` | algorithm 1 (`pearl`) |
| `--p2` `--w2` `--worker2` `--pass2` `--devices2` `--cpu_threads2` | algorithm 2 (`randomx`) |

No suffix means algorithm 1, so a one-algorithm command line is unchanged. **Where
a flag sits on the line means nothing — only its number does.** A second `--p1` is
a *failover* pool for algorithm 1, not a second algorithm. `start_multi_algo` in
your download is this, filled in.

**GPUs on one coin, the CPU on another** — the common case, since `verus` and
`randomx` are CPU-only and `ergo` and `kawpow` are GPU-only:

```bash
./bzminer -a ergo,randomx \
  --p1 stratum+tcp://pool.us.woolypooly.com:3100 --w1 9YOUR_ERGO_WALLET \
  --p2 stratum+ssl://gulf.moneroocean.stream:20128 --w2 YOUR_XMR_WALLET
```

Left unset, the cards are dealt out among the algorithms that can use them and the
CPU's threads are split evenly between the algorithms that mine it, on separate
processors. To place them yourself, `--devices<N>` names each algorithm's cards
and `--cpu_threads<N>` its share of the processor:

```bash
./bzminer -a xelis,ergo \
  --p1 stratum+ssl://us.vipor.net:5177 --w1 xel:YOUR_WALLET --devices1 07:00,09:00 \
  --p2 stratum+tcp://pool.us.woolypooly.com:3100 --w2 9YOUR_ERGO_WALLET --devices2 03:00
```

Addresses rather than numbers, because a card should keep its assignment if
another is added or pulled. **A GPU belongs to one algorithm**; two entries of the
same algorithm naming *different* devices are two groups.

**Two algorithms on the same card is not possible** and would not help: a GPU
running two kernels gets each of them a fraction of the card. The one algorithm
that genuinely uses two device classes at once is `warthog`, and it does so by
design — it plans CPU workers and GPU flow together, so it wants the whole
machine and shares a rig poorly.

## A farm on one pool connection

One bzminer holds the pool connection for an algorithm and serves the work to
every other bzminer on your network — so a farm shows the pool **one** worker,
and the rigs need no route to the internet of their own:

```bash
# the proxy — mines too, and serves port 4100
./bzminer -a cn -p <pool> -w <wallet> --proxy_port 4100

# every other rig
./bzminer -a cn -p bzproxy://<proxy ip>:4100 --worker rig1
```

`start_proxy_<algo>` and `start_worker_<algo>` in your download are those two
lines, filled in.

Each rig is a **light-blue row** in the proxy's tables and dashboard, named by its
`--worker`, with its own hashrate, power, hottest card and shares; the device page
(`[d]`) and the dashboard open a rig's row to show its individual cards. The proxy
can mine on its own devices as well, or on none at all — add `--devices none` for
a box that only proxies.

**Each rig still pays its own dev fee**, and pays it *through* the proxy: the fee
connection is tunnelled out from the rig, which is why a rig with no internet of
its own still works. The proxy signals when its own slice starts so the whole farm
pays in one window rather than each rig at its own hour.

**Keep a real pool behind the proxy on every rig** (a second `-p`): if the proxy
goes away the rig fails over to it and mines it directly, and comes back when the
proxy returns.

The listener is TLS, with a certificate made fresh at every start that the rigs
pin through their handshake — so a rig will not mine through something that is not
really your proxy. On anything but a private LAN, also set `--proxy_pass <secret>`
on the proxy and `--pass <secret>` on each rig. Run the same bzminer version
everywhere: the job format a proxy serves is versioned, and a mismatched rig is
refused rather than fed work it would search wrongly.

The price of the arrangement is that the pool sees one worker for the whole farm,
so **per-rig statistics live in the proxy, not at the pool**.

## Rigs with several GPUs — or several CPUs

Every device mines by default — every GPU **and** the CPU. Three ways to narrow
it: by type, by individual card, and permanently in the config.

### Which devices are which

Every run prints its roster, and each line carries **both** names a card answers
to — its device number and its PCI address:

```
  device: [0] CPU (11 of 12 threads) [cpu]
  device: [1] NVIDIA GeForce RTX 5060 Ti (0000:07:00) [cuda]
  device: [2] NVIDIA GeForce RTX 3060 (0000:09:00) [cuda]
  device: [3] gfx1100 (0000:03:00) [opencl]
```

The number counts **every** device enumerated, so turning one off does not
renumber the rest. The address does not move at all — it survives a card being
added, removed or re-slotted, which is why it is the better one to write into a
config that has to keep working. Both are accepted everywhere below, and the PCI
address may be given in full (`0000:09:00`) or in the short form the console
tables print (`9:0`).

### By type

```
--nvidia --amd --intel --cpu
```

Naming any of them makes those the *only* ones that mine: `--nvidia` mines the
NVIDIA cards and nothing else, `--nvidia --cpu` adds the CPU back. Give a value
instead to change one type and leave the rest alone — `--amd 0` stops AMD mining
without touching anything else. The same lives under `device_types` in
`config.txt`.

### By individual card — `--devices`

For when four identical cards are fine and the fifth is doing something else, or
when you want one card and nothing more. A comma- or space-separated list of
device numbers, PCI addresses, or a mix of the two:

```bash
./bzminer --devices 2          -a xelis -p ... -w ...   # ONLY device 2
./bzminer --devices 09:00      -a xelis -p ... -w ...   # the same card, by address
./bzminer --devices 1,2        -a xelis -p ... -w ...   # only those two
./bzminer --devices 07:00,09:00 -a xelis -p ... -w ...  # the same two, by address
./bzminer --devices !1         -a xelis -p ... -w ...   # every GPU EXCEPT device 1
./bzminer --devices !09:00     -a xelis -p ... -w ...   # every GPU except that card
```

A bare list is an **allowlist** — only those cards mine. Entries prefixed `!` are
a **denylist** — everything but those. The two cannot be mixed: `0,!1` reads
equally well as "card 0, and also not card 1" and as "everything except 1, plus
0", so bzminer says so and ignores the list rather than picking one meaning and
mining the wrong cards.

Every card the list excludes says so by name at startup, so a typo shows up as a
missing card with a reason rather than as a rig that is quietly slower:

```
  device skipped (--devices 2): [1] NVIDIA GeForce RTX 5060 Ti (0000:07:00)
  device: [2] NVIDIA GeForce RTX 3060 (0000:09:00) [cuda]
```

`--devices` covers **GPUs only**. The CPU is not a card and has `--cpu` and
[`--cpu_threads`](#how-much-of-the-cpu-mines) of its own — so `--devices 2` picks
one GPU without quietly stopping the CPU half of a Warthog rig. To mine on one
card and nothing else at all, add `--nvidia` (or the matching type) to make the
run GPU-only:

```bash
./bzminer --devices 09:00 --nvidia -a ergo -p ... -w ...
```

It is a setting too, so a mining OS that only lets you edit the config can say the
same thing: `"device_select": "09:00"`.

### Permanently, in `config.txt`

`devices[]` is the per-card block, and it holds intensity and overclocks as well
as the on/off switch. Address an entry by `index`, or — better — by `pci`, which
wins where both are present and does not move when the rig changes:

```json
{
  "devices": [
    { "pci": "0000:07:00", "enabled": true,  "intensity": 0 },
    { "pci": "0000:09:00", "enabled": false },
    { "index": 3,          "enabled": true,  "intensity": 20 }
  ]
}
```

bzminer writes `pci`, `name`, `architecture` and `pci_subsystem_id` into each
entry on its first run, so the file ends up saying which card it means without the
rig in front of you.

`device_select` and `devices[].enabled` are **ANDed** — a card turned off in
either place does not mine — so a config can hold the rig's permanent shape while
a `--devices` on the command line narrows one run further.

**Intensity** is how much work one GPU launch is asked for, in units of 65536
nonces, from `1` to `4096`. `0` is auto and means `64` — a ~4M-nonce launch, which
is what every card used before this was configurable, so leaving it alone changes
nothing. Raise it to keep a card busy for longer per launch; lower it to pick up a
new job sooner, to keep a display responsive, or to shorten a launch that is
tripping a driver watchdog. The current value is in the table's `cfg` column, and
`intensity <n>` on the console command line (`c`) changes every device live, with
`0` to put them back on their configured values.

The index counts **every** device enumerated, so disabling one does not renumber
the others.

### How much of the CPU mines

**By default bzminer does not mine on every thread.** It holds back two processors
on anything above 8 threads, and one from 4 up. A CPU miner runs every worker flat
out, so mining all of them leaves the miner's own threads — the watchdog, the
telemetry poll, the console, the pool sockets — fighting workers for a processor,
and the machine feels frozen. On a small box it is worse than a feeling: a
12-thread rig mining Warthog stopped answering SSH entirely, because `sshd` could
not get scheduled either. The cost of holding two back is a percent or two; the
benefit is a rig you can still log into.

```bash
--cpu_threads 12               # mine on 12 threads (bzminer 1.x's option, same meaning)
--cpu_threads 0                # auto — the default described above
--cpu_affinity 0-7,16,18       # mine on exactly these processors
```

`--cpu_affinity` **takes precedence** over `--cpu_threads`: it names the
processors outright, so a count alongside it could only disagree. Whatever is left
over — the processors not mining — is where the miner's own threads are pinned.
Both are also config settings (`cpu_threads`, `cpu_affinity`), and `t` / `T` move
the count live.

If you want every thread mining, ask for it: `--cpu_threads <all of them>`.

Multi-socket machines are detected properly: bzminer reads the real topology —
packages, cores, threads, NUMA nodes and cache layout — and prints it at startup.
CPU mining threads are placed with that layout in mind rather than scattered, so a
dual-socket box does not spend its time moving work between NUMA nodes.

## Configuration

Three ways to say the same thing. The command line always wins:

```bash
./bzminer -a xelis -p stratum+ssl://host:5177 -w <wallet>   # pool flags
./bzminer --set http_port=8080                              # any setting, by path
./bzminer --config myrig.txt                                # a config file
```

A `config.txt` next to the binary is picked up automatically, and the log says
which file it read. Everything a rig owner can change is reachable from both the
command line and the config file — there is nothing you need an environment
variable for.

`bzminer --config-doc` prints a fully commented template of every setting —
generated from the binary itself, so it can never describe an option your build
does not have. `--print-config` prints the resolved settings as JSON, and
`--save-config` writes them, after all three layers, to `config.effective.json`.

Anything bzminer does not recognise is reported rather than ignored, so a flag
that has been renamed shows up as a warning at startup instead of silently doing
nothing. bzminer 1.x's start scripts run as they are: `-p` is the pool URL as it
always was, and the underscore spellings (`--oc_power_limit`, `--oc_fan_speed`,
`--cpu_threads`, `--warthog_verus_hr_target`) are accepted and mapped. The one
flag that changed meaning is **`-o`, which now selects the console screen** — a
URL passed to it is still taken as a pool, with a warning saying so.

## Every command-line option

Printed by the binary itself, so this list cannot describe a flag your build does
not have. Same text as `bzminer --help`.

```
bzminer - GPU/CPU cryptocurrency miner

Usage: bzminer [options]

Run modes:
  --dmon                  Device-monitor mode: local metrics + web dashboard only (no mining)
  --cpu-info              Print CPU topology + live metrics, then exit
  --gpu-info              Print NVIDIA GPU (CUDA) devices, then exit
  --ram-info              Print memory modules, DRAM timings + SMBus diagnostic, then exit
  --list-metrics          List every available metric (backends + per-device, incl. vendor), then exit
  --list-columns          List every console-table column name (metrics / device_columns /
                          mining_columns), incl. this rig's vendor keys, then exit
  --list-algos            Print this build's algorithms + dev fees as JSON, then exit
  --list-plugins          Print this build's plugins (static + dynamic) as JSON, then exit
  --build-info            Print version + whether release secrets were set; exit 3 if not
  --pawnio-dump           Dump the RyzenSMU PM-table (Zen4 per-CCD offset capture), then exit
  --print-config          Print the resolved settings as JSON, then exit
  --config-doc            Print a commented config.txt template, then exit
  --bench, --test         Mine a local bench job (default sha256d; select with -a)
  --benchmark [diff]      Run a local stratum server and mine it over real TCP
                          (default algo xelis, difficulty 1000; e.g. --benchmark -a xelis)
  --test-dev-fee <s>[:<s>]
                          Dev-fee TEST CYCLE: mine for you <s> seconds, then for the
                          algorithm's author, and repeat. `--test-dev-fee 30` is 30
                          seconds each way; `30:10` sets the legs apart. The real
                          schedule pays the fee as a time debt - first slice five
                          minutes in, next an hour later - so this is how you watch a
                          whole cycle without waiting out an afternoon. It only ever
                          pays MORE than the declared fee: a split that would come to
                          less has its dev leg raised until it does not
  --cpu-metrics <0|1>     Collect CPU telemetry (default 1; 0 skips the CPU sensors)
  --tui-width <n>         Console width in characters (0 = follow the terminal)
  --tui-height <n>        Console height in rows (0 = follow the terminal)
  --hashrate-window <s>   Seconds the reported hashrate averages over (default 30;
                          higher is steadier, lower reacts to a change sooner)
  --disable_sse           Pretend this CPU has no SSE4.1/AES/PCLMUL, nor anything
                          above it - for measuring the portable path
  --disable_avx2          ... no AVX/AVX2 (so no VAES or AVX-512 either). This is
                          how to measure the AVX2 path on a newer CPU
  --disable_avx512        ... no AVX-512, and no AMX with it
  --disable_vaes          ... no VAES/VPCLMULQDQ
  --disable_amx           ... no AMX
  --no-bc250-gpu          AMD BC-250: leave the shader clock at its stock 1500 MHz
  --no-bc250-cpu          AMD BC-250: leave 2 of the 8 CPU cores switched off
  --bc250-gpu-clock <mhz> AMD BC-250: forced shader clock (default 1800)
  --disable_huge_pages    Allocate with ordinary pages, to measure what huge pages
                          are actually worth on this rig
  --intensity <n>         GPU mining intensity for every device that does not set its
                          own devices[].intensity (0 = auto). Units of 65536 nonces
                          per launch; shown as i<n> in the mining table's cfg column
  --cpu_threads <n>       How many CPU threads mine (0 = auto, which leaves one or
                          two for everything else so the machine stays usable)
  --cpu_affinity <list>   Which processors mine: 0-7,16,18. Takes precedence over
                          --cpu_threads; the rest run the miner's own threads
  --benchmark-job-interval <ms>  Keep each deterministic benchmark job this long
                          (default 600000; use 4000 to stress stale-job handling)
  --no-watchdog           Disable watchdog supervision/restarts (debugging)
  -h, --help              Show this help, then exit
  -V, --version           Print version, then exit

Display:
  -o, --output <name>     Console front-end: log (scrolling) | tui | monitor | a plugin
                          (-O too. A URL given to -o is taken as a pool, with a warning)
  -I, --interval <ms>     Metric/snapshot cadence (default 1000; feeds log/tui/monitor/web)
  --log-table-interval <ms>  How often the 'log' output reprints the device table
                          (default 30000; 0 = every snapshot)
  --tui-interval <ms>     How often the 'tui' dashboard redraws (default 400)

  PLUGINS define flags of their own, which cannot be listed here - bzminer does not
  know what an installed plugin accepts. Any --flag bzminer does not recognise is
  passed through for a plugin to read, and warned about if none does. See the
  plugin's README; e.g. out_device adds --device-select, --device-metrics and
  --device-interval. `--list-plugins` shows what is installed.
  --metrics <list>        Columns/groups to show (comma/space list; the active output defines
                          the names). Groups expand to all matching sensors, e.g. 'temps',
                          'pcie', 'clocks', 'powers', 'volts', 'currents',
                          'utilization', 'counters', 'performance', 'memory', 'timings', 'all'.
                          '--metrics ?' lists every group and live metric.
  --device-columns <list> Columns on the mining screen's DEVICE table (same vocabulary as
                          --metrics; default free,total,core,mem,fan,power,temp)
  --mining-columns <list> Columns on the MINING table: id, name, cfg, shares, accepted,
                          rejected, pending, stale, errors, eff, poolhr, hr, avghr, power, temp,
                          fan, core, mem, tbs (measured time between shares), status
                          (default id,cfg,tbs,shares,eff,poolhr,hr,status)
  --list-columns          List every column name the three tables accept here, then exit
  --unlock-bc250-cu       AMD BC-250: enable all 40 CUs (24 by default), then exit.
                          Needs root, and an amdgpu module reload afterwards -
                          the command explains exactly what to do.
  --restore-bc250-cu      AMD BC-250: put the factory 24 CUs back, then exit
  --color / --no-color    Force ANSI color on/off (default: auto-detect a terminal)

Devices:
  --nvidia / --amd / --intel / --cpu [0|1]
                          Which device TYPES may mine. With no value the flag is an
                          ALLOWLIST: naming any type means only the named types mine
                          (`--nvidia --cpu` = NVIDIA and the CPU, nothing else). With a
                          value it sets just that type (`--amd 0` turns AMD off and
                          leaves the rest alone). No flags at all = mine everything
  --devices <list>        Which GPUs mine, by device number or pci id (comma or space
                          separated). Bare entries are an ALLOWLIST - only those mine -
                          and '!' entries a DENYLIST; the two cannot be mixed:
                            --devices 0,2       only devices 0 and 2
                            --devices 29:0      only that card, by pci id
                            --devices !1        every GPU except device 1
                            --devices none      no GPU at all (a proxy-only rig)
                          The startup roster prints both ids for every device found.
                          GPUs only - the CPU has --cpu / --cpu_threads of its own
  --bandwidth-test [arg]  Measure each GPU's host<->device PCIe bandwidth at startup and
                          publish it (pcie.h2d / pcie.d2h). on | off | <MiB buffer size>;
                          default on, 64 MiB. Algorithms that stream results off the GPU
                          (e.g. warthog) use it to tell an x16 slot from a x1 riser
  --no-bandwidth-test     Skip it (saves ~1s per GPU at startup)
                          --devices above is the per-card form; devices[].enabled =
                          false in config.txt does the same thing permanently (an index
                          counts every device enumerated, so disabling one does not
                          renumber the rest, and devices[].pci matches instead when set)

Overclocking (NVIDIA; on Linux every knob is a privileged NVML write):
  --oc-power-limit <W>            Board power cap
  --oc-core-clock-offset <MHz>    Persistent VF-curve offset for the core
  --oc-memory-clock-offset <MHz>  Same for memory
  --oc-lock-core-clock <MHz>      Pin the core clock instead of letting boost wander
                                  (0 = unlock)
  --oc-lock-memory-clock <MHz>    Pin the memory clock (0 = unlock)
  --oc-fan-speed <spec>           Either a fixed duty ('60') or a temperature curve:
                                    t:<target>[<min>-<max>]   keep the CORE at <target> C,
                                                              fan within <min>..<max> %
                                    th:<target>[<min>-<max>]  the same for the HOTSPOT
                                    tm:<target>[<min>-<max>]  the same for MEMORY temp
                                  Both may be given - whichever wants more fan wins:
                                    --oc-fan-speed "t:60[25-75] tm:80[50-100]"
  --oc-reset                      Put every card back to driver defaults FIRST (offsets
                                  cleared, clocks unlocked, power default, fan automatic).
                                  Use alone to undo settings a killed run left behind
  Every value may be one number for all cards, or a per-device list ('160,180').
  bzminer 1.x's --oc_* spelling (underscores) is accepted for all of these.

Logging:
  --internal-log          Also show DEVELOPER lines: kernel tile shapes, operand
                          layouts, per-share prover timings. They are for whoever
                          wrote the kernel, not whoever is running it, so they are
                          off at every user level. This is also the only way to
                          `trace`, which carries nothing else.
  --log-level <level>     network | debug | info (default) | warn | error
                          `network` shows the exact pool/stratum wire messages
  --logfile <file>        Write the log to <file> instead of bzminer.worker.log. There
                          is always a file log; this only changes its name/path.
                          It holds what the `log` SCREEN holds - the lines AND the
                          periodic tables - whichever screen you are watching.
  --logfile-level <level> the same, for the LOG FILE only. Empty = follow --log-level,
                          including the live [+/-] keys. Setting it PINS the file:
                          `--log-level info --logfile-level debug` keeps the screen
                          readable and still writes a file worth sending to support.
  --logfile-mode <mode>   append (default) | overwrite | timestamp. `timestamp` keeps
                          every run: bzminer.worker.<date>_<time>.log
  --monitor-logfile <f>   Write the MONITOR screen's sensor table to <f>, one block per
                          sample, ONLY while that screen is being viewed. Unset = off.
  --monitor-logfile-mode  append (default) | overwrite | timestamp, for that file
  -v, -vv, -v2 ...        More verbose (each step toward network; -v2 == -vv)
  -q, -qq, -q2 ...        Quieter (each step toward error)
  --log-wire-max <n>      In `network` logs, elide any single field longer than n chars
                          (default 256). Long values become <elided, N chars> and runs of
                          unprintable bytes become <binary, N bytes>; the rest of the
                          frame is logged exactly as sent. 0 = raw, nothing removed.

Configuration:
  --config <path>         Config file to load (default: config.txt)
  --set <path>=<value>    Override any setting below (e.g. --set http_port=8080)
  --<path> <value>        Same as --set, as a flag (dashes map to '_')
  --save-config           Write the resolved config to config.effective.json
  -a, --algo <algo>[,..]  The algorithm(s) to mine. A comma list mines them ALL AT
                          ONCE, each on its own devices: the first is algorithm 1,
                          the second 2, and so on (--a1/--a2 name them singly)
  -p, --pool, --url <url> Add a pool for algorithm 1. Repeat for backups: the first
                          is the primary, the rest are rotated through on
                          disconnection. --p2 <url> is algorithm 2's pool list
  -w, --wallet <addr>     Payout wallet (--w2 for algorithm 2)
  --worker <name>         Worker/rig name (login sent to the pool: wallet.worker)
  -u, --user <user>       LEGACY combined login (use --wallet/--worker instead)
  --pass <pass>           Pool password (NOTE: -p is the pool URL)
  --ssl-verify            Verify the pool's TLS cert (stratum+ssl; default off)
  --devices<N> <list>     Which devices mine algorithm N: numbers, pci ids, or the
                          types cpu, gpu, nvidia, amd, intel (--devices2 cpu). Unset,
                          the GPUs are dealt out among the algorithms that can use
                          them and the CPU is shared by every one that can
  --cpu_threads<N> <n>    Algorithm N's share of the CPU when it is shared; unset =
                          an even split of the rig's --cpu_threads budget. Placed on
                          unused processors first, overlapping only once they run out
  --cpu_affinity<N> <l>   ...or algorithm N's processors, named outright
  --proxy_port<N> <port>  Be the farm's PROXY for algorithm N: keep the one pool
                          connection and serve its work to other bzminer instances
                          started with `-p bzproxy://<this host>:<port>`. Their shares
                          go upstream from here and each instance is a light-blue row
                          in the tables here. The port is TLS. Each rig keeps its own
                          dev fee and pays it through a tunnel this proxy opens to the
                          fee pools, so the rigs need no internet of their own. Mine on
                          this rig's own devices too, or on none (--devices none)
  Every flag above takes the same <N> suffix: no suffix means algorithm 1, so a
  one-algorithm command line is unchanged. WHERE a flag sits on the line means
  nothing - only its number does. bzminer 1.x's --a1/--p1/--w1/--r1/
  --pool_password1/--pool_devices1 are the same flags.
  --proxy <url>           Route all pools through a SOCKS5 proxy: socks5://[user:pass@]host:port
  --proxy_bind <addr>     Address the --proxy_port listener binds (default 0.0.0.0)
  --proxy_pass <pass>     Password instances must give to connect to this proxy (their
                          --pass); empty = any instance that can reach the port
  --disable_udp           Stop announcing this rig on UDP 4014 and stop listening for
                          other bzminers. That announce is what fills the web UI's farm
                          view: every instance on the network, their combined hashrate
                          and power, and a link to each one's dashboard
  --plugin-update <mode>  Auto-download/update plugins: on | off | auto (lite=on, full=off)
  --plugin-manifest <url> Override the plugin manifest URL (beta channel / testing)
  --pool <index|[0,2]>    ...or, given a number instead of a URL, which configured
                          pool(s) to activate; [] = monitoring mode
  --force_algo <algos>    Override the pools' algorithms, whatever wrote them: one
                          name for pool 0, or a list - pearl,randomx or [pearl,randomx]
                          or two words - for one pool each, in order. Also a
                          top-level config.txt option (a string or a JSON array),
                          which is where a mining OS whose UI lacks the algorithm
                          wants it
  --cu-kernel [algo=]<f>  Override with an offline-compiled CUDA .cubin
  --cl-kernel [algo=]<f>  Override with an OpenCL .spv or native .bin
                          (source/PTX overrides are not accepted)

Settings (set in config.txt, or with --set <path>=<value>):
  log.level                  Log verbosity: network, debug, info, warn, or error (network = the exact pool/stratum wire messages). `trace` is developer detail and is reached only with --internal-log; a config that still says trace is read as network
  log.file                   Write the log to this file instead of bzminer.worker.log. There is always a file log; this only changes its name/path
  log.file_level             Verbosity of the LOG FILE, when it should differ from the console. Same names as `level`. Empty = follow `level`, INCLUDING live changes from the [+/-] keys. Set this to pin the file: e.g. file_level debug with level info keeps the screen readable while still capturing a file worth sending to support (CLI: --logfile-level)
  log.file_mode              What to do with an existing log file: append (default - what bzminer has always done), overwrite (truncate, so the file is only this run), or timestamp (leave it and open bzminer.worker.<date>_<time>.log, keeping every run). A watchdog RESTART never truncates - it would delete the crash that caused it - so overwrite appends from the second worker onwards. CLI: --logfile-mode
  log.monitor_file           Write the MONITOR screen's sensor table to this file, one block per sample. Only while that screen is being viewed - on any other output there is nothing of its to record. Empty (default) = no monitor log. CLI: --monitor-logfile
  log.monitor_file_mode      The same three choices as `file_mode`, for the monitor log. Separate so a rolling appended miner log can sit beside a timestamped file per monitoring session. CLI: --monitor-logfile-mode
  log.wire_max               Longest single field kept in a `network`-level wire log line, in characters. Anything longer becomes "<elided, N chars>" and runs of unprintable bytes become "<binary, N bytes>" - the rest of the frame is logged exactly as it went over the wire, so a Pearl share's ~140 KB proof no longer buries the frames either side of it. 0 prints raw frames with nothing removed. CLI: --log-wire-max
  network.timeout_ms         Pool / network socket timeout, in milliseconds
  network.reconnect_ms       Delay before reconnecting after a disconnect, in milliseconds
  network.proxy              Route all pool connections through a SOCKS5 proxy: socks5://[user:pass@]host:port (empty = direct). CLI: --proxy
  plugins.update             Auto-download/update plugins from the manifest: auto|on|off (auto = on for the lite build, off for the full build). CLI: --plugin-update
  plugins.manifest_url       Override the plugin manifest URL (empty = built-in GitHub default + bzminer.com fallback). CLI: --plugin-manifest
  safety.max_temp_c          Pause a GPU (mining stops on that card only, auto-resumes when it cools) when its core or hotspot temp exceeds this, in C. 0 = off
  safety.max_power_w         Pause a GPU when its board power exceeds this, in W. 0 = off
  safety.max_current_a       Pause a GPU when its 12VHPWR CONNECTOR current (aggregate of all pins, not per-pin) exceeds this, in A. NVIDIA Blackwell on Windows only. 0 = off
  safety.max_pin_current_a   Pause a GPU when its WORST SINGLE 12VHPWR pin exceeds this, in A - the connector failure mode the aggregate cannot see. Needs per-pin shunts: ASUS ROG Astral RTX 5080/5090 on Windows only. 0 = off
  safety.max_pin_imbalance_a Pause a GPU when the difference between its highest- and lowest-current 12VHPWR pins exceeds this, in A. ASUS ROG Astral RTX 5080/5090 on Windows only. GPUProbe's sample alarm uses 5.5 A. 0 = off
  safety.sustain_s           How long a card must stay over a limit before it is paused, in seconds (rejects transient spikes)
  safety.resume_margin       Resume only once the value drops to (limit - this), in the tripped limit's unit (C/W/A) - hysteresis so it does not flap
  safety.resume_s            And stayed under the resume point this long, in seconds
  dmon                       Device-monitor mode: telemetry + web UI only, no mining (CLI: --dmon)
  http_enabled               Serve the monitoring web UI + JSON API (needs the webui plugin; no plugin = no HTTP server)
  http_address               Web server bind address ("0.0.0.0" allows LAN access)
  http_port                  Web server TCP port
  disable_udp                Stop this rig announcing itself on UDP 4014 and stop it listening for other bzminers. That announce is what fills the web UI's farm view - every instance on the network, their combined hashrate and power, and a link to each one's dashboard. It carries the rig name, web port, version and this rig's totals, and nothing else; turn it off on a network you do not control. CLI: --disable_udp
  proxy_bind                 Address the proxy listener (pools[].proxy_port) binds: "0.0.0.0" for every interface, or one interface's address. Rig-wide - a farm has one network. CLI: --proxy_bind
  proxy_pass                 Password every instance connecting to this proxy must present (its --pass). Empty accepts any instance that can reach the port - fine on a private LAN, not elsewhere. CLI: --proxy_pass
  save_effective             On startup, write the resolved config to config.effective.json
  benchmark_diff             Run a local stratum server at this fixed share difficulty and mine against it - no pool, no wallet. 0 = off. The CLI form is --benchmark [difficulty]
  benchmark_job_interval_ms  How long --benchmark keeps one deterministic job, in ms. A persistent job makes fixed-difficulty share accounting honest for memory-hard kernels. CLI: --benchmark-job-interval
  log_table_interval         How often the 'log' output reprints the device/pool table, in ms. Separate from --interval, which is how often the engine PUBLISHES a snapshot (the web UI and metrics want that fast). 0 = print on every snapshot. CLI: --log-table-interval
  tui_interval               How often the 'tui' dashboard redraws, in ms. A keypress redraws immediately regardless. CLI: --tui-interval
  metrics                    Columns on the MONITORING sensor table (--dmon, and the monitoring screen of the tui/log outputs). Comma- or space-separated. Takes single columns (memused, memfree, memtotal, util, memutil, core, mem, temp, memtemp, hotspot, power, fan), sensor GROUPS that expand to everything present (temps, clocks, powers, fans, volts, currents, pcie, memory, utilization, performance, all), and any vendor metric key this rig's backends emit (pcie.tx, temp.vr_core, ...). Empty = memused,memfree,util,memutil,core,mem,temp,memtemp,hotspot. Run 'bzminer --list-columns' for every name available on THIS machine. CLI: --metrics
  device_columns             Columns on the mining screen's DEVICE table (the hardware box above the algorithm box). Same vocabulary as metrics above. Empty = memfree,memtotal,core,mem,fan,power,temp. Run 'bzminer --list-columns' for the full list. CLI: --device-columns
  mining_columns             Columns on the MINING table (the algorithm box: shares, hashrate, pool). Its own vocabulary - id, name, cfg, shares, accepted, rejected, pending, stale, errors, eff, poolhr, hr, avghr, power, temp, fan, core, mem, tbs, status, poolinfo. Empty = id,cfg,tbs,shares,eff,poolhr,hr,status,poolinfo. `tbs` is the MEASURED average time between shares found - what a device delivers, as opposed to the `est. tbs` its difficulty predicts. `poolinfo` is the pool's own column - height, difficulty, est. tbs and latency, one per row. Warthog leaves it out by default, because its status column carries the rig summary and wants the width; name it explicitly to get it there too. Drop it and those four pack into 'status' instead. Drop 'status' as well and the per-device status text goes with them - nothing is left wide enough to hold it. Run 'bzminer --list-columns' for the full list. CLI: --mining-columns
  cpu_metrics                Collect CPU telemetry (per-core clocks, temperatures, usage, CCD sensors). Set false on a GPU rig that does not want them, or where the reads need a privileged driver - GPU metrics are unaffected. CLI: --cpu-metrics 0
  cache_qos                  Reserve a private slice of the last-level cache for each CPU mining thread (Linux, root, and a CPU with cache allocation - AMD Zen 2 and newer, or Intel with RDT). RandomX's 2 MiB scratchpads add up to exactly the L3 on a typical rig, so left alone they evict one another; fencing them off measured +4.7% on a Threadripper PRO 9955WX. Machine-wide while mining and restored on exit. CLI: --cache-qos 0
  superio_fans               Read the motherboard's sensor chip directly for fan speeds when no kernel driver publishes them (Linux, x86, and only as root). Many boards - mini-PCs especially - carry an ITE or Nuvoton chip that Linux has no driver for, and then nothing reports a CPU fan at all. Reads only, and never runs where hwmon already has fans. Set false to leave that chip alone. CLI: --superio-fans 0
  tui_width                  Console width in characters. 0 = use the terminal's own width and follow a resize, which is the default and is what you want. Set it to pin the width - for a terminal that misreports, or output being captured with no tty to ask. Pinning it LARGER than the real window makes every row wrap and the dashboard scroll, so measure before setting it. CLI: --tui-width
  tui_height                 Console height in rows. 0 = use the terminal's own height and follow a resize. Independent of tui_width: pinning one leaves the other tracking the window, and where there is no window to ask the unpinned one falls back to 80x25 rather than discarding both. CLI: --tui-height
  hashrate_window            Seconds the reported hashrate averages over. A GPU's hash counter advances one batch at a time, so an un-averaged rate reads 0, 0, 0, BURST, 0; this window is what makes it steady. Raise it for a calmer number on a bursty rig, lower it to see an intensity or thread change take effect sooner. Does not affect the 'avg' figure, which is always since start. CLI: --hashrate-window
  disable_sse                Behave as if this CPU had no SSE4.1/AES/PCLMUL, nor anything built on them. For measuring a narrower path than the hardware has - here, the portable fallback. CLI: --disable_sse
  disable_avx2               Behave as if there were no AVX/AVX2. VAES and AVX-512 go with it, since both need AVX: a CPU with AVX-512 and no AVX2 does not exist, and pretending otherwise would select a path you meant to exclude. This is the switch for measuring the AVX2 path most Zen3-and-older rigs actually run. CLI: --disable_avx2
  disable_avx512             Behave as if there were no AVX-512. AMX goes with it, riding on the same register state. CLI: --disable_avx512
  disable_vaes               Behave as if there were no VAES/VPCLMULQDQ. CLI: --disable_vaes
  disable_amx                Behave as if there were no AMX. CLI: --disable_amx
  bc250                      AMD BC-250 only; ignored on every other machine. The board ships deliberately cut down - the shader array locked to 1500 MHz, and 2 of its 8 CPU cores switched off - and neither is a fuse, so both are undone at startup. `gpu` forces the shader clock: measured on a BC-250, warthog's sha side gains 32% and its janus score 12%, while xelis gains nothing, being bound by memory rather than compute. `cpu` enables all 8 cores and takes effect on the NEXT boot, the core mask being read when the OS enumerates CPUs. Both are volatile - a cold power cycle restores the factory setup. CLI: --no-bc250-gpu / --no-bc250-cpu / --bc250-gpu-clock
  disable_huge_pages         Allocate with ordinary pages rather than huge ones. Huge pages are meant to help - a large ring is far fewer TLB entries at 2 MB than at 4 KB - but on a working set that already fits the TLB they buy nothing while still needing privileges. This is how a rig owner finds out which case theirs is. CLI: --disable_huge_pages
  pool                       Active pool(s) from pools[]: an index (0 or "0"), an array ([0, 2] = multiple pools), or [] for monitoring mode. Omit = all pools (first primary, rest failover). CLI: --pool
  force_algo                 Override the algorithm on the configured pools, whatever wrote them. One algorithm ("warthog") sets pool 0; a list sets one pool each, in order - either JSON (["warthog", "xelis"]) or comma separated ("warthog,xelis"). It is TOP-LEVEL, not a field inside pools[], because a mining OS rewrites the pool block from its own algorithm list - so an algorithm bzminer gained after that front-end shipped cannot be selected in its UI, and an override placed inside pools[] would be overwritten by it. CLI: --force_algo
  device_select              Which GPUs mine, as a comma- or space-separated list of device numbers and/or pci ids ("0,2", "29:0", "0000:29:00"). Bare entries are an ALLOWLIST - only those cards mine - and entries prefixed '!' a DENYLIST ("!1" = every GPU but device 1). The two cannot be mixed, since "0,!1" has two readings and neither is obviously right. Empty = every GPU mines. This is the one-line form of devices[].enabled and the two are ANDed, so a card disabled in either place does not mine. GPUs only: the CPU is governed by device_types.cpu / cpu_threads. CLI: --devices
  intensity                  Mining intensity for every GPU that does not name its own in devices[]: how much work one launch is asked for, in units of 65536 nonces (1-4096). 0 = auto, which lets the algorithm choose. devices[].intensity is addressed by enumeration index and still wins where it is set, so this is the one to use for a whole rig. Shown as i<n> in the mining table's cfg column. CLI: --intensity
  cpu_threads                How many CPU threads mine. 0 = auto, which holds back one processor on a small machine and two above 8 threads, so the miner's own threads - and yours - are not competing with workers. Set it to the full thread count to mine on everything. Caps every CPU pool the run starts, including an algorithm's own - Pearl's share prover used to size itself from the machine and ignore this. Movable while mining with the console's [t]/[T] keys. CLI: --cpu_threads
  cpu_affinity               Which processors mine, as a list: "0-7,16,18". Takes PRECEDENCE over cpu_threads, since it names the processors outright. Everything not listed is left for the miner's management threads. CLI: --cpu_affinity
  bandwidth_test             Measure each GPU's host<->device PCIe bandwidth once at startup and publish it (pcie.h2d / pcie.d2h). Costs ~1s per GPU; algorithms that stream results off the GPU use it as a hard ceiling. CLI: --bandwidth-test
  bandwidth_test_mb          Per-copy buffer size for that measurement, in MiB. Large enough to measure throughput rather than launch latency
  test_dev_fee_user_seconds  Dev-fee TEST CYCLE: mine for you this many seconds, then for the algorithm's author, and repeat. 0 = off, the normal schedule. The real one pays the fee as a time debt, so its first slice is five minutes into a run and its second an hour later - correct, and impossible to sit and watch. This makes the whole cycle visible in a minute: the switch, the dev pool's subscribe, the switch back, your own work resuming. A diagnostic, not a tuning knob - the cycle can only ever pay MORE than the declared fee, never less. CLI: --test-dev-fee <user_s>[:<dev_s>]
  test_dev_fee_dev_seconds   How long each dev leg of that cycle runs. 0 = the same as test_dev_fee_user_seconds, so a bare `--test-dev-fee 30` is 30 seconds each way. Raised automatically if the split you asked for would come to less than the declared fee
  pools[].url                Pool URL (CLI: -p / --pool / --url; stratum+ssl:// for TLS). Repeat the flag for backups - the first is the primary and the rest are rotated through on disconnection
  pools[].wallet             Payout wallet address (CLI: -w / --wallet)
  pools[].worker             Worker / rig name; sent to the pool as wallet.worker (CLI: --worker)
  pools[].pass               Password (CLI: --pass; note -p is the POOL URL, as in bzminer 1.x)
  pools[].algo               Algorithm (CLI: -a / --algo). Active pools are grouped by algorithm and every algorithm mines AT ONCE, each on its own devices: entries sharing an algorithm are one group, primary first, the rest its failovers
  pools[].ssl_verify         stratum+ssl: verify the pool's TLS certificate chain + hostname (CLI: --ssl-verify)
  pools[].user               LEGACY combined login, sent verbatim instead of wallet.worker. Only for a pool that wants something other than that shape - set wallet and worker instead (CLI: -u / --user)
  pools[].devices            WHICH devices mine this pool's algorithm when the rig runs more than one: device numbers, pci ids, '!' exclusions, and the types cpu, gpu, nvidia, amd, intel. Empty = this algorithm's default share - the GPUs are dealt out among the algorithms that can use them, and the CPU is shared by every algorithm that can. A GPU belongs to one algorithm; two entries of the same algorithm naming DIFFERENT devices are two groups (CLI: --devices<N>)
  pools[].cpu_threads        How many of the rig's CPU threads this algorithm gets when it shares the CPU with another. 0 = an even share of the rig-wide cpu_threads budget. Placed on unused processors first; counts that add up to more than there are overlap and share (CLI: --cpu_threads<N>)
  pools[].cpu_affinity       ...or the processors this algorithm mines on, named outright ("0-7,16"). Wins over cpu_threads and is honoured exactly, overlaps included (CLI: --cpu_affinity<N>)
  pools[].proxy_port         Serve this algorithm's work to OTHER bzminer instances on this TCP port: this instance keeps the one pool connection, and every bzminer started with -p bzproxy://<this host>:<port> mines the same jobs through it. Their shares go upstream from here, and each instance is shown here as one light-blue row with its hashrate, power and devices. The port is TLS. Each rig keeps its own dev fee and pays it through a tunnel this proxy opens to the algorithm's fee pools, so the rigs need no internet of their own; this proxy signals its own slice so the farm pays in one window. A proxy may mine on its own devices as well, or on none (--devices none). 0 = off. Per algorithm, like devices (CLI: --proxy_port<N>)
  devices[].index            Device index - counts EVERY device enumerated, so disabling one does not renumber the others
  devices[].intensity        Mining intensity: how much work one GPU launch is asked for, in units of 65536 nonces (1-4096). 0 = auto, which is 64. Higher keeps the card busy longer per launch; lower picks up a new job sooner. Shown as i<n> in the mining table's cfg column
  devices[].enabled          Whether to mine on this device. To turn off a whole vendor or the CPU instead, use device_types below
  devices[].pci              WHICH card this entry is, as "domain:bus:device" - written by bzminer on first run, and the field entries are matched on. Preferred over index, which moves when a card is added or removed
  devices[].name             The card's name, written by bzminer so the file says what it means without the rig in front of you. Not read back - pci is the key
  devices[].architecture     The card's architecture ("Ampere", "RDNA3"), written by bzminer. Not read back
  devices[].pci_subsystem_id The board's PCI subsystem pair, "vendor:device" - written by bzminer. The device id alone names the CHIP, which is the same on a reference card and a partner's; this names the board. Not read back
  devices[].core_clock_offset This card's core VF-curve offset in MHz. Empty = use the global oc.core_clock_offset
  devices[].memory_clock_offset This card's memory VF-curve offset in MHz. Empty = use the global oc.memory_clock_offset
  devices[].power_limit      This card's board power cap in watts. Empty = use the global oc.power_limit
  devices[].lock_core_clock  Pin THIS card's core clock in MHz (0 = unlock). Empty = use the global oc.lock_core_clock. A lock pins the clock where an offset shifts the curve and still lets it boost - a card can carry both
  devices[].lock_memory_clock Pin THIS card's memory clock in MHz (0 = unlock). Empty = use the global oc.lock_memory_clock. Only accepted at one of the board's own supported clocks; bzminer says which one you will get if you ask for another
  devices[].fan_speed        This card's fan: a fixed duty ("70") or a temperature curve, same syntax as oc.fan_speed. Empty = use the global one
  device_types.nvidia        Mine on NVIDIA devices (CLI: --nvidia)
  device_types.amd           Mine on AMD devices (CLI: --amd)
  device_types.intel         Mine on Intel devices (CLI: --intel)
  device_types.cpu           Mine on the CPU (CLI: --cpu)
  oc.power_limit             Board power cap in watts. One value for every card, or a per-device list ("160,180"). NVIDIA only (CLI: --oc-power-limit)
  oc.core_clock_offset       Core VF-curve offset in MHz (CLI: --oc-core-clock-offset)
  oc.memory_clock_offset     Memory VF-curve offset in MHz (CLI: --oc-memory-clock-offset)
  oc.lock_core_clock         Pin the core clock in MHz instead of letting boost wander; 0 = unlock (CLI: --oc-lock-core-clock)
  oc.lock_memory_clock       Pin the memory clock in MHz; 0 = unlock (CLI: --oc-lock-memory-clock)
  oc.fan_speed               Fixed duty ("60") or a temperature curve: t:<target>[<min>-<max>] for CORE temp, th:... for HOTSPOT, tm:... for MEMORY, e.g. "t:60[25-75] tm:80[50-100]". With several clauses the one demanding the most cooling wins. The fan moves about a point per second toward the target, so it settles instead of hunting. NVIDIA and AMD; an Intel card reports its fan but exposes no way to drive it (CLI: --oc-fan-speed)
  oc.reset                   Put every card back to driver defaults before applying anything above (CLI: --oc-reset)

Examples:
  bzminer -p stratum+tcp://pool:3333 -w <wallet> --worker rig --pass x -a sha256d
  bzminer -a warthog -w <wallet>.rig -p stratum+tcp://pool:3001 --nvidia
          --oc-power-limit 160 --oc-fan-speed "t:60[25-75] tm:80[50-100]"
  bzminer --pool 1                (mine only pools[1] from config.txt)
  bzminer --dmon                  (device-monitor mode: metrics + web UI, no mining)
  bzminer --oc-reset              (undo overclock settings a killed run left behind)
  bzminer --set http_address=0.0.0.0 --set http_port=8080

Environment:
  Some tuning and diagnostic settings have no flag - e.g. BZ_CPU_THREADS and
  BZ_ZK_THREADS cap worker counts. See docs/env-vars.txt.
```

## Every setting

`config.txt` is JSON, and every field is optional — what follows is the complete
set with its defaults, straight out of `bzminer --config-doc`. Any of them can
also be set on the command line with `--set <path>=<value>`.

The shipped `config.txt` is this file, so you can edit it in place. It starts in
**monitoring mode** (`"pool": []`) — sensors and the dashboard, no mining — so a
fresh install does not start hashing to a placeholder wallet. Put your wallet in
`pools[]` and set `"pool": 0` to mine.

```jsonc
// bzminer configuration (config.txt), JSON. Every field is optional; the
// values below are the defaults. Override any of them on the command line
// with:  --set <path>=<value>
{
  "log": {
    // Log verbosity: network, debug, info, warn, or error (network = the exact pool/stratum wire messages). `trace` is developer detail and is reached only with --internal-log; a config that still says trace is read as network
    "level": "info",
    // Write the log to this file instead of bzminer.worker.log. There is always a file log; this only changes its name/path
    "file": "",
    // Verbosity of the LOG FILE, when it should differ from the console. Same names as `level`. Empty = follow `level`, INCLUDING live changes from the [+/-] keys. Set this to pin the file: e.g. file_level debug with level info keeps the screen readable while still capturing a file worth sending to support (CLI: --logfile-level)
    "file_level": "",
    // What to do with an existing log file: append (default - what bzminer has always done), overwrite (truncate, so the file is only this run), or timestamp (leave it and open bzminer.worker.<date>_<time>.log, keeping every run). A watchdog RESTART never truncates - it would delete the crash that caused it - so overwrite appends from the second worker onwards. CLI: --logfile-mode
    "file_mode": "append",
    // Write the MONITOR screen's sensor table to this file, one block per sample. Only while that screen is being viewed - on any other output there is nothing of its to record. Empty (default) = no monitor log. CLI: --monitor-logfile
    "monitor_file": "",
    // The same three choices as `file_mode`, for the monitor log. Separate so a rolling appended miner log can sit beside a timestamped file per monitoring session. CLI: --monitor-logfile-mode
    "monitor_file_mode": "append",
    // Longest single field kept in a `network`-level wire log line, in characters. Anything longer becomes "<elided, N chars>" and runs of unprintable bytes become "<binary, N bytes>" - the rest of the frame is logged exactly as it went over the wire, so a Pearl share's ~140 KB proof no longer buries the frames either side of it. 0 prints raw frames with nothing removed. CLI: --log-wire-max
    "wire_max": 256
  },
  "network": {
    // Pool / network socket timeout, in milliseconds
    "timeout_ms": 30000,
    // Delay before reconnecting after a disconnect, in milliseconds
    "reconnect_ms": 5000,
    // Route all pool connections through a SOCKS5 proxy: socks5://[user:pass@]host:port (empty = direct). CLI: --proxy
    "proxy": ""
  },
  "plugins": {
    // Auto-download/update plugins from the manifest: auto|on|off (auto = on for the lite build, off for the full build). CLI: --plugin-update
    "update": "auto",
    // Override the plugin manifest URL (empty = built-in GitHub default + bzminer.com fallback). CLI: --plugin-manifest
    "manifest_url": ""
  },
  "safety": {
    // Pause a GPU (mining stops on that card only, auto-resumes when it cools) when its core or hotspot temp exceeds this, in C. 0 = off
    "max_temp_c": 0,
    // Pause a GPU when its board power exceeds this, in W. 0 = off
    "max_power_w": 0,
    // Pause a GPU when its 12VHPWR CONNECTOR current (aggregate of all pins, not per-pin) exceeds this, in A. NVIDIA Blackwell on Windows only. 0 = off
    "max_current_a": 0,
    // Pause a GPU when its WORST SINGLE 12VHPWR pin exceeds this, in A - the connector failure mode the aggregate cannot see. Needs per-pin shunts: ASUS ROG Astral RTX 5080/5090 on Windows only. 0 = off
    "max_pin_current_a": 0,
    // Pause a GPU when the difference between its highest- and lowest-current 12VHPWR pins exceeds this, in A. ASUS ROG Astral RTX 5080/5090 on Windows only. GPUProbe's sample alarm uses 5.5 A. 0 = off
    "max_pin_imbalance_a": 0,
    // How long a card must stay over a limit before it is paused, in seconds (rejects transient spikes)
    "sustain_s": 15,
    // Resume only once the value drops to (limit - this), in the tripped limit's unit (C/W/A) - hysteresis so it does not flap
    "resume_margin": 5,
    // And stayed under the resume point this long, in seconds
    "resume_s": 5
  },
  // Device-monitor mode: telemetry + web UI only, no mining (CLI: --dmon)
  "dmon": false,
  // Serve the monitoring web UI + JSON API (needs the webui plugin; no plugin = no HTTP server)
  "http_enabled": true,
  // Web server bind address ("0.0.0.0" allows LAN access)
  "http_address": "127.0.0.1",
  // Web server TCP port
  "http_port": 4014,
  // Stop this rig announcing itself on UDP 4014 and stop it listening for other bzminers. That announce is what fills the web UI's farm view - every instance on the network, their combined hashrate and power, and a link to each one's dashboard. It carries the rig name, web port, version and this rig's totals, and nothing else; turn it off on a network you do not control. CLI: --disable_udp
  "disable_udp": false,
  // Address the proxy listener (pools[].proxy_port) binds: "0.0.0.0" for every interface, or one interface's address. Rig-wide - a farm has one network. CLI: --proxy_bind
  "proxy_bind": "0.0.0.0",
  // Password every instance connecting to this proxy must present (its --pass). Empty accepts any instance that can reach the port - fine on a private LAN, not elsewhere. CLI: --proxy_pass
  "proxy_pass": "",
  // On startup, write the resolved config to config.effective.json
  "save_effective": false,
  // Run a local stratum server at this fixed share difficulty and mine against it - no pool, no wallet. 0 = off. The CLI form is --benchmark [difficulty]
  "benchmark_diff": 0,
  // How long --benchmark keeps one deterministic job, in ms. A persistent job makes fixed-difficulty share accounting honest for memory-hard kernels. CLI: --benchmark-job-interval
  "benchmark_job_interval_ms": 600000,
  // How often the 'log' output reprints the device/pool table, in ms. Separate from --interval, which is how often the engine PUBLISHES a snapshot (the web UI and metrics want that fast). 0 = print on every snapshot. CLI: --log-table-interval
  "log_table_interval": 30000,
  // How often the 'tui' dashboard redraws, in ms. A keypress redraws immediately regardless. CLI: --tui-interval
  "tui_interval": 400,
  // Columns on the MONITORING sensor table (--dmon, and the monitoring screen of the tui/log outputs). Comma- or space-separated. Takes single columns (memused, memfree, memtotal, util, memutil, core, mem, temp, memtemp, hotspot, power, fan), sensor GROUPS that expand to everything present (temps, clocks, powers, fans, volts, currents, pcie, memory, utilization, performance, all), and any vendor metric key this rig's backends emit (pcie.tx, temp.vr_core, ...). Empty = memused,memfree,util,memutil,core,mem,temp,memtemp,hotspot. Run 'bzminer --list-columns' for every name available on THIS machine. CLI: --metrics
  "metrics": "",
  // Columns on the mining screen's DEVICE table (the hardware box above the algorithm box). Same vocabulary as metrics above. Empty = memfree,memtotal,core,mem,fan,power,temp. Run 'bzminer --list-columns' for the full list. CLI: --device-columns
  "device_columns": "",
  // Columns on the MINING table (the algorithm box: shares, hashrate, pool). Its own vocabulary - id, name, cfg, shares, accepted, rejected, pending, stale, errors, eff, poolhr, hr, avghr, power, temp, fan, core, mem, tbs, status, poolinfo. Empty = id,cfg,tbs,shares,eff,poolhr,hr,status,poolinfo. `tbs` is the MEASURED average time between shares found - what a device delivers, as opposed to the `est. tbs` its difficulty predicts. `poolinfo` is the pool's own column - height, difficulty, est. tbs and latency, one per row. Warthog leaves it out by default, because its status column carries the rig summary and wants the width; name it explicitly to get it there too. Drop it and those four pack into 'status' instead. Drop 'status' as well and the per-device status text goes with them - nothing is left wide enough to hold it. Run 'bzminer --list-columns' for the full list. CLI: --mining-columns
  "mining_columns": "",
  // Collect CPU telemetry (per-core clocks, temperatures, usage, CCD sensors). Set false on a GPU rig that does not want them, or where the reads need a privileged driver - GPU metrics are unaffected. CLI: --cpu-metrics 0
  "cpu_metrics": true,
  // Reserve a private slice of the last-level cache for each CPU mining thread (Linux, root, and a CPU with cache allocation - AMD Zen 2 and newer, or Intel with RDT). RandomX's 2 MiB scratchpads add up to exactly the L3 on a typical rig, so left alone they evict one another; fencing them off measured +4.7% on a Threadripper PRO 9955WX. Machine-wide while mining and restored on exit. CLI: --cache-qos 0
  "cache_qos": true,
  // Read the motherboard's sensor chip directly for fan speeds when no kernel driver publishes them (Linux, x86, and only as root). Many boards - mini-PCs especially - carry an ITE or Nuvoton chip that Linux has no driver for, and then nothing reports a CPU fan at all. Reads only, and never runs where hwmon already has fans. Set false to leave that chip alone. CLI: --superio-fans 0
  "superio_fans": true,
  // Console width in characters. 0 = use the terminal's own width and follow a resize, which is the default and is what you want. Set it to pin the width - for a terminal that misreports, or output being captured with no tty to ask. Pinning it LARGER than the real window makes every row wrap and the dashboard scroll, so measure before setting it. CLI: --tui-width
  "tui_width": 0,
  // Console height in rows. 0 = use the terminal's own height and follow a resize. Independent of tui_width: pinning one leaves the other tracking the window, and where there is no window to ask the unpinned one falls back to 80x25 rather than discarding both. CLI: --tui-height
  "tui_height": 0,
  // Seconds the reported hashrate averages over. A GPU's hash counter advances one batch at a time, so an un-averaged rate reads 0, 0, 0, BURST, 0; this window is what makes it steady. Raise it for a calmer number on a bursty rig, lower it to see an intensity or thread change take effect sooner. Does not affect the 'avg' figure, which is always since start. CLI: --hashrate-window
  "hashrate_window": 30,
  // Behave as if this CPU had no SSE4.1/AES/PCLMUL, nor anything built on them. For measuring a narrower path than the hardware has - here, the portable fallback. CLI: --disable_sse
  "disable_sse": false,
  // Behave as if there were no AVX/AVX2. VAES and AVX-512 go with it, since both need AVX: a CPU with AVX-512 and no AVX2 does not exist, and pretending otherwise would select a path you meant to exclude. This is the switch for measuring the AVX2 path most Zen3-and-older rigs actually run. CLI: --disable_avx2
  "disable_avx2": false,
  // Behave as if there were no AVX-512. AMX goes with it, riding on the same register state. CLI: --disable_avx512
  "disable_avx512": false,
  // Behave as if there were no VAES/VPCLMULQDQ. CLI: --disable_vaes
  "disable_vaes": false,
  // Behave as if there were no AMX. CLI: --disable_amx
  "disable_amx": false,
  // AMD BC-250 only; ignored on every other machine. The board ships deliberately cut down - the shader array locked to 1500 MHz, and 2 of its 8 CPU cores switched off - and neither is a fuse, so both are undone at startup. `gpu` forces the shader clock: measured on a BC-250, warthog's sha side gains 32% and its janus score 12%, while xelis gains nothing, being bound by memory rather than compute. `cpu` enables all 8 cores and takes effect on the NEXT boot, the core mask being read when the OS enumerates CPUs. Both are volatile - a cold power cycle restores the factory setup. CLI: --no-bc250-gpu / --no-bc250-cpu / --bc250-gpu-clock
  "bc250": {"gpu": true, "cpu": true, "gpu_mhz": 1800, "gpu_mv": 0},
  // Allocate with ordinary pages rather than huge ones. Huge pages are meant to help - a large ring is far fewer TLB entries at 2 MB than at 4 KB - but on a working set that already fits the TLB they buy nothing while still needing privileges. This is how a rig owner finds out which case theirs is. CLI: --disable_huge_pages
  "disable_huge_pages": false,
  // Active pool(s) from pools[]: an index (0 or "0"), an array ([0, 2] = multiple pools), or [] for monitoring mode. Omit = all pools (first primary, rest failover). CLI: --pool
  "pool": [],
  // Override the algorithm on the configured pools, whatever wrote them. One algorithm ("warthog") sets pool 0; a list sets one pool each, in order - either JSON (["warthog", "xelis"]) or comma separated ("warthog,xelis"). It is TOP-LEVEL, not a field inside pools[], because a mining OS rewrites the pool block from its own algorithm list - so an algorithm bzminer gained after that front-end shipped cannot be selected in its UI, and an override placed inside pools[] would be overwritten by it. CLI: --force_algo
  "force_algo": "",
  // Which GPUs mine, as a comma- or space-separated list of device numbers and/or pci ids ("0,2", "29:0", "0000:29:00"). Bare entries are an ALLOWLIST - only those cards mine - and entries prefixed '!' a DENYLIST ("!1" = every GPU but device 1). The two cannot be mixed, since "0,!1" has two readings and neither is obviously right. Empty = every GPU mines. This is the one-line form of devices[].enabled and the two are ANDed, so a card disabled in either place does not mine. GPUs only: the CPU is governed by device_types.cpu / cpu_threads. CLI: --devices
  "device_select": "",
  // Mining intensity for every GPU that does not name its own in devices[]: how much work one launch is asked for, in units of 65536 nonces (1-4096). 0 = auto, which lets the algorithm choose. devices[].intensity is addressed by enumeration index and still wins where it is set, so this is the one to use for a whole rig. Shown as i<n> in the mining table's cfg column. CLI: --intensity
  "intensity": 0,
  // How many CPU threads mine. 0 = auto, which holds back one processor on a small machine and two above 8 threads, so the miner's own threads - and yours - are not competing with workers. Set it to the full thread count to mine on everything. Caps every CPU pool the run starts, including an algorithm's own - Pearl's share prover used to size itself from the machine and ignore this. Movable while mining with the console's [t]/[T] keys. CLI: --cpu_threads
  "cpu_threads": 0,
  // Which processors mine, as a list: "0-7,16,18". Takes PRECEDENCE over cpu_threads, since it names the processors outright. Everything not listed is left for the miner's management threads. CLI: --cpu_affinity
  "cpu_affinity": "",
  // Measure each GPU's host<->device PCIe bandwidth once at startup and publish it (pcie.h2d / pcie.d2h). Costs ~1s per GPU; algorithms that stream results off the GPU use it as a hard ceiling. CLI: --bandwidth-test
  "bandwidth_test": true,
  // Per-copy buffer size for that measurement, in MiB. Large enough to measure throughput rather than launch latency
  "bandwidth_test_mb": 64,
  // Dev-fee TEST CYCLE: mine for you this many seconds, then for the algorithm's author, and repeat. 0 = off, the normal schedule. The real one pays the fee as a time debt, so its first slice is five minutes into a run and its second an hour later - correct, and impossible to sit and watch. This makes the whole cycle visible in a minute: the switch, the dev pool's subscribe, the switch back, your own work resuming. A diagnostic, not a tuning knob - the cycle can only ever pay MORE than the declared fee, never less. CLI: --test-dev-fee <user_s>[:<dev_s>]
  "test_dev_fee_user_seconds": 0,
  // How long each dev leg of that cycle runs. 0 = the same as test_dev_fee_user_seconds, so a bare `--test-dev-fee 30` is 30 seconds each way. Raised automatically if the split you asked for would come to less than the declared fee
  "test_dev_fee_dev_seconds": 0,
  "pools": [
    {
      // Pool URL (CLI: -p / --pool / --url; stratum+ssl:// for TLS). Repeat the flag for backups - the first is the primary and the rest are rotated through on disconnection
      "url": "stratum+tcp://us.pearl.herominers.com:1200",
      // Payout wallet address (CLI: -w / --wallet)
      "wallet": "<your wallet address>",
      // Worker / rig name; sent to the pool as wallet.worker (CLI: --worker)
      "worker": "rig",
      // Password (CLI: --pass; note -p is the POOL URL, as in bzminer 1.x)
      "pass": "x",
      // Algorithm (CLI: -a / --algo). Active pools are grouped by algorithm and every algorithm mines AT ONCE, each on its own devices: entries sharing an algorithm are one group, primary first, the rest its failovers
      "algo": "pearl",
      // stratum+ssl: verify the pool's TLS certificate chain + hostname (CLI: --ssl-verify)
      "ssl_verify": false,
      // LEGACY combined login, sent verbatim instead of wallet.worker. Only for a pool that wants something other than that shape - set wallet and worker instead (CLI: -u / --user)
      "user": "",
      // WHICH devices mine this pool's algorithm when the rig runs more than one: device numbers, pci ids, '!' exclusions, and the types cpu, gpu, nvidia, amd, intel. Empty = this algorithm's default share - the GPUs are dealt out among the algorithms that can use them, and the CPU is shared by every algorithm that can. A GPU belongs to one algorithm; two entries of the same algorithm naming DIFFERENT devices are two groups (CLI: --devices<N>)
      "devices": "",
      // How many of the rig's CPU threads this algorithm gets when it shares the CPU with another. 0 = an even share of the rig-wide cpu_threads budget. Placed on unused processors first; counts that add up to more than there are overlap and share (CLI: --cpu_threads<N>)
      "cpu_threads": 0,
      // ...or the processors this algorithm mines on, named outright ("0-7,16"). Wins over cpu_threads and is honoured exactly, overlaps included (CLI: --cpu_affinity<N>)
      "cpu_affinity": "",
      // Serve this algorithm's work to OTHER bzminer instances on this TCP port: this instance keeps the one pool connection, and every bzminer started with -p bzproxy://<this host>:<port> mines the same jobs through it. Their shares go upstream from here, and each instance is shown here as one light-blue row with its hashrate, power and devices. The port is TLS. Each rig keeps its own dev fee and pays it through a tunnel this proxy opens to the algorithm's fee pools, so the rigs need no internet of their own; this proxy signals its own slice so the farm pays in one window. A proxy may mine on its own devices as well, or on none (--devices none). 0 = off. Per algorithm, like devices (CLI: --proxy_port<N>)
      "proxy_port": 0
    }
  ],
  "devices": [
    {
      // Device index - counts EVERY device enumerated, so disabling one does not renumber the others
      "index": 0,
      // Mining intensity: how much work one GPU launch is asked for, in units of 65536 nonces (1-4096). 0 = auto, which is 64. Higher keeps the card busy longer per launch; lower picks up a new job sooner. Shown as i<n> in the mining table's cfg column
      "intensity": 0,
      // Whether to mine on this device. To turn off a whole vendor or the CPU instead, use device_types below
      "enabled": true,
      // WHICH card this entry is, as "domain:bus:device" - written by bzminer on first run, and the field entries are matched on. Preferred over index, which moves when a card is added or removed
      "pci": "",
      // The card's name, written by bzminer so the file says what it means without the rig in front of you. Not read back - pci is the key
      "name": "",
      // The card's architecture ("Ampere", "RDNA3"), written by bzminer. Not read back
      "architecture": "",
      // The board's PCI subsystem pair, "vendor:device" - written by bzminer. The device id alone names the CHIP, which is the same on a reference card and a partner's; this names the board. Not read back
      "pci_subsystem_id": "",
      // This card's core VF-curve offset in MHz. Empty = use the global oc.core_clock_offset
      "core_clock_offset": "",
      // This card's memory VF-curve offset in MHz. Empty = use the global oc.memory_clock_offset
      "memory_clock_offset": "",
      // This card's board power cap in watts. Empty = use the global oc.power_limit
      "power_limit": "",
      // Pin THIS card's core clock in MHz (0 = unlock). Empty = use the global oc.lock_core_clock. A lock pins the clock where an offset shifts the curve and still lets it boost - a card can carry both
      "lock_core_clock": "",
      // Pin THIS card's memory clock in MHz (0 = unlock). Empty = use the global oc.lock_memory_clock. Only accepted at one of the board's own supported clocks; bzminer says which one you will get if you ask for another
      "lock_memory_clock": "",
      // This card's fan: a fixed duty ("70") or a temperature curve, same syntax as oc.fan_speed. Empty = use the global one
      "fan_speed": ""
    }
  ],
  "device_types": {
    // Mine on NVIDIA devices (CLI: --nvidia)
    "nvidia": true,
    // Mine on AMD devices (CLI: --amd)
    "amd": true,
    // Mine on Intel devices (CLI: --intel)
    "intel": true,
    // Mine on the CPU (CLI: --cpu)
    "cpu": true
  },
  "oc": {
    // Board power cap in watts. One value for every card, or a per-device list ("160,180"). NVIDIA only (CLI: --oc-power-limit)
    "power_limit": "",
    // Core VF-curve offset in MHz (CLI: --oc-core-clock-offset)
    "core_clock_offset": "",
    // Memory VF-curve offset in MHz (CLI: --oc-memory-clock-offset)
    "memory_clock_offset": "",
    // Pin the core clock in MHz instead of letting boost wander; 0 = unlock (CLI: --oc-lock-core-clock)
    "lock_core_clock": "",
    // Pin the memory clock in MHz; 0 = unlock (CLI: --oc-lock-memory-clock)
    "lock_memory_clock": "",
    // Fixed duty ("60") or a temperature curve: t:<target>[<min>-<max>] for CORE temp, th:... for HOTSPOT, tm:... for MEMORY, e.g. "t:60[25-75] tm:80[50-100]". With several clauses the one demanding the most cooling wins. The fan moves about a point per second toward the target, so it settles instead of hunting. NVIDIA and AMD; an Intel card reports its fan but exposes no way to drive it (CLI: --oc-fan-speed)
    "fan_speed": "",
    // Put every card back to driver defaults before applying anything above (CLI: --oc-reset)
    "reset": false
  }
}
```


## If something goes wrong

The first forty lines of the log name the build, the config file, the CPU, every
device found and the pool being tried. Start there, then:

| | |
|---|---|
| no GPUs found | update the GPU driver; `--gpu-info` shows what bzminer can see |
| a GPU is listed at startup but never mines | the log says why on its own line — no kernel for that architecture, a driver too old to load one, or not enough VRAM for the algorithm's table |
| pool rejects shares | almost always the wallet — press `u` to see exactly what is being sent, and check it is an address for the coin you are mining |
| overclock does nothing | on Linux these are privileged; run as root. On AMD only the fan is settable at all. The miner reports which knob the driver refused and why |
| dashboard not reachable | it listens on `127.0.0.1` only unless you set `http_address` |
| warthog finds nothing | it needs both a GPU and the CPU; check neither is disabled |
| ergo skips a card | the ~2 GB table has to fit in VRAM alongside everything else on the card |
| randomx is slow | it wants 2080 MB plus 2 MB per thread and huge pages; the startup line says which it got, and a CPU without AES runs it ~4x slower |
| a second algorithm is not mining | one process mines one algorithm — see [mining several at once](#mining-several-algorithms-at-once) |
| a message about MSR tweaks | CPU register tuning is an *optional* speed-up, not a requirement — mining carries on without it. On Windows it needs PawnIO (`scripts/install-pawnio.bat`); being an administrator is not by itself enough. On Linux it needs the `msr` module and access to `/dev/cpu/*/msr`. Run with `--log-level debug` to see which register was refused |
| xelis hashrate lower than an older bzminer | xelis now publishes a rate it measures itself, where the older figure was derived and read high — the same card doing the *same* work reports a smaller number. Compare accepted shares over a fixed period instead; that is the same unit in both versions, and by that measure this build is faster |

More detail: `--log-level debug`, or `--log-level network` for the raw pool
conversation. The log file sits next to the binary and its name is printed at
startup.

**Reporting a bug** — please include your OS, GPU models and driver version, the
algorithm and pool, and the first ~40 lines of the log. For sensor or overclock
problems, `--list-metrics` and `--gpu-info` output help.

## Dev fee

Some algorithms mine to the developer for a small share of the time. Each declares
its own, it is in [the table at the top](#algorithms-and-coins), and it is printed
in the log at startup — so you can see it before committing a rig, not discover it
later. `sha256d`, the open-source example, has none.

Every line about it is tagged `[dev fee]`, and each slice announces both ends:

```
[dev fee] warthog 2% - announced here, and at START and END of every slice
[dev fee] START - mining for the warthog developer on stratum+ssl://ussw.vipor.net:5120 (2.00% -> 13 pools (split evenly))
[dev fee] END - back to your pool stratum+ssl://us.vipor.net:5120 (paid 1.94% of this run so far, aiming for 2.00% -> 13 pools (split evenly))
```

The fee is paid as a **time debt** rather than on a fixed clock: the first slice
lands about five minutes into a run and the next an hour later, and the running
total is what the schedule aims at. Dev-fee shares carry a worker name that says
which rig paid them, and they stay out of your own share table.

The fee is spread over several destinations rather than sent to one, so no single
pool sees all of it. **You do not have to be able to reach them all.** A
destination that will not connect is backed off, and after three failed attempts
it is dropped for the rest of the run — the others carry its share and the fee is
still paid in full. If you block some of these pools at your firewall, the miner
notices once and stops trying; it will not warn you about a pool you blocked on
purpose.

**The fee is owed in time, not in shares.** If *every* destination is
unreachable, the slices still run — the rig keeps hashing and submits nothing for
their duration, so the time is taken whether or not there is anywhere to spend
it. Blocking the pools therefore costs you exactly what paying the fee costs you,
and gains you nothing; unblocking any one of them turns that time back into
shares for the developer instead of into nothing for anybody:

```
[dev fee] none of the 13 destinations could be reached. The fee is owed in time, not in
shares, so each slice is now taken as idle mining: the rig keeps hashing and submits
nothing for its duration. Unblocking any one of them pays it as shares instead.
```

A destination that is merely *down* is a different case and is treated as one: it
is backed off and retried, and until it retires the debt simply accrues and is
paid in full later. Nothing goes idle for a pool that is coming back.

None of this happens on the thread that reads your pool's jobs, so an unreachable
dev pool costs your mining nothing — not a stale job, and not a reconnect.

To watch a whole cycle without waiting out an afternoon:

```bash
./bzminer --test-dev-fee 30 -a xelis -p ... -w ...   # 30 seconds each way
./bzminer --test-dev-fee 30:10 -a xelis -p ... -w ... # 30 for you, 10 for the dev
```

It makes the switch, the dev pool's subscribe, the switch back and your own work
resuming all visible in a minute. It is a diagnostic, not a tuning knob: a split
that would come to *less* than the declared fee has its dev leg raised until it
does not, so the test cycle can only ever pay more.
