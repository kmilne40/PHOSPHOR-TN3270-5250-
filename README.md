<div align="center">

# PHOSPHOR

**A native, clean-room TN3270 / TN5250 mainframe audit terminal.**

Speaks the 3270 and 5250 data streams directly. No x3270, no s3270, no subprocess.
Full colour, IND$FILE transfer, FTP/JES, DB2/DRDA probing, a scriptable automation layer,
and a phosphor CRT look with switchable skins.

Companion tooling to *Hacking Mainframes* (No Starch Press) from [OffensiveSec.org](https://offensivesec.org).

</div>

<!-- Drop a screenshot or the promo GIF here once you have it in the repo, e.g. docs/phosphor.gif -->
<!-- ![PHOSPHOR](docs/phosphor.png) -->

---

## Authorised use only

PHOSPHOR connects to, logs into, and drives live mainframe sessions, and its recon suite
actively probes a target. Only point it at systems you own or have explicit written
permission to test. You are responsible for how you use it.

---

## What it does

| Area | Detail |
| --- | --- |
| **Protocols** | TN3270 / TN3270E and TN5250 (IBM i), spoken natively in pure Python |
| **Colour** | Carries colour and highlighting per cell, so ISPF / 3279 screens render in real colour |
| **File transfer** | IND$FILE in DFT mode (structured field 0xD0), ASCII and binary, plus FTP / JES job submission |
| **DB2** | DRDA EXCSAT probe that reports the DB2 server name and release |
| **TLS / AT-TLS** | Implicit TLS and STARTTLS, with cipher profiles for talking to legacy z/OS and IBM i |
| **Interfaces** | A desktop CRT app, a full-screen curses TUI (like c3270), and a headless recon scanner |
| **Automation** | A macro engine (record, script, replay) and an unattended bot layer with alerting |
| **Look** | 19 skins in a phosphor CRT aesthetic, from classic green through amber, LCARS, and brand skins |

PHOSPHOR is clean-room. It does not embed or wrap x3270/s3270. It also ships a
py3270-compatible `Emulator` shim so an existing py3270-based tool (for example a ported
CICSpwn) can run on PHOSPHOR's native client with no x3270 present.

---

## Requirements

- **Python 3.9 or newer** (for a source install)
- Runtime dependencies, pulled in automatically: `pygame`, `numpy`, `pillow`, `cryptography`
- A desktop session for the GUI. The curses TUI and the headless scanner run in a plain terminal.

Optional extras (install only what you need):

| Extra | Adds |
| --- | --- |
| `crt` | The GLSL CRT frontend (`moderngl`, `pyglet`) |
| `ezrecon` | Live OSINT recon (`dnspython`, `requests`, `beautifulsoup4`, `shodan`) |
| `tnz` | The IBM `tnz` engine for real z/OS / zPDT (`tnz`, `ebcdic`) |
| `report` | HTML report templating (`jinja2`) |
| `build` | Building standalone binaries (`pyinstaller`) |

---

## Install

### Option A: prebuilt installer (recommended for most users)

Grab the installer for your platform from the **[Releases page](../../releases)**:

- **Windows**: run the `.exe` installer, then launch PHOSPHOR from the Start menu.
- **macOS**: open the `.dmg` and drag PHOSPHOR to Applications. The build is unsigned, so on
  first launch use right-click then Open (or allow it under System Settings, Privacy and
  Security) to get past Gatekeeper.
- **Linux**: use the packaged build from Releases, or install from source (below).

Prebuilt installers bundle their own Python, so nothing else is required.

### Option B: from source (for developers and Linux)

```bash
git clone https://github.com/<your-org>/phosphor.git
cd phosphor
python3 -m venv .venv
source .venv/bin/activate            # Windows: .venv\Scripts\activate
pip install .
```

Add extras in the usual way:

```bash
pip install ".[crt,ezrecon,report]"
```

For a live-editing checkout while you work on it, use `pip install -e .`.

This installs the console commands `phosphor`, `phosphor-gui`, `phosphor-crt`, and
`phosphor-run-cicspwn`.

---

## Quick start

Launch the desktop app and connect straight to a host:

```bash
phosphor 192.168.0.96:23
```

Or launch the app unconnected and type the host into the CONNECT panel:

```bash
phosphor-gui
```

Drive a host from a full-screen terminal instead of the GUI:

```bash
phosphor 192.168.0.96 23 --client
```

Run the recon audit suite against a target:

```bash
phosphor 192.168.0.96 --scan --json
```

For IBM i (TN5250) add `--5250`. For real z/OS through the IBM stack add `--engine tnz`
(after `pip install ".[tnz]"`).

---

## The desktop app

The app is the main way to use PHOSPHOR: a CRT terminal with a connect panel, an operator
keypad (PF1 to PF24, PA keys, Enter, Clear, and the cursor and edit keys), a transfer panel,
and a tools panel.

**Connecting.** Type the host and port into the CONNECT panel and press Connect. For TLS,
you can use x3270-style host prefixes in the host field (`L:` for implicit TLS, `Y:` to
accept the presented certificate) or the TLS toggles in the panel.

**Skins.** PHOSPHOR ships 19 skins. Six are free and always available:
`classic`, `green`, `lcars`, `pink`, `bbc`, `z17`. The rest (including brand skins like
`covertaccess` and `hvck`) are unlocked with a code, see [Skins and unlocking](#skins-and-unlocking).

**Colour and mono.** Screens render in full 3279 colour by default. You can switch to a
single-phosphor mono look per skin.

---

## Terminal (TUI)

`--client` gives you a full-screen curses terminal that behaves like c3270: type into
fields, move with the arrows and Tab, and use your real function keys as PF keys. A colon
command line handles everything else.

```bash
phosphor 192.168.0.96 23 --client
```

Common colon commands:

| Command | Action |
| --- | --- |
| `:at R C` | Move the cursor to row R, column C |
| `:enter` | Send Enter |
| `:pf3` / `:pa1` | Send a PF or PA key |
| `:clear` | Send Clear |
| `:mono` | Toggle mono / colour |
| `:menu` | Open the transfer and FTP menu (IND$FILE up/down, FTP) |
| `:q` | Quit |

Add `--repl` if you want a plain line prompt rather than the full-screen interface.

---

## File transfer (IND$FILE)

PHOSPHOR speaks real IND$FILE DFT mode (structured field 0xD0), the same protocol c3270,
x3270 and wc3270 use, in both ASCII and binary.

- **Binary** transfers bytes verbatim. Use it for load modules, RACF database images, and
  anything that is not line-oriented text.
- **ASCII** (text) transfers translate between your local text and the host EBCDIC (CP037),
  splitting records on the EBCDIC newline. Line endings are normalised to the host record
  model, which means a text file's trailing newline is treated as a line terminator rather
  than content. If you need every byte preserved exactly, use binary.

In the GUI use the TRANSFER panel; in the TUI use `:menu`.

> **Note on real hosts.** The ASCII translation assumes CP037. Some systems use a different
> EBCDIC variant (for example CP1047 or a country-specific page). If a text transfer against
> a live host comes back with a handful of wrong characters, that codepage is the setting to
> check.

---

## TLS and legacy AT-TLS

Modern OpenSSL refuses old TLS versions and cipher suites by default, which is exactly what
you do not want when auditing an older z/OS or IBM i that only offers them. PHOSPHOR exposes
cipher profiles so you can dial the client down to meet the host:

| Profile | For |
| --- | --- |
| `modern` | Default. Current TLS only. |
| `compat` / `mainframe` | Older z/OS AT-TLS: lowers the security level and floors TLS at 1.0 |
| `ibmi` | Older IBM i Telnet-SSL |
| `legacy` | Last resort. Accepts the widest set of old suites. |

In a frozen / installer build, the very old suites (3DES, RC4, single DES) also need
OpenSSL's legacy provider enabled in the environment, since OpenSSL 3.x drops them from the
default provider. This is a deployment detail, not a code change.

> Cipher-profile behaviour is validated in the test suite against a controlled TLS 1.0 host.
> Against a specific live mainframe you should still confirm the handshake, since AT-TLS
> policies vary site to site.

---

## Automation

### Macros

The macro engine drives a session with a small, readable verb set. Use it as a `.pmac`
script, as a Python API, or record a session and replay it. Every step is written to an
audit transcript, which matters for authorised testing: you get a record of exactly what the
macro did.

A `.pmac` script (`logon.pmac`):

```
# log on to TSO over AT-TLS and capture the READY screen
connect 192.168.0.96 992 tls profile=mainframe
wait_for LOGON 20
string LOGON APPLID(TSO)
key ENTER
wait_for PASSWORD 15
string CHANGEME
key ENTER
wait_for READY 20
snapshot tso-ready
save_screen tso-ready.txt
disconnect
```

Run it:

```bash
python -m phosphor.macro logon.pmac
```

Or script it in Python for loops and conditionals:

```python
from phosphor.macro import MacroEngine

m = MacroEngine(default_timeout=20)
m.connect("192.168.0.96", 992, tls=True, cipher_profile="mainframe")
m.wait_for("LOGON")
m.enter("LOGON APPLID(TSO)")
m.wait_for("PASSWORD")
m.enter("CHANGEME")
m.wait_for("READY")
print(m.transcript_text())
m.disconnect()
```

Verbs: `connect`, `string`, `move`, `key` (ENTER, CLEAR, PF1 to PF24, PA1 to PA3), `enter`,
`wait`, `wait_for`, `expect`, `snapshot`, `save_screen`, `log`, `disconnect`. A recorder
turns any run into a replayable script.

### Bots (unattended runs)

The bot layer runs a macro on a schedule with retry, reconnect-on-drop, a JSON run log, and
alert hooks.

```bash
python -m phosphor.bot audit.pmac --every 3600 --retries 3 --log bot.jsonl
```

In Python you can wire real alerts (Slack / Teams webhook, or SMTP e-mail):

```python
from phosphor.bot import Bot, Job, webhook_alerter, email_alerter, combine

def probe(engine):
    engine.connect("192.168.0.96", 992, tls=True, cipher_profile="mainframe")
    engine.wait_for("READY", timeout=30)
    engine.snapshot("ready")

notify = combine(
    webhook_alerter("https://hooks.example.com/phosphor"),
    email_alerter("soc@example.com"),          # SMTP details from PHOSPHOR_SMTP_* env vars
)

bot = Bot(log_path="phosphor-bot.jsonl")
bot.add_job(Job("ready-probe", probe, every=300, max_retries=3, on_failure=notify))
bot.run()
```

E-mail alerts read `PHOSPHOR_SMTP_HOST`, `PHOSPHOR_SMTP_PORT`, `PHOSPHOR_SMTP_USER`,
`PHOSPHOR_SMTP_PASS` and `PHOSPHOR_SMTP_FROM` from the environment.

---

## Recon and audit

The `--scan` mode runs the audit suite: TN3270 screen capture, VTAM applid enumeration, CICS
transaction enumeration, FTP anonymous check, and a DB2 DRDA probe. It writes findings to the
console, to JSON with `--json`, or to an HTML or Markdown report with `--report out.html`.

```bash
phosphor 192.168.0.96 --scan --applids TSO,CICS,DB2,IMS --report scan.html
```

Useful flags: `--menu` for an interactive recon menu, `--lab` to refuse any target that is
not RFC1918 (a guard rail for lab work), and `--users` / `--applids` to seed the enumerators.

The recon suite is gated behind an unlock code, see below.

---

## Skins and unlocking

Six skins are free and always on: `classic`, `green`, `lcars`, `pink`, `bbc`, `z17`. The
remaining skins and the pen-test tooling are unlocked with a code.

- **Skins** unlock with a shared master code. Enter it once in the app (About, then the
  unlock field) and it is remembered.
- **Pen-test tools** (the recon suite) unlock with a separate code, passed as
  `phosphor ... --unlock CODE` or entered in the tools panel, and then remembered.

Codes are stored only as scrypt hashes in the binary, never in the clear, so the shipped
build never contains the code itself.

If you want the skin pack or the tooling unlock, the support tiers are **GBP 5** for an
individual licence and **GBP 50** for a corporate licence. That covers a receipt or invoice,
a named licence, and priority support. Contact details are on
[OffensiveSec.org](https://offensivesec.org).

---

## Python API

The native client is usable on its own:

```python
from phosphor.protocols.tn3270.client import TN3270Client

c = TN3270Client()
c.connect("192.168.0.96", 23)
c.move_to(20, 14)
c.send_string("LOGON APPLID(TSO)")
c.send_enter()
for line in c.screen_get():
    print(line)
```

The client exposes `connect`, `send_string`, `send_enter`, `send_clear`, `send_pf(n)`,
`send_pa(n)`, `move_to(row, col)`, `screen_get()`, `find(text)`, `get_pos()`, and
`colour_grid()`, plus TLS options (`tls`, `tls_starttls`, `tls_verify`, `cipher_profile`).

---

## Development

```bash
pip install -e .
./run-tests.sh              # or: for t in tests/test_*.py; do python3 "$t"; done
```

The suite is self-contained and runs headless (set `SDL_VIDEODRIVER=dummy` and
`SDL_AUDIODRIVER=dummy` for the GUI-adjacent tests). IND$FILE interop against the GIBSON
training host is covered by `verify_indfile_interop.py`.

---

## Credits

Clean-room implementation. The lineage it builds on is credited in `CREDITS`: CICSpwn by
Ayoub Elaassal, and the TN3270 research tradition (mainframed767 and others). PHOSPHOR does
not include their code; it reimplements the protocol and provides a compatible shim so their
tools can run without x3270.

---

## Licence

See `LICENSE` and `THIRD-PARTY-NOTICES` in the repository. PHOSPHOR is offensive security
tooling for authorised testing. Use it lawfully.
