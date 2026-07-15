<div align="center">

# PHOSPHOR

### A native, clean-room TN3270 / TN5250 mainframe audit terminal and security toolkit — with a phosphor-CRT interface.

[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-informational)](#installation)
[![Python](https://img.shields.io/badge/python-3.11%20%7C%203.12%20%7C%203.13-blue)](#build-from-source)
[![Licence](https://img.shields.io/badge/licence-Proprietary%20(EULA)-lightgrey)](LICENSE.txt)
[![Status](https://img.shields.io/badge/status-active-success)]()

*Speaks the 3270 / 5250 datastream directly — no x3270, no s3270, no subprocess.*

</div>

---

> ### ⚠️ Authorised testing only
> PHOSPHOR is a security-testing and training tool for IBM mainframe (z/OS) and midrange
> (IBM i) environments. **Use it only against systems you own or are explicitly, currently
> authorised in writing to test.** You are solely responsible for obtaining permission and
> for complying with all applicable laws. See the [licence](LICENSE.txt).

---

## What is PHOSPHOR?

PHOSPHOR is a full-colour TN3270 / TN5250 terminal and an integrated mainframe security
toolkit in one application. It implements the 3270 and 5250 datastreams natively in pure
Python — so it renders ISPF, SDSF, CICS and IBM i panels in true colour, drives sessions
programmatically, and wraps a suite of recon and offensive-security modules behind a
retro cathode-ray-tube interface.

It is the companion tool to the mainframe-security work at
[OffensiveSec.org](https://offensivesec.org), built for red teams, penetration testers,
and anyone learning how mainframe attack surfaces really work.

<div align="center">

<!-- Replace with real screenshots before publishing -->
<em>[ screenshot: docs/images/standard.png]&nbsp;&nbsp;&nbsp;[ screenshot: recon menu ]&nbsp;&nbsp;&nbsp;[ screenshot: DB2 / DRDA client ]</em>

</div>

---

## Features

**Terminal**
- Native **TN3270** and **TN5250** clients — full colour per cell, correct field/attribute
  handling, proper `IAC EOR` sync (no sleeps).
- **Terminal models 2/3/4/5** (24×80 / 32×80 / 43×80 / 27×132).
- **TLS / AT-TLS** (implicit, incl. the secure telnet port 992) and negotiated
  **STARTTLS** (RFC 2946), with a no-verify mode for legacy self-signed hosts.
- A GPU **CRT frontend** (`phosphor-crt`) with adjustable bloom, scanlines, curvature,
  burn-in, flicker and colour presets.
- A full-screen **curses TUI** (`--client`) that behaves like c3270, and a scriptable
  py3270-style `Emulator` API.

**Recon & audit**
- Menu-driven recon: `tn3270-screen`, `vtam-enum`, `tso-enum`, `cics-enum`,
  `cics-user-enum`, `cics-info`, `ftp-anon`, `db2-das-info`.
- Runs with a built-in **native engine** (no nmap required) or emits the exact
  `nmap --script` command; findings export to an **HTML / Markdown report**.
- **EZrecon** external-recon integration (nmap, whois, subfinder, amass, Shodan, DNS).

**Offensive toolkit**
- **iNJEctor** — NJE tooling: node OPEN-probe and enumeration, sign-on testing, NMR
  operator commands, and **job submission** that runs on the target as an arbitrary
  user (the classic NJE lateral-movement / RACF-backdoor path).
- **DB2 / DRDA** — server fingerprinting, credential testing and spraying, immediate SQL
  (`EXCSQLIMM`) and native **result-set retrieval** (`OPNQRY` / FD:OCA) with no driver;
  plus a DB2 WebSocket client and an `ibm_db` command builder.
- **PassTicket** — RACF PassTicket generation and classification.
- **RACF DES** hash cracking, hash identification, and a local shell (with an optional
  reverse-shell listener).
- **CICSpwn** — runs the real CICSpwn through PHOSPHOR's native client (no x3270).

**File transfer**
- **IND$FILE** upload/download over a real TSO session.
- **FTP / JES** — banner/auth/SITE/LIST/RETR/STOR and JES job submission.

---

## Installation

Download the build for your platform from the [**Releases**](../../releases) page.

> The release binaries are **not code-signed**, so each OS warns you on first launch. This
> is expected for an independent release — see [First-run warnings](#first-run-warnings).

### 🪟 Windows

1. Download **`PHOSPHOR-Setup.exe`** from the latest release.
2. Run it. If SmartScreen appears, click **More info → Run anyway**.
3. Launch **PHOSPHOR** from the Start menu.

Windows 10/11, 64-bit.

### 🍎 macOS

1. Download **`PHOSPHOR-<version>.dmg`** (match your chip — Apple Silicon or Intel).
2. Open the DMG and drag **PHOSPHOR** to **Applications**.
3. First launch: **right-click the app → Open → Open** (Gatekeeper blocks unsigned apps on
   a normal double-click). Or clear it from Terminal:
   ```bash
   xattr -dr com.apple.quarantine /Applications/PHOSPHOR.app
   ```

### 🐧 Linux

Several ways, pick what suits you.

**Option 1 — AppImage (portable, no install):**
```bash
chmod +x PHOSPHOR-*.AppImage
./PHOSPHOR-*.AppImage
```

**Option 2 — tarball (portable):**
```bash
tar -xzf PHOSPHOR-*-linux-x86_64.tar.gz
./PHOSPHOR/PHOSPHOR
```

**Option 3 — install from source with `git clone` (recommended for developers):**
```bash
git clone https://github.com/[YOUR-USER]/phosphor.git
cd phosphor
python3 -m venv .venv && source .venv/bin/activate
pip install -e .

# then run any of:
phosphor-app                       # the CRT GUI application
phosphor 192.168.0.96 --menu       # interactive recon menu
phosphor 192.168.0.96 --client     # full-screen c3270-style TUI
```

Install optional capabilities as extras:
```bash
pip install -e ".[ezrecon,tnz,crt,report]"
#   ezrecon -> nmap/whois/subfinder/amass/Shodan/DNS recon backends
#   tnz     -> IBM tnz backend for TN3270/TN5250
#   crt     -> the GPU cathode-ray-tube frontend (phosphor-crt)
#   report  -> HTML/Markdown report generation
```

**Option 4 — desktop menu entry** (after building the frozen app, see below):
```bash
bash packaging/linux/install.sh              # installs to ~/.local + adds a menu icon
bash packaging/linux/install.sh --with-recon # also installs the EZrecon external tools
# uninstall:
bash packaging/linux/uninstall.sh
```

Runtime needs the usual SDL/X11 desktop libraries (present on any desktop; absent on bare
headless servers). On Debian/Ubuntu you may also need `sudo apt install python3-venv`.

---

## Quick start

```bash
# GUI (the CRT terminal + toolkit)
phosphor-app

# CLI recon suite against a host
phosphor 192.168.0.96 --scan --applids TSO,CICS,DB2 --report scan.html

# Interactive terminal (full-screen TUI, like c3270)
phosphor 192.168.0.96 --client

# TN5250 (IBM i)
phosphor 192.168.0.96 --client --5250

# Standalone GPU CRT terminal
phosphor-crt 192.168.0.96          # live
phosphor-crt --demo                # offline colour demo
```

Scriptable, py3270-style API:
```python
from phosphor.protocols.tn3270.client import TN3270Client
c = TN3270Client()
c.connect("192.168.0.96", 23)
c.move_to(20, 14); c.send_string("LOGON APPLID(CICS)"); c.send_enter()
for line in c.screen_get():
    print(line)
```

> Some offensive modules require an **unlock code** (`--unlock CODE`); premium **skins** are
> unlocked with a paid code. See the [licence](LICENSE.txt).

---

## Build from source

PyInstaller is **not** a cross-compiler — build each target on that platform. The build
scripts create a clean virtualenv, install dependencies, run a preflight and the test
suite, freeze the app, and package it. Each needs a 64-bit **CPython 3.11–3.13**.

| Platform | Script | Produces | Also needs |
|----------|--------|----------|------------|
| Windows  | `Build-PHOSPHOR-Windows.ps1` | `PHOSPHOR-Setup.exe` | [Inno Setup 6](https://jrsoftware.org/isdl.php) |
| macOS    | `Build-PHOSPHOR-macOS.sh`    | `PHOSPHOR.app` + `.dmg` | Xcode command-line tools |
| Linux    | `Build-PHOSPHOR-Linux.sh`    | tarball + AppImage | `appimagetool` (for the AppImage) |

```bash
# examples
pwsh  ./Build-PHOSPHOR-Windows.ps1
bash  ./Build-PHOSPHOR-macOS.sh   --python /opt/homebrew/bin/python3.12
bash  ./Build-PHOSPHOR-Linux.sh   --python python3.12
```

Run them from the extracted source folder (the one containing `phosphor/`, `packaging/`,
`tests/`), or place the release zip beside the script and it will expand it.

---

## First-run warnings

Because the binaries are unsigned:

- **Windows** — SmartScreen: **More info → Run anyway**.
- **macOS** — Gatekeeper: **right-click → Open → Open**, or `xattr -dr com.apple.quarantine`.
- **Linux** — no gate; `chmod +x` and run.

PyInstaller-packaged security tools are sometimes flagged as false positives by
antivirus/SmartScreen; the binaries are produced from the source in this repository.

---

## Requirements

- **Run the packaged app:** nothing — everything is bundled.
- **Run / build from source:** CPython **3.9+** to run, **3.11–3.13 (64-bit)** to build
  binaries. Core dependencies: `pygame`, `numpy`, `pillow`, `cryptography`.

---

## Project layout

```
phosphor/
├── core/           colour-carrying screen model (Cell, Field, ScreenBuffer)
├── protocols/
│   ├── tn3270/     telnet + TN3270E negotiation, datastream, client
│   ├── tn5250/     TN5250 client (IBM i)
│   ├── drda/       native DB2 DRDA client (fingerprint, SQL, result sets)
│   ├── indfile/    IND$FILE transfer
│   ├── ftp/        FTP / JES
│   └── tls.py      TLS / AT-TLS / STARTTLS
├── toolchain/      nje (iNJEctor), passticket, racf_des, crack, shell, cicspwn, ...
├── audit/          recon + EZrecon
├── crt/            GPU cathode-ray-tube frontend
├── app/            the CRT GUI application
└── cli.py          command-line entry point
packaging/          build specs + Windows/macOS/Linux installer tooling
tests/              self-contained test suite
```

---

## Licence

PHOSPHOR is proprietary software, licensed under its End User Licence Agreement —
see [`LICENSE.txt`](LICENSE.txt). In short: you may download and use it, but you may not
modify, reverse-engineer, or redistribute it; additional skins require a paid licence.

Bundled third-party open-source components and their licences are listed in
[`THIRD-PARTY-NOTICES.txt`](THIRD-PARTY-NOTICES.txt).

---

## Credits

PHOSPHOR is a clean-room implementation. It gratefully acknowledges the research and tools
that mapped this territory:

- **CICSpwn** — Ayoub Elaassal
- **TN3270 / mainframe security research** — Philip Young (mainframed767)
- **NJE tooling lineage** — Soldier of Fortran

---

## Disclaimer

PHOSPHOR is provided "as is", without warranty of any kind. The authors accept no liability
for misuse. It is intended for lawful, authorised security testing and education only.

---

<div align="center">

**[YOUR NAME / TRADING NAME]** · [OffensiveSec.org](https://offensivesec.org) · [CONTACT EMAIL]

</div>
