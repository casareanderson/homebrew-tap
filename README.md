# casareanderson/tap

A Homebrew tap for casareanderson tools. It currently ships one formula: sieve, a CVE scanner for Linux hosts and containers that ranks findings by CISA KEV and EPSS.

![Real sieve run, 19 October 2026: a `sieve path` scan ranks the known-malicious flatmap-stream 0.1.1 first](docs/sieve-run.png)

[![Licence: AGPL-3.0](https://img.shields.io/github/license/casareanderson/homebrew-tap)](LICENSE)

## What it does

- Installs the current release of sieve on macOS or Linux with one command: `brew install casareanderson/tap/sieve`.
- Installs each tool into its own virtualenv from its released PyPI build, so it never touches your system Python or the `requests` and `certifi` versions other tools rely on.
- Stays current through `brew upgrade`, and ships a `brew test` block that checks the installed version and runs a real path scan.
- Is a distribution point only: the tap contains no scanner code. The scanner itself lives in the [sieve repository](https://github.com/casareanderson/sieve).

## Screenshot

The image above is a real run from 19 October 2026. After installing, `sieve path ./demo` reads a project's lockfiles (no node or pip needed), queries OSV.dev live and ranks the known-malicious `flatmap-stream` 0.1.1 first. sieve exits 1 when it finds malware or a known-exploited CVE.

## Quick start

Prerequisites:

- Homebrew on macOS or Linux (this tap is Linux-first; see Status and limits).
- An internet connection the first time you scan, so sieve can fetch its advisory cache.

```sh
brew install casareanderson/tap/sieve
sieve --version
```

Success looks like `sieve 0.6.0`. To check the formula itself:

```sh
brew test casareanderson/tap/sieve
```

## Usage

The commands you use after installing (full reference in the [sieve README](https://github.com/casareanderson/sieve)):

- `sieve host` — scan this machine's dpkg, apk or rpm packages, plus pip and npm packages outside the OS.
- `sieve path DIR` — scan a project from its lockfiles or `node_modules`, without running node or pip.
- `sieve remote user@host` — scan another machine over SSH with nothing installed there.
- `sieve sbom FILE` — scan a CycloneDX or SPDX SBOM produced by another tool.
- `sieve verify` — check that no installed file changed since its package manager put it there.

A real `sieve path` scan, reproduced on 19 October 2026:

```
# path:/tmp/demo: project files, 1 packages  (read 1 node_modules, 1 package-lock.json)

ALERT      EPSS  SEVERITY  IN USE  ID              PACKAGE         INSTALLED → FIXED             TARGET
☠ MALWARE  -     CRITICAL  ?       MAL-2025-20690  flatmap-stream  0.1.1 → malicious: remove it  path:/tmp/demo

1 MALICIOUS PACKAGE (remove it, rotate secrets), 1 findings, 0 actively exploited, 0 fixable
```

## Configuration

The tap itself reads no configuration. The tool it installs does: sieve reads one environment variable, which the `--cache` flag overrides.

| Variable | Default | What it does |
|---|---|---|
| `XDG_CACHE_HOME` | `~/.cache` | Where sieve keeps its advisory cache (`sieve/sieve.db`); the CISA KEV and EPSS files sit next to it. |

## How it works

`brew install casareanderson/tap/sieve` tells Homebrew to fetch the `sieve.rb` formula from this tap. The formula:

1. Downloads the released `sieve_scan` 0.6.0 build from PyPI with its pinned dependencies (`requests`, `urllib3`, `idna`, `charset-normalizer`) and `certifi`.
2. Creates a dedicated Python virtualenv in the Homebrew prefix.
3. Installs the package and its dependencies into that virtualenv and links the `sieve` binary.

The installed sieve reads the package lists already on the machine, asks OSV.dev which advisories affect those exact versions, and ranks the results by CISA KEV and EPSS. Only package names and versions go to OSV.dev; nothing else about the machine does.

```
homebrew-tap/
├── Formula/
│   └── sieve.rb        # the only formula; pins sieve_scan 0.6.0 from PyPI
├── docs/
│   └── sieve-run.png   # real capture of a scan (19 October 2026)
└── README.md
```

## Status, limits and real results

Measured 19 October 2026:

- `sieve 0.6.0` installs from PyPI into a fresh virtualenv and runs — the same release and mechanism the formula pins.
- A real `sieve path` scan against the live OSV.dev API ranked `MAL-2025-20690` (the known-malicious `flatmap-stream` 0.1.1) first and exited 1, as designed.
- The formula's `test` block (run by `brew test`) checks `sieve --version` and runs a path scan over a pinned `requests` dependency.

Not measured: a full `brew install` from this tap on a real Homebrew box. This build host has no Homebrew, so that path is untested here; if it breaks for you, open an issue.

What the tap does not do:

- It only distributes sieve. It does not build, vendor or fix the scanner.
- `sieve host` reads dpkg, apk and rpm databases, which macOS does not have. On a Mac, host scans cover pip and npm packages but not OS packages; `sieve path`, `sieve sbom` and `sieve remote` work whatever OS you run them from.
- New tools only appear here after a formula is added; nothing watches releases automatically.

## Roadmap

Nothing planned beyond keeping the sieve formula current as new releases land.

## Licence and credits

The formula — and the sieve project it installs — are AGPL-3.0-only (see the repo LICENSE). sieve queries OSV.dev, the CISA Known Exploited Vulnerabilities catalogue and the FIRST EPSS feed; see the [sieve repository](https://github.com/casareanderson/sieve) for those upstream data sources.
