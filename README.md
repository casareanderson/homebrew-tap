# casareanderson/tap

Homebrew formulae for my tools.

```sh
brew install casareanderson/tap/sieve
```

| Formula | What it is |
|---|---|
| [sieve](https://github.com/casareanderson/sieve) | CVE scanner for Linux hosts and containers using OSV.dev, CISA KEV and EPSS |

Each formula installs the released version from PyPI into its own virtualenv. A daily job installs it from this tap on a
clean Homebrew-on-Linux box and runs a real scan, so a broken formula shows up within a day.

sieve is Linux-first: `sieve host` reads dpkg, apk and rpm databases, which a Mac doesn't have. `sieve path`, `sieve sbom`
and `sieve remote` (scan a Linux box over ssh) don't depend on the OS you run them from.
