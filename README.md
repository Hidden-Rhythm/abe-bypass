<div align="center">

# 👻 phantom elevator

### Multi-Browser Credential Extraction · App-Bound Encryption Bypass

*Walk Chromium and Firefox profile stores, recover the plaintext, rank what matters.*

<br>

<a href="https://github.com/Hidden-Rhythm/abe-bypass">
  <img src="https://img.shields.io/badge/💻_Source-GitHub-18181b?style=for-the-badge&logo=github&logoColor=white" alt="GitHub Source">
</a>
<img src="https://img.shields.io/badge/🐍_Python-3.10%2B-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python 3.10+">
<img src="https://img.shields.io/badge/🪟_Windows-10%20%7C%2011-0078D6?style=for-the-badge&logo=windows11&logoColor=white" alt="Windows 10/11">
<img src="https://img.shields.io/badge/Version-1.0.0-52525b?style=for-the-badge" alt="Version 1.0.0">

<br><br>

**DPAPI + V20 BYPASS** &nbsp;·&nbsp; **COM ELEVATION BRIDGE** &nbsp;·&nbsp; **SIGNAL-FIRST RANKING**

<br>

[**Quickstart**](#quickstart) · [**CLI**](#cli) · [**Browsers**](#browsers) · [**Bypass**](#bypass) · [**Obfuscation**](#obfuscation) · [**Limitations**](#limitations)

</div>

---

## What Is Phantom Elevator?

**phantom elevator** is a Windows credential extractor built for the post-ABE era. It walks Chromium and Firefox profile directories, locates every encrypted secret store, and recovers the plaintext — including the **v20 App-Bound Encryption** keys that broke every legacy DPAPI-only extractor.

It is a single-file Python application. No service to install, no driver to sign, no agent to persist.

> **The goal:** point it at a machine, let it detect what is installed, and read the output.

<br>

<table>
<tr>
<td width="33%" valign="top">

**Detect**
Enumerates Chromium and Firefox families from standard install paths. Missing browsers are skipped, not fatal.

</td>
<td width="33%" valign="top">

**Recover**
Resolves each browser's key store — `v10` via DPAPI, `v20` via an in-process elevation bridge compiled at runtime.

</td>
<td width="33%" valign="top">

**Rank**
Scores every local-storage entry before writing. Wallets, tokens, and seed phrases lead the output.

</td>
</tr>
</table>

---

<a id="chain"></a>
## The Chain That Broke

For years, credential tooling relied on one chain:

```text
Local State  →  encrypted_key (v10)  →  CryptUnprotectData  →  AES-256-GCM  →  plaintext
```

That chain still works — for `v10`.

Chrome 127 introduced **App-Bound Encryption**. The master key is now sealed inside a COM elevation service that only the browser's own signed process is supposed to reach. DPAPI returns ciphertext. The old chain dead-ends.

---

<a id="bypass"></a>
## The ABE Problem

Phantom Elevator adds the missing bridge.

```text
Local State  →  encrypted_key (v20)  →  elevation-service bridge  →  master key  →  AES-256-GCM  →  plaintext
```

<br>

<details open>
<summary><b>The bypass, step by step</b></summary>

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

---

<a id="browsers"></a>
## Supported Browsers

### Chromium family

| | Browser | Key protection | ABE bridge |
|:---:|:---|:---|:---:|
| <img src="https://www.google.com/s2/favicons?domain=google.com&sz=64" height="18"> | **Google Chrome** | DPAPI + ABE | ✅ |
| <img src="https://www.google.com/s2/favicons?domain=google.com&sz=64" height="18"> | Chrome Beta | DPAPI + ABE | ✅ |
| <img src="https://www.google.com/s2/favicons?domain=brave.com&sz=64" height="18"> | **Brave** | DPAPI + ABE | ✅ |
| <img src="https://www.google.com/s2/favicons?domain=microsoft.com&sz=64" height="18"> | Microsoft Edge | DPAPI + ABE | ✅ |
| <img src="https://www.google.com/s2/favicons?domain=avast.com&sz=64" height="18"> | Avast Secure Browser | DPAPI + ABE | ✅ |
| <img src="https://www.google.com/s2/favicons?domain=vivaldi.com&sz=64" height="18"> | Vivaldi | DPAPI | ➖ |
| <img src="https://www.google.com/s2/favicons?domain=chromium.org&sz=64" height="18"> | Chromium | DPAPI | ➖ |
| <img src="https://www.google.com/s2/favicons?domain=opera.com&sz=64" height="18"> | Opera | DPAPI | ➖ |

### Firefox family

| | Browser | Key protection | Status |
|:---:|:---|:---|:---:|
| <img src="https://www.google.com/s2/favicons?domain=mozilla.org&sz=64" height="18"> | **Firefox** | NSS / `key4.db` | ✅ |
| <img src="https://www.google.com/s2/favicons?domain=waterfox.net&sz=64" height="18"> | Waterfox | NSS / `key4.db` | ✅ |
| <img src="https://www.google.com/s2/favicons?domain=librewolf.net&sz=64" height="18"> | LibreWolf | NSS / `key4.db` | ✅ |
| <img src="https://www.google.com/s2/favicons?domain=floorp.app&sz=64" height="18"> | Floorp | NSS / `key4.db` | ✅ |
| <img src="https://www.google.com/s2/favicons?domain=zen-browser.app&sz=64" height="18"> | Zen Browser | NSS / `key4.db` | ✅ |

Installation is detected automatically from standard `%LocalAppData%` and `%ProgramFiles%` paths. Missing browsers are skipped, not fatal.

---

## Extraction Matrix

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

Every category is optional. Run a single target or the full set.

---

<a id="signal"></a>
## Signal-First Filtering

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

---

<a id="quickstart"></a>
## Getting Started

### Requirements

- Windows **10 / 11**, x64
- Python **3.10 or later**
- MinGW-w64 `gcc` or MSVC `cl` — optional; enables the ABE / v20 path
- Network on first run, for dependency bootstrap

Dependencies resolve automatically on launch:

| Package | Purpose |
|:---|:---|
| `pycryptodome` | AES-256-GCM |
| `python-snappy` | LevelDB block decompression |
| `gcc` | attempted via `winget`, `choco`, then `scoop` |

If no compiler can be installed, the tool prints a warning and continues in `v10`-only mode.

### 1. Clone the Repository

```bash
git clone https://github.com/Hidden-Rhythm/abe-bypass.git
cd abe-bypass
```

### 2. Launch the Application

```bash
python phantom_elevator.py
```

### 3. Read the Output

```text
cookies.txt
passwords.txt
local_storage.txt
```

The tool prints a banner, detects installed browsers, bootstraps missing packages, compiles the ABE bridge when needed, and walks each profile. Progress is reported per browser and per category.

```bash
python phantom_elevator.py --help
```

Full flag surface, recipes, and exit codes: [CLI Reference](#cli).

---

## Input & Output

Phantom Elevator discovers profile directories on its own. Use a profile override when pointing at a portable install, another user's profile, or a mounted disk.

**Example input**

```text
C:\Users\<name>\AppData\Local\Google\Chrome\User Data\Default\
```

**Output**

Every extracted category is written as a plain-text file beside the script:

| Artifact | Format | Notes |
|:---|:---|:---|
| `cookies.txt` | text | import-ready |
| `passwords.txt` | text | `url · user · password` |
| `history.txt` | text | timestamped |
| `autofill.txt` | text | field / value |
| `cards.txt` | text | masked in console |
| `local_storage.txt` | text | ranked by signal score |
| `bookmarks.txt` | text | folder tree preserved |
| `extensions.txt` | text | ID, name, version, path |

Outputs land beside the script unless an alternate path is supplied. Temporary injection artifacts are written to a per-run sandbox under `TEMP` and are not reused across runs.

---

<a id="cli"></a>
## Command Line

With no arguments, Phantom Elevator runs fully interactive: it detects browsers, prints the menu, and asks what to extract.

```bash
python phantom_elevator.py
```

```bash
python phantom_elevator.py --help
```

> **Flag table pending.** Every optional flag, its argument, its default, and its exit-code meaning needs to come from the `parser.add_argument(...)` calls in `phantom_elevator.py`. Paste that block and the full Options / Recipes / Exit-codes tables land in one pass — no guessing.

---

<a id="obfuscation"></a>
## Hardening the Payload

> **Obfuscate before you ship.** Phantom Elevator is a single-file Python application — a drop-in target for the [**Hidden-Rhythm obfuscator**](https://github.com/Hidden-Rhythm/obfuscator) pipeline.

```text
              ┌──────────────────────────┐
              │     01. Read Source      │
              └────────────┬─────────────┘
                           ▼
              ┌──────────────────────────┐
              │ 02. Encrypt Constants    │
              └────────────┬─────────────┘
                           ▼
              ┌──────────────────────────┐
              │    03. Build Payload     │
              └────────────┬─────────────┘
                           ▼
              ┌──────────────────────────┐
              │    04. Build Stage 2     │
              └────────────┬─────────────┘
                           ▼
              ┌──────────────────────────┐
              │   05. Build Bootstrap    │
              └────────────┬─────────────┘
                           ▼
              ┌──────────────────────────┐
              │    06. Emit Surface      │
              └────────────┬─────────────┘
                           ▼
              ┌──────────────────────────┐
              │    07. Verify Syntax     │
              └────────────┬─────────────┘
                           ▼
              ┌──────────────────────────┐
              │     08. Write Output     │
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

| Layer | Effect |
|:---|:---|
| AST transformation | restructures eligible syntax-tree nodes |
| Constant encoding | XOR-encodes strings and byte constants |
| Payload serialization | compiles to a `marshal` code object |
| Compression | `zlib` + base64 chunking |
| Layered loader | bootstrap and runtime-loading stages |
| Generated identifiers | breaks static symbol matching |
| Syntax verification | compiles the output before writing |

<br>

The generated payload uses `marshal`-serialized code objects, which are Python-version dependent. Build and run on the **same interpreter version**. The obfuscator's runtime environment checks can affect execution in unusual sandboxes — test the generated file before deploying it.

The obfuscator also reports a SHA-256 build fingerprint and per-stage timings, so every hardened build is traceable to its source.

---

## Architecture

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
│                 plain-text output files                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Technical Details

| Component | Implementation |
|---|---|
| Language | Python 3.10+ |
| ABE bridge | C (MinGW-w64 `gcc` / MSVC `cl`) |
| Key unprotection | DPAPI `CryptUnprotectData` + AES-256-GCM |
| ABE access | COM elevation service + DLL injection |
| Injection primitives | `OpenProcess` · `VirtualAllocEx` · `WriteProcessMemory` · `CreateRemoteThread` |
| LevelDB parsing | `python-snappy` |
| Crypto | `pycryptodome` |
| Cookie serialization | Netscape |
| Password serialization | TSV |
| Output format | plain text |
| Build fingerprint | SHA-256 (via obfuscator) |

<a id="limitations"></a>
### Compatibility & Limitations

- **Windows only.** The ABE bridge and DPAPI calls are Win32-specific.
- **ABE is per-build.** Chromium CLSIDs, IIDs, service names, and vtable offsets change across browser releases. The browser table in the script is the source of truth — update it when a target ships a new elevation service.
- **Compiler required for v20.** Without `gcc` or `cl`, the ABE path is disabled. `v10` and Firefox extraction still run.
- **Running browsers may lock stores.** SQLite and LevelDB files can be held open. Close the target browser for the cleanest read, or rely on the tool's retry logic (`DEFAULT_RETRIES = 5`).
- **Antivirus is expected to react.** DLL injection and credential access are high-signal behaviors. The obfuscator layer raises static-analysis cost; it does not make the runtime behavior invisible.
- **Obfuscator output is version-bound.** Generate and execute on the same Python minor version.

Test generated output thoroughly before using it in a production workflow.

---

## Project Structure

```text
abe-bypass/
└── phantom_elevator.py
```

The entire application — browser tables, DPAPI layer, ABE compiler, injector, profile walkers, and output writers — lives in one file. No package tree, no build step, no external config.

---

## Built With

| Technology | Purpose |
|---|---|
| Python | Core application and extraction pipeline |
| DPAPI | `v10` key unprotection |
| COM | Elevation-service access for `v20` |
| `pycryptodome` | AES-256-GCM decryption |
| `python-snappy` | LevelDB block decompression |
| Rich | Terminal banner and progress display |

---

## Links

<div align="center">

<a href="https://github.com/Hidden-Rhythm/abe-bypass">
  <img src="https://img.shields.io/badge/Repository-View%20Source-18181b?style=for-the-badge&logo=github&logoColor=white" alt="View Repository">
</a>

<a href="https://github.com/Hidden-Rhythm/obfuscator">
  <img src="https://img.shields.io/badge/Companion-obfuscator-52525b?style=for-the-badge&logo=github&logoColor=white" alt="Obfuscator">
</a>

<a href="https://github.com/Hidden-Rhythm">
  <img src="https://img.shields.io/badge/Developer-Hidden--Rhythm-52525b?style=for-the-badge&logo=github&logoColor=white" alt="Developer Profile">
</a>

<a href="https://discord.com/users/hidden_rhythm">
  <img src="https://img.shields.io/badge/Discord-hidden__rhythm-5865F2?style=for-the-badge&logo=discord&logoColor=white" alt="Discord">
</a>

</div>

---

<div align="center">

### 👻 Built with Python.

**Hidden-Rhythm**

<sub>phantom elevator · v1.0.0 · ABE bypass · 13 browser families</sub>

</div>
