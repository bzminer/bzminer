# bzminer

A GPU and CPU cryptocurrency miner for Windows, Linux and macOS.

Mines **Ergo**, **Ravencoin**, **Neurai**, **Neoxa**, **Meowcoin**, **Clore**,
**Pearl**, **VERUS**, **Warthog** and **XELIS**, and ships **sha256d** as the
open-source SDK example.

One binary, no installer. It writes nothing outside its own folder except the log
file and the config you point it at.

**No CUDA toolkit, no OpenCL install.** Every NVIDIA architecture from the GTX
1080 up ships a precompiled kernel, so the driver is all you need. Nothing is
compiled on your machine and nothing is linked against a vendor SDK: CUDA,
OpenCL, AMD's HIP and Apple's Metal are all opened at runtime *if they are
present*, and a backend whose driver is missing is simply left out. An NVIDIA rig
with no OpenCL installed loses nothing, and the build refuses to ship if any of
those libraries creeps back into the import table.

> **Public beta.** It mines, and it has been run against real pools on real
> hardware — but it is new. Watch a rig before you leave it alone with it, and
> please report what you find.

**Contents** — [Download](#download) · [Start mining](#start-mining) ·
[Algorithms](#algorithms) · [Testing without a pool](#testing-without-a-pool) ·
[Pools and failover](#pools-backups-and-failover) · [The console](#the-console) ·
[Overclocking](#overclocking) · [Safety limits](#safety-limits) ·
[Web dashboard](#the-web-dashboard) · [Monitoring only](#monitoring-without-mining) ·
[Several GPUs](#rigs-with-several-gpus--or-several-cpus) ·
[Configuration](#configuration) · [Every option](#every-command-line-option) ·
[Every setting](#every-setting) · [Environment variables](#environment-variables) ·
[Troubleshooting](#if-something-goes-wrong) · [Dev fee](#dev-fee)

## Download

Take the archive for your platform from the [latest release][releases], unpack it,
and check the SHA-256 published beside it.

[releases]: ../../releases

```bash
tar -xzf bzminer_<version>_linux.tar.gz && cd bzminer_<version>_linux
```

One archive per platform. Every algorithm, every hardware monitor, both console
screens and the web dashboard are compiled **into the binary** — there is nothing
else to download and no plugins folder to manage.

| | |
|---|---|
| `bzminer_<ver>_windows.zip` | Windows x64 |
| `bzminer_<ver>_linux.tar.gz` | Linux x64 |
| `bzminer_<ver>_macos.tar.gz` | macOS, Apple Silicon (arm64) |

Inside: the binary, a launcher per algorithm (`start_xelis.sh` / `.bat` and so
on), a fully commented `config.txt`, and `readme.txt`.

## Start mining

Open the launcher for your coin, put your wallet in it, run it. Or go straight to
the command line:

```bash
./bzminer -a xelis -p stratum+ssl://us.vipor.net:5177 -w xel:YOUR_WALLET --worker rig1
```

Then open **<http://127.0.0.1:4014/>** for the dashboard.

The three flags that matter: `-a` the algorithm, `-p` the pool URL, `-w` your
wallet. `--worker` names the rig at the pool and is optional.

macOS blocks unsigned downloads outright — if it refuses to open, clear the
quarantine flag once, in the unpacked folder:

```bash
xattr -dr com.apple.quarantine .
```

### Coming from bzminer 1.x

Old start scripts run as they are. `-p` is the **pool URL**, as it always was (the
password is `--pass`), and the underscore spellings of the tuning flags —
`--oc_power_limit`, `--oc_fan_speed`, `--cpu_threads`, `--warthog_verus_hr_target`
— are all accepted and mapped to their modern equivalents.

One flag changed meaning: **`-o` now selects the console screen** (it is short for
`--output`), because `-O` for that and `-o` for a pool was backwards. A URL passed
to `-o` is still taken as a pool, with a warning saying so, rather than silently
leaving you with no pool at all.

Two 1.x options describe machinery this version no longer has:
`--warthog_max_ram_gb` and `--warthog_target_cpu_pressure`. The GPU→CPU hand-off
is now a small cache-local ring rather than a RAM buffer you size, and the balance
between the two stages is found by a tuner rather than aimed at a pressure number.
bzminer accepts both options and says what governs that behaviour now, instead of
quietly mapping them onto something that merely looks similar.

Anything it does not recognise is reported rather than ignored, so a flag that has
been renamed shows up as a warning at startup instead of silently doing nothing.

## Algorithms

| algorithm | coin | dev fee | notes |
|---|---|---|---|
| `ergo` | Ergo | 1% | Autolykos v2. GPU only, and memory-hard: it builds a table of N 32-byte elements (about 2 GB) in VRAM, so a card needs the room for it. N grows with block height, and the table is rebuilt when it changes. Wallet addresses start with '9'. |
| `kawpow` | Ravencoin (`rvn`)<br>Neurai (`xna`)<br>Neoxa (`neoxa`)<br>Meowcoin (`meowcoin`)<br>Clore (`clore`) | 1% | KawPow on NVIDIA CUDA, AMD OpenCL, and Apple Metal GPUs. NVIDIA specializes the three-block random program through the driver's PTX JIT, with precompiled cubins as its fallback; AMD uses offline-native gfx code objects; Apple Silicon uses a portable offline metallib. No CUDA toolkit, NVRTC, ROCm compiler, OpenCL C source, Metal source, or external GPU compiler is required on the mining rig; the AMD and Apple paths do no runtime compilation. Answers to `kawpow` and to each KawPow coin by name (rvn, xna, neoxa, meowcoin, clore), which is also what selects the dev-fee destination. A wallet address for the coin being mined, optionally followed by .worker. |
| `pearl` | Pearl | 1% | a zk proof-of-work: each share is a STARK proof, so it is far heavier per hash than a normal algorithm. Also mines SOLO - point -p at your node's RPC URL instead of a pool. Wallet addresses start with 'prl1'. |
| `sha256d` | — | none | The open-source SDK example: a full algorithm plugin (CPU+GPU, pool stratum, bench job source). Copy plugins/public/sha256d as a template for your own. |
| `verus` | VERUS | 1% | VerusHash v2.2 CPU mining. A Verus R-address, optionally followed by .worker. |
| `warthog` | Warthog | 2% | janushash - the GPU filters sha256t and the CPU runs verushash, on the same nonces. Needs BOTH a GPU and the CPU; one without the other finds nothing. Wallet addresses are 48 hex characters. |
| `xelis` | XELIS | 1% | xelhash. Wallet addresses start with 'xel:'. |

The dev fee is declared by each algorithm, printed in the log at startup, and
listed here straight from the shipped binary — so what you read is what you run.

Which hardware each one uses:

| algorithm | GPU | CPU |
|---|---|---|
| `ergo` | NVIDIA, AMD, Intel, Apple | — |
| `xelis` | NVIDIA, AMD, Intel, Apple | yes |
| `pearl` | NVIDIA, AMD, Intel | yes |
| `warthog` | NVIDIA, AMD, Intel, Apple | **required, with a GPU** |
| `verus` | — | yes |
| `sha256d` | NVIDIA, AMD, Intel, Apple | yes |

"Apple" means an Apple Silicon GPU through Metal, on macOS. Pearl is the one
exception: its GPU kernels need integer matrix hardware that Metal does not
expose, so on a Mac it mines on the CPU.

### Mining each one

**XELIS** — CPU and GPU, both at once on the same rig.

```bash
./bzminer -a xelis -p stratum+ssl://us.vipor.net:5177 -w xel:YOUR_WALLET --worker rig1
```

Pools: `stratum+ssl://us.vipor.net:5177` ·
`stratum+ssl://us.xelis.herominers.com:1225` · solo to your own node with
`ws://127.0.0.1:8080`.

**Ergo** — Autolykos v2, **GPU only**, and memory-hard: it builds a table of about
2 GB in VRAM before it can mine, so a card needs the room for it and there is a
pause at startup (and again whenever the table is rebuilt) while it is filled.
Cards with less VRAM than the table needs are reported and skipped rather than
quietly producing nothing.

```bash
./bzminer -a ergo -p stratum+tcp://pool.woolypooly.com:3100 -w 9YOUR_ERGO_WALLET --worker rig1
```

Pools: `stratum+tcp://pool.woolypooly.com:3100`. Wallets start with `9`.

**Pearl** — a zk proof-of-work: every share is a STARK proof, so it is far heavier
per hash than an ordinary algorithm and the hashrate numbers look small by
comparison. Normal.

```bash
./bzminer -a pearl -p stratum+tcp://us.pearl.herominers.com:1200 -w prl1YOUR_WALLET --worker rig1
```

Pools: `stratum+tcp://us.pearl.herominers.com:1200` ·
`stratum+tcp://prl-us.kryptex.network:7048` ·
`stratum+tcp://pearl-cpu-eu1.luckypool.io:3370`.

Pearl also mines **solo** — point `-p` at your own node's RPC URL instead of a
pool.

**Warthog** — needs a GPU *and* the CPU together. The GPU filters sha256t and the
CPU runs VerusHash over the same nonces; neither half finds anything alone, so
disabling either one stops the rig. The first two minutes show `calibrating
verus` while it works out how hard to drive the filter — a near-zero rate during
that is expected, not a fault.

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

**VERUS** — VerusHash v2.2, **CPU only**. No GPU is used, so a rig can mine this
on the processor while the cards do something else.

```bash
./bzminer -a verus -p stratum+ssl://bzdev.vipor.net:5140 -w YOUR_VERUS_ADDRESS.rig1
```

The wallet is a Verus R-address, optionally with `.worker` appended — that form
carries the worker name, so `--worker` is not needed.

**SHA-256d** is the open-source example that ships with the plugin SDK. It mines,
but SHA-256d is ASIC territory — it is there to be read and copied, not to earn.

```bash
./bzminer -a sha256d -p stratum+tcp://your-pool:3333 -w YOUR_WALLET
```

## Testing without a pool

Two ways to make the miner work without pointing it at anyone, both useful for
different questions. Neither needs a wallet, an internet connection or an account.

**`--bench`** — mine a job made up locally. No sockets at all. This answers *"does
this machine mine this algorithm"*: it exercises the kernels and the CPU path and
nothing else.

```bash
./bzminer --bench -a ergo         # also: xelis, pearl, warthog, verus, sha256d
```

**`--benchmark [difficulty]`** — bzminer starts a **real stratum server inside
itself**, on a local port, and then connects to it over TCP like any other pool.
The whole client path runs: subscribe, authorize, `mining.notify`, share submit,
the pool's verdict, the a/r/s counters and the pool-hashrate column. It is the
honest way to measure a card, because a share only counts once the server has
accepted it.

```bash
./bzminer --benchmark -a warthog              # default difficulty 1000
./bzminer --benchmark 25000000000000 -a pearl # fixed difficulty 25T
```

Pick the difficulty deliberately. Too low and a fast rig floods the local server
with submissions and measures the plumbing instead of the kernel; too high and you
will wait a long time for enough shares to mean anything. As a rule aim for a
share every few seconds: Pearl on a big GPU wants `25000000000000` (25T), xelis
wants something in the thousands.

The share count is what to trust. `shares × difficulty ÷ seconds` is a rate the
miner cannot flatter — compare it against the displayed hashrate, and if the two
disagree for long, one of them is lying.

Each benchmark job is held for ten minutes by default so that a memory-hard
kernel is not thrown away mid-batch every time the job refreshes, which would
make fixed-difficulty share accounting read low. `--benchmark-job-interval <ms>`
changes that; set it to something short, say `4000`, if what you actually want
to exercise is how the miner copes with jobs going stale.

Both modes honour everything else: `--nvidia` to test one vendor, `--oc-*` to
measure at a given power limit, `-o log` for a scrolling log instead of the table.

## Pools, backups and failover

List as many pools as you like. The **first is primary and the rest are backups**:
bzminer connects to the primary, and on a disconnect rotates through the others
until one answers, then keeps mining there.

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

Backups must be the **same algorithm** as the primary. `"pool": 0` picks one entry
out of the list; `"pool": [0, 2]` runs a chosen subset; omit it and you get all of
them.

Other pool options: `stratum+ssl://` for TLS (add `--ssl-verify` to check the
certificate), `--pass <password>` where a pool wants one — note `-p` is the pool
URL, not the password — and `--proxy socks5://host:port` to route every pool
connection through a SOCKS5 proxy.

## The console

Three screens. Press **`o`** to cycle, or start on one with `-o <name>`
(`--output` in full).

| screen | what it shows | when |
|---|---|---|
| `tui` | boxed dashboard — devices, shares, pool state, log pane. In `--dmon` it drops the mining boxes and shows a sensors table instead | **the default whenever there is a terminal**, mining or monitoring |
| `monitor` | the same sensors, `nvidia-smi dmon` style: one dense table, no boxes | `--dmon`, when you want it plain |
| `log` | plain scrolling log, with the device table reprinted every 30s | always; the default when output is redirected to a file |

If you pipe or redirect bzminer's output, it stays on `log` on purpose — a
full-screen dashboard sent to a file is just a file full of cursor codes. That is
what mining OSes get.

How often each one refreshes is yours to set. These are separate from
`--interval`, which is how often the engine *publishes* data (the web dashboard
and the metric history want that fast) rather than how often a screen is redrawn:

```bash
--log-table-interval 30000   # 'log': reprint the device table every 30s (0 = every snapshot)
--tui-interval 400           # 'tui': repaint the dashboard every 400ms
--interval 1000              # the underlying data rate for all of them, and the web UI
--hashrate-window 30         # seconds the reported hashrate averages over
```

`--hashrate-window` is a different thing from the three above it: they decide how
often a number is *drawn*, it decides what the number *is*. A GPU's hash counter
advances one batch at a time, so an un-averaged rate reads 0, 0, 0, BURST, 0 - the
window is what makes it a rate. Raise it for a calmer number on a bursty rig, lower
it to see an intensity or thread change take effect sooner. The `avg` figure is
unaffected; that is always since start.

The mining table's **`cfg`** column says how each device is *configured*, as
opposed to what it is doing: `i64` is a GPU at intensity 64, `i0` a GPU on auto,
and `t16` a CPU mining on 16 threads. It is the quickest way to confirm a setting
actually took.

Under the summary row's **`pool hr`** is that rate as a percentage of the miner's
own: **100%** means the pool is crediting exactly what your rig reports, below
means it is finding fewer shares than its hashrate implies, above means luck has
run your way. It is the number to watch if a miner looks fast but pays badly.
Expect it to swing over minutes and settle over hours - it is computed from
accepted shares, so a short run says very little.

### Hotkeys

The same on every screen. `h` prints this list in the miner, with the current
setting beside each key that has one.

| key | |
|---|---|
| `o` | next screen |
| `d` | device detail page |
| `c` | open the command line |
| `h` | this list |
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
| `w` / `W` | board power cap, up / down |

The tuning keys move **one card**, not the rig. `n` picks which, and its row is
marked `>` in the device tables. A lock and an offset are separate settings and a
card can carry both, which is why each has its own key. Every step starts from
what the card is actually set to, read back per device, so a rig started with an
offset in `config.txt` does not jump on the first press.

The device page adds `a` (all metrics), `/` (filter), arrows and `Esc`.

On the `tui` dashboard the log pane **scrolls**: the mouse wheel, `Up`/`Down`,
`PageUp`/`PageDown`, `Home` and `End`. While you are scrolled up the pane holds
still — new lines keep arriving behind it rather than dragging the text out from
under you — and the bar says `held N up  [End] live` so a frozen pane is never
mistaken for a stalled miner. `End` returns to following the newest line.

**Selecting and copying text keeps working too.** Scrolling and selecting are
separate actions, and where the terminal can serve both bzminer never takes the
mouse: it asks the terminal to turn wheel ticks into key presses instead. That
covers every Linux and macOS terminal, and on Windows every modern one — Windows
Terminal, VS Code, ConEmu. Drag out a log line to paste into a bug report while the
wheel still scrolls the pane.

The old Windows console window (`conhost`, still the default on Windows 10) is the
exception. It cannot translate wheel ticks, and it hands a program the wheel only
with its own quick-edit selection switched off, so there the two really are
exclusive — and the wheel wins. Set `BZ_MOUSE=0` to keep selection instead, or run
bzminer under Windows Terminal and have both. (`BZ_MOUSE=1` goes the other way and
takes the mouse everywhere, which adds click events at the cost of drag-selection —
you should not need it.)

### Commands

Press `c`, type, press enter.

| command | what it does |
|---|---|
| `help` | list the commands |
| `pause` / `resume` | stop and restart mining without exiting |
| `oc ...` | overclock — see below |
| `intensity <n>` | mining intensity; `0` is auto |
| `level <name>` | log level: `network`, `debug`, `info`, `warn`, `error` |
| `output <name>` | switch screen by name |
| `quit` | exit |

`level network` prints the raw stratum traffic, which is the fastest way to settle
an argument with a pool about what was actually sent.

## Overclocking

NVIDIA cards. Three routes — a start script, the console, or the web dashboard —
all going through **one implementation**, so they cannot disagree with each other.
Every refusal tells you *why* ("needs root/administrator", "the driver would not
expose the clock-offset API for this device") instead of failing quietly.

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

A worked example — mine xelis with the cards capped at 160 W, a mild core offset,
memory pushed harder, and the fans held at a target temperature:

```bash
./bzminer -a xelis -p stratum+ssl://us.vipor.net:5177 -w xel:YOUR_WALLET \
  --oc-power-limit 160 --oc-core-clock-offset 150 --oc-memory-clock-offset 1000 \
  --oc-fan-speed "t:60[25-75] tm:80[50-100]"
```

Each takes one value for every card, or one per card: `--oc-power-limit 160,180`
sets 160 W on the first and 180 W on the second. A short list leaves the remaining
cards untouched.

**Fan control** takes a fixed duty, or a temperature target with a range to move
inside:

```
--oc-fan-speed 60                        just run at 60%
--oc-fan-speed "t:60[25-75]"             hold the CORE at 60C, using 25-75% fan
--oc-fan-speed "tm:80[50-100]"           the same, aimed at MEMORY temperature
--oc-fan-speed "t:60[25-75] tm:80[50-100]"   both — whichever wants more fan wins
```

That last form is the one worth knowing: a card can sit at a perfectly comfortable
core temperature while its memory cooks, and the second clause is what catches it.
The fan moves gradually toward the target rather than jumping, and stays inside
the range you gave.

A card keeps a fan duty or a locked clock until something clears it, so a run that
is **killed** rather than closed leaves them behind. `bzminer --oc-reset` on its
own undoes them.

Coming from bzminer 1.x? The underscore spellings (`--oc_power_limit`,
`--oc_fan_speed`, …) all still work, so an old start script needs no editing.

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
percentage or `auto`. A device is named by its number as shown in the table, or
`all`.

A memory lock is only honoured at one of the clocks the board publishes as
lockable — ask for anything else and the driver quietly runs the nearest one it
does support, so bzminer reads the clock back and tells you what the card is
actually doing.

`fan=auto` is not `fan=0`. Zero means *stop the fan*.

On Linux, clock, power and fan changes need root. bzminer restores whatever it
changed when it exits.

## Safety limits

Off by default. Turn on what you want and bzminer will park a card that misbehaves
and bring it back when it recovers — **only the offending card stops, the rest of
the rig keeps mining**.

```bash
--set safety.max_temp_c=85      # pause a GPU over 85 C
--set safety.max_power_w=600    # ...or over 600 W board power
--set safety.sustain_s=15       # only after 15s over the limit, not on a spike
--set safety.resume_margin=5    # resume once it is 5 back under
```

Set the temperature limit first. It works on every card, and hardware in trouble —
a failing 12VHPWR connector included — gets hot.

There are 12VHPWR current limits too (`safety.max_current_a`, and per-pin
`safety.max_pin_current_a` / `safety.max_pin_imbalance_a`), but they need sensors
only some boards carry. `config.txt` says exactly which.

## The web dashboard

Served at **<http://127.0.0.1:4014/>**, and it is the same data the console shows:
per-device hashrate, shares, pool state, every sensor, and the overclock controls.

```bash
--set http_port=8080            # different port
--set http_address=0.0.0.0      # reachable from the rest of the LAN
--set http_enabled=false        # off
```

It binds to localhost by default. `0.0.0.0` exposes it to anything that can reach
the machine — there is no password on it, so put it behind something you trust.

JSON, for scripting or your own dashboard:

| endpoint | |
|---|---|
| `/api/snapshot` | everything: devices, pools, hashrate, shares |
| `/api/stream` | the same, pushed as it changes |
| `/api/gpus`, `/api/metrics`, `/api/topology`, `/api/ram`, `/api/storage` | hardware detail |
| `/api/oc` | read and apply overclocks |
| `/status` | the shape other mining dashboards expect |
| `/hive_status` | HiveOS |

## Monitoring without mining

```bash
./bzminer --dmon
```

Telemetry and the web dashboard, no mining. Useful for watching a rig that is
busy with something else, or for checking sensors before you commit a card to a
job.

`--metrics` picks the columns, on both monitoring screens (`tui` and `monitor`):

```bash
./bzminer --dmon --metrics temps,powers,clocks
./bzminer --metrics ?            # every group and live metric on this machine
```

A group expands to every sensor of that kind the machine actually reports, so
`temps` on a card with per-die sensors gives you all of them without naming one.
Columns nothing reports are dropped rather than filled with dashes.

Groups are `temps`, `clocks`, `powers`, `volts`, `currents`, `pcie`,
`utilization`, `memory`, `timings`, `counters`, `performance`, `all`.

One-shot inspection, no mining, no dashboard:

```bash
./bzminer --gpu-info       # NVIDIA devices as CUDA sees them
./bzminer --cpu-info       # CPU topology and live metrics
./bzminer --ram-info       # memory modules and SMBus temperatures
./bzminer --list-metrics   # every sensor this machine exposes
./bzminer --list-algos     # algorithms in this build, with their dev fees
```

## Rigs with several GPUs — or several CPUs

Every device mines by default — every GPU **and** the CPU. Two ways to narrow it.

**By type**, on the command line:

```
--nvidia --amd --intel --cpu
```

Naming any of them makes those the *only* ones that mine: `--nvidia` mines the
NVIDIA cards and nothing else, `--nvidia --cpu` adds the CPU back. Give a value
instead to change one type and leave the rest alone — `--amd 0` stops AMD mining
without touching anything else. The same lives under `device_types` in
`config.txt`.

**By single device**, for when four identical cards are fine and the fifth is
doing something else. Devices are addressed by the number in the table
(`0`, `1`, ...) — which is also what the `oc` command takes:

```json
{
  "devices": [
    { "index": 0, "enabled": true,  "intensity": 0 },
    { "index": 1, "enabled": false },
    { "index": 2, "enabled": true,  "intensity": 20 }
  ]
}
```

**Intensity** is how much work one GPU launch is asked for, in units of 65536
nonces, from `1` to `4096`. `0` is auto and means `64` — a ~4M-nonce launch, which
is what every card used before this was configurable, so leaving it alone changes
nothing. Raise it to keep a card busy for longer per launch; lower it to pick up a
new job sooner, to keep a display responsive, or to shorten a launch that is
tripping a driver watchdog. The current value is in the table's `cfg` column, and
`intensity <n>` on the command line (`c`) changes every device live, with `0` to
put them back on their configured values.

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
Both are also config settings (`cpu_threads`, `cpu_affinity`).

If you want every thread mining, ask for it: `--cpu_threads <all of them>`.

Multi-socket machines are detected properly: bzminer reads the real topology —
packages, cores, threads, NUMA nodes and cache layout — and prints it at startup.
CPU mining threads are placed with that layout in mind rather than scattered, so a
dual-socket box does not spend its time moving work between NUMA nodes.

Warthog goes further and plans the whole rig at once, because its two halves feed
each other: it groups CPU workers by shared L3 cache and measures each GPU's PCIe
bandwidth so it knows an x16 slot from an x1 riser, then sizes the candidate flow
per card. `--no-bandwidth-test` skips that measurement if you would rather save
the second per GPU at startup.

## Configuration

Three ways to say the same thing. The command line always wins:

```bash
./bzminer -a xelis -p stratum+ssl://host:5177 -w <wallet>   # pool flags
./bzminer --set http_port=8080                              # any setting, by path
./bzminer --config myrig.txt                                # a config file
```

A `config.txt` next to the binary is picked up automatically, and the log says
which file it read. `bzminer --config-doc` prints a fully commented template of
every setting — generated from the binary itself, so it can never describe an
option your build does not have. `--save-config` writes the resolved settings,
after all three layers, to `config.effective.json`.

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
  --no-watchdog           Run the worker in-process (no supervisor; debugging)
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
  --bandwidth-test [arg]  Measure each GPU's host<->device PCIe bandwidth at startup and
                          publish it (pcie.h2d / pcie.d2h). on | off | <MiB buffer size>;
                          default on, 64 MiB. Algorithms that stream results off the GPU
                          (e.g. warthog) use it to tell an x16 slot from a x1 riser
  --no-bandwidth-test     Skip it (saves ~1s per GPU at startup)
                          To turn off ONE device rather than a whole type, set
                          devices[].enabled = false in config.txt (index counts every
                          device enumerated, so disabling one does not renumber the rest)

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
  -p, --pool, --url <url> Add a pool (starts a new pool entry). Repeat for backups:
                          the first is the primary, the rest are rotated through
                          on disconnection
  -w, --wallet <addr>     Payout wallet for the current pool
  --worker <name>         Worker/rig name (login sent to the pool: wallet.worker)
  -u, --user <user>       LEGACY combined login (use --wallet/--worker instead)
  --pass <pass>           Password for the current pool (NOTE: -p is the pool URL)
  -a, --algo <algo>       Algorithm for the current pool
  --ssl-verify            Verify the current pool's TLS cert (stratum+ssl; default off)
  --proxy <url>           Route all pools through a SOCKS5 proxy: socks5://[user:pass@]host:port
  --plugin-update <mode>  Auto-download/update plugins: on | off | auto (lite=on, full=off)
  --plugin-manifest <url> Override the plugin manifest URL (beta channel / testing)
  --pool <index|[0,2]>    ...or, given a number instead of a URL, which configured
                          pool(s) to activate; [] = monitoring mode
  --force_algo <algos>    Override the pools' algorithms, whatever wrote them: one
                          name for pool 0, or a comma-separated list for one pool
                          each. Also a top-level config.txt option, which is where a
                          mining OS whose UI lacks the algorithm wants it
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
  save_effective             On startup, write the resolved config to config.effective.json
  benchmark_diff             Run a local stratum server at this fixed share difficulty and mine against it - no pool, no wallet. 0 = off. The CLI form is --benchmark [difficulty]
  benchmark_job_interval_ms  How long --benchmark keeps one deterministic job, in ms. A persistent job makes fixed-difficulty share accounting honest for memory-hard kernels. CLI: --benchmark-job-interval
  log_table_interval         How often the 'log' output reprints the device/pool table, in ms. Separate from --interval, which is how often the engine PUBLISHES a snapshot (the web UI and metrics want that fast). 0 = print on every snapshot. CLI: --log-table-interval
  tui_interval               How often the 'tui' dashboard redraws, in ms. A keypress redraws immediately regardless. CLI: --tui-interval
  metrics                    Columns on the MONITORING sensor table (--dmon, and the monitoring screen of the tui/log outputs). Comma- or space-separated. Takes single columns (memused, memfree, memtotal, util, memutil, core, mem, temp, memtemp, hotspot, power, fan), sensor GROUPS that expand to everything present (temps, clocks, powers, fans, volts, currents, pcie, memory, utilization, performance, all), and any vendor metric key this rig's backends emit (pcie.tx, temp.vr_core, ...). Empty = memused,memfree,util,memutil,core,mem,temp,memtemp,hotspot. Run 'bzminer --list-columns' for every name available on THIS machine. CLI: --metrics
  device_columns             Columns on the mining screen's DEVICE table (the hardware box above the algorithm box). Same vocabulary as metrics above. Empty = memfree,memtotal,core,mem,fan,power,temp. Run 'bzminer --list-columns' for the full list. CLI: --device-columns
  mining_columns             Columns on the MINING table (the algorithm box: shares, hashrate, pool). Its own vocabulary - id, name, cfg, shares, accepted, rejected, pending, stale, errors, eff, poolhr, hr, avghr, power, temp, fan, core, mem, tbs, status, poolinfo. Empty = id,cfg,tbs,shares,eff,poolhr,hr,status,poolinfo. `tbs` is the MEASURED average time between shares found - what a device delivers, as opposed to the `est. tbs` its difficulty predicts. `poolinfo` is the pool's own column - height, difficulty, est. tbs and latency, one per row. Warthog leaves it out by default, because its status column carries the rig summary and wants the width; name it explicitly to get it there too. Drop it and those four pack into 'status' instead. Drop 'status' as well and the per-device status text goes with them - nothing is left wide enough to hold it. Run 'bzminer --list-columns' for the full list. CLI: --mining-columns
  cpu_metrics                Collect CPU telemetry (per-core clocks, temperatures, usage, CCD sensors). Set false on a GPU rig that does not want them, or where the reads need a privileged driver - GPU metrics are unaffected. CLI: --cpu-metrics 0
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
  intensity                  Mining intensity for every GPU that does not name its own in devices[]: how much work one launch is asked for, in units of 65536 nonces (1-4096). 0 = auto, which lets the algorithm choose. devices[].intensity is addressed by enumeration index and still wins where it is set, so this is the one to use for a whole rig. Shown as i<n> in the mining table's cfg column. CLI: --intensity
  cpu_threads                How many CPU threads mine. 0 = auto, which holds back one processor on a small machine and two above 8 threads, so the miner's own threads - and yours - are not competing with workers. Set it to the full thread count to mine on everything. Caps every CPU pool the run starts, including an algorithm's own - Pearl's share prover used to size itself from the machine and ignore this. Movable while mining with the console's [t]/[T] keys. CLI: --cpu_threads
  cpu_affinity               Which processors mine, as a list: "0-7,16,18". Takes PRECEDENCE over cpu_threads, since it names the processors outright. Everything not listed is left for the miner's management threads. CLI: --cpu_affinity
  bandwidth_test             Measure each GPU's host<->device PCIe bandwidth once at startup and publish it (pcie.h2d / pcie.d2h). Costs ~1s per GPU; algorithms that stream results off the GPU use it as a hard ceiling. CLI: --bandwidth-test
  bandwidth_test_mb          Per-copy buffer size for that measurement, in MiB. Large enough to measure throughput rather than launch latency
  pools[].url                Pool URL (CLI: -p / --pool / --url; stratum+ssl:// for TLS). Repeat the flag for backups - the first is the primary and the rest are rotated through on disconnection
  pools[].wallet             Payout wallet address (CLI: -w / --wallet)
  pools[].worker             Worker / rig name; sent to the pool as wallet.worker (CLI: --worker)
  pools[].pass               Password (CLI: --pass; note -p is the POOL URL, as in bzminer 1.x)
  pools[].algo               Algorithm (CLI: -a / --algo)
  pools[].ssl_verify         stratum+ssl: verify the pool's TLS certificate chain + hostname (CLI: --ssl-verify)
  pools[].user               LEGACY combined login, sent verbatim instead of wallet.worker. Only for a pool that wants something other than that shape - set wallet and worker instead (CLI: -u / --user)
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
      // Algorithm (CLI: -a / --algo)
      "algo": "pearl",
      // stratum+ssl: verify the pool's TLS certificate chain + hostname (CLI: --ssl-verify)
      "ssl_verify": false,
      // LEGACY combined login, sent verbatim instead of wallet.worker. Only for a pool that wants something other than that shape - set wallet and worker instead (CLI: -u / --user)
      "user": ""
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

## Environment variables

A few knobs are deliberately environment-only: they are tuning and diagnostic
settings rather than configuration, and giving each a flag would bury the options
that matter. These are the ones worth knowing.

| variable | what it does |
|---|---|
| `BZ_CPU_THREADS=<n>` | Cap CPU mining workers, for scripts that would rather set an environment variable than pass `--cpu_threads`. It caps what the thread plan already chose, so it cannot exceed it |
| `BZ_XELIS_THREADS=<n>` | The GPU thread budget for xelis. It is chosen from the device — 128 threads per compute unit on Ada and older (floor 8192, ceiling 16384), 256 per unit and no ceiling on Blackwell, which needs the larger launch. Setting this by hand is for tuning experiments |
| `BZ_XELIS_BLOCK=32\|64\|128\|256` | The xelis CUDA launch block. Default 64, or 256 on a recognised GA100 mining card |
| `BZ_XELIS_STREAMS=1\|2` | Force one launch slot, or force the two-slot schedule measured on GA100, onto another NVIDIA card. A/B testing knob |
| `BZ_ZK_THREADS=<n>` | Worker cap for Pearl's ZK prover |
| `BZ_NO_CPU=1`, `BZ_NO_CUDA=1`, `BZ_NO_OPENCL=1` | Skip a whole compute backend. Blunter than `--nvidia`/`--amd`/`--cpu`, and useful for isolating one device while benchmarking. `BZ_NO_OPENCL=1` also stops the OpenCL loader being opened at all |
| `BZ_MOUSE=1` | Take the mouse everywhere, by full mouse reporting. Adds click events, but the terminal loses click-and-drag text selection while it is on. The wheel already scrolls without it |
| `BZ_MOUSE=0` | Never take the mouse. Only matters in the old Windows console window (`conhost`), which cannot give a program the wheel and keep its own text selection: this picks selection, and the wheel stops scrolling the pane |
| `BZ_WARTHOG_SHAVERUS=<H/s>` | Pin the candidate rate each Warthog GPU is asked for instead of letting the tuner find it. This is the modern equivalent of bzminer 1.x's `--warthog_verus_hr_target`, which maps onto it automatically |
| `BZ_WARTHOG_SHA_QUALITY=<pct>` | Nudge Warthog's balance point without pinning it |
| `BZ_WARTHOG_AFFINITY=auto\|group\|none\|pu` | How Warthog's verus workers are pinned to cores |
| `BZ_WARTHOG_INCLUDE_INTEGRATED=1` | Mine on an integrated GPU that shares system memory beside a discrete card. Off by default because on a Ryzen APU it took nearly half the CPU's verus rate to contribute almost no sha |
| `BZ_ERGO_INCLUDE_INTEGRATED=1` | The same for Ergo, where an integrated GPU also cannot size the 2 GB table safely |
| `BZ_PEARL_HIP=0\|opencl\|force` | Pearl's AMD path: `0`/`opencl` force portable OpenCL, `force` fails rather than silently falling back |
| `BZ_POOL_PROBE_S=<n>` | Seconds between probes of a failed primary pool (minimum 5) |

The full list, including the diagnostic ones, is in `docs/env-vars.txt` in the
source repository.

## What is in the binary

Algorithms, hardware monitors, the console screens and the web dashboard are all
plugins behind one C ABI, and this release has every one of them compiled in:

- `ergo` — Autolykos v2. GPU only, and memory-hard: it builds a table of N 32-byte elements (about 2 GB) in VRAM, so a card needs the room for it. N grows with block height, and the table is rebuilt when it changes.
- `kawpow` — KawPow on NVIDIA CUDA, AMD OpenCL, and Apple Metal GPUs. NVIDIA specializes the three-block random program through the driver's PTX JIT, with precompiled cubins as its fallback; AMD uses offline-native gfx code objects; Apple Silicon uses a portable offline metallib. No CUDA toolkit, NVRTC, ROCm compiler, OpenCL C source, Metal source, or external GPU compiler is required on the mining rig; the AMD and Apple paths do no runtime compilation. Answers to `kawpow` and to each KawPow coin by name (rvn, xna, neoxa, meowcoin, clore), which is also what selects the dev-fee destination.
- `mon_amd` — AMD GPU sensors, via ADL and sysfs.
- `mon_cpu` — CPU sensors: per-core temperature, frequency, package power.
- `mon_intel` — Intel GPU sensors, via IGCL and sysfs.
- `mon_nvidia` — NVIDIA sensors and overclocking, via NVML and NvAPI.
- `mon_pawnio` — the PawnIO driver path for AMD Ryzen SMU tables.
- `mon_priv_cpu` — CPU sensors that need elevated access (MSRs).
- `out_device` — the device inspector - every GPU and the CPU, their identity, firmware and every sensor, on a scrollable page.
- `out_log` — the plain scrolling console screen ('log'), with the device table reprinted on its own interval. The automatic choice when output is redirected to a file.
- `out_monitor` — the dense sensors console screen ('monitor'), nvidia-smi dmon style: one in-place table whose columns are chosen with --metrics. The default screen in --dmon.
- `out_tui` — the interactive console - live table, hotkeys, and the 'c' command menu.
- `pearl` — a zk proof-of-work: each share is a STARK proof, so it is far heavier per hash than a normal algorithm. Also mines SOLO - point -p at your node's RPC URL instead of a pool.
- `sha256d` — The open-source SDK example: a full algorithm plugin (CPU+GPU, pool stratum, bench job source). Copy plugins/public/sha256d as a template for your own.
- `verus` — VerusHash v2.2 CPU mining.
- `warthog` — janushash - the GPU filters sha256t and the CPU runs verushash, on the same nonces. Needs BOTH a GPU and the CPU; one without the other finds nothing.
- `webui` — the web dashboard on http_port: live hashrate, shares, every hardware sensor, and overclocking. Also serves /status and /hive_status for mining OSes.
- `xelis` — xelhash.

`bzminer --list-plugins` prints what a binary actually loaded, and
`bzminer --list-algos` what it can mine.

## If something goes wrong

The first forty lines of the log name the build, the config file, the CPU, every
device found and the pool being tried. Start there, then:

| | |
|---|---|
| no GPUs found | update the GPU driver; `--gpu-info` shows what bzminer can see |
| a GPU is listed at startup but never mines | the log says why on its own line — no kernel for that architecture, a driver too old to load one, or not enough VRAM for the algorithm's table |
| pool rejects shares | almost always the wallet — check it is for the right coin, and that the placeholder in the sample script was replaced |
| overclock does nothing | on Linux these are privileged; run as root. The miner reports which knob the driver refused and why |
| dashboard not reachable | it listens on `127.0.0.1` only unless you set `http_address` |
| warthog finds nothing | it needs both a GPU and the CPU; check neither is disabled |
| ergo skips a card | the ~2 GB table has to fit in VRAM alongside everything else on the card |
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
its own, it is in the table above, and it is printed in the log at startup — so
you can see it before committing a rig, not discover it later. `sha256d`, the
open-source example, has none.

Every line about it is tagged `[dev fee]`, and each slice announces both ends:

```
[dev fee] warthog 2% - announced here, and at START and END of every slice
[dev fee] START - mining for the warthog developer on stratum+ssl://ussw.vipor.net:5120 (2.00% -> 13 pools (split evenly))
[dev fee] END - back to your pool stratum+ssl://us.vipor.net:5120 (paid 1.94% of this run so far, aiming for 2.00% -> 13 pools (split evenly))
```

The fee is spread over several destinations rather than sent to one, so no single
pool sees all of it. **You do not have to be able to reach them all.** A
destination that will not connect is backed off, and after three failed attempts
it is dropped for the rest of the run — the others carry its share and the fee is
still paid in full. If you block some of these pools at your firewall, the miner
notices once and stops trying; it will not warn you about a pool you blocked on
purpose. The one case it does warn about is the fee having nowhere left to go:

```
[dev fee] none of the 13 destinations could be reached - the fee is not being paid. Mining continues on your pool.
```

None of this happens on the thread that reads your pool's jobs, so an unreachable
dev pool costs your mining nothing — not a stale job, and not a reconnect.
