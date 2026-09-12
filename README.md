# HEXPYRION

**Static Nuitka source recovery** — translate Nuitka's native output back into Python. No runtime hooks, no injection; static mode is the default.

By [**DimaReverse**](https://github.com/DimaReverse) — devirt / onefile-unpack / native reconstruction.

> ⚠️ **Authorized use only.** Use this tool only on software you own or are authorized to inspect. It is intended for recovering your own lost source, security research, and malware analysis (Nuitka and Nuitka Commercial are widely used to pack malware precisely because they hamper analysis).

## A note from the author

Yes, I know — there are a lot of repos on my profile, and I'm sorry for opening yet another one. But every decompiler I've ever built is a piece of progress planted in the history of reverse engineering, and I want each of them to be remembered.

This one is my last mark on GitHub, at least for now. I've grown tired of reverse engineering — this is not a goodbye, it's a pause, whether it turns out long or short. I'm not disappearing: you'll find me over on my gaming YouTube channel, [**@dimamilitiagaming**](https://www.youtube.com/@dimamilitiagaming), playing video games instead of dissecting binaries.

HEXPYRION still has some bugs, but it can now genuinely work miracles. If you want to push it further, **open your forks** — and I'll be keeping a kind of leaderboard of the forks with the most stars and the ones that prove most useful.

— [**DimaReverse**](https://github.com/DimaReverse), Venice, Italy

## What it does

HEXPYRION recovers the original Python from Nuitka-compiled binaries by translating Nuitka's native language back into Python:

- **Constants blob** (`mod_consts`) + module table recovery
- **Native x64 of Nuitka helpers** (`LOOKUP_ATTRIBUTE`, `CALL_FUNCTION_*`, `MAKE_FUNCTION_*`, branches, iterators, …) → Python statements
- **`f_lineno` source maps** + structured CFG / register simulation
- Works on **PE / ELF / Mach-O**, standalone and onefile

> Note: this is *not* a CPython `.bytecode` / pycdc-style tool (compiled Nuitka modules are native code, not marshalled `.pyc` — see [pycdc](https://github.com/zrax/pycdc) for that artifact).

## Features

### `--devirt` — static devirtualization (zero execution)
Every marshalled code object in the `.bytecode` chunk is emitted 1:1 as `.pyc` (the original compiled Python bytecode, byte for byte), the `.files` chunk is unpacked, and native modules get constants pools + code-object maps. The binary is never executed.

### `--onefile-unpack` — static onefile unpacking
Statically unpack any Nuitka onefile binary (zstd included) without ever executing the bootstrap.

### `--trigger` — runtime blob capture
Run under gdb, stop **before any module code executes**, dump the *decoded* constants blob from process memory, then kill the process. The payload never runs; the captured blob feeds the same static devirt pipeline. Defeats in-memory and Nuitka Commercial data-hiding blob protection — useful when analyzing Commercial-packed malware.

### NDX engine — full source recovery (primary interface)

**NDX is the most important and useful part of HEXPYRION — the most mature and reliable engine in the tool, and the one that genuinely revolutionizes recovery work.** Give it a compiled binary and it recovers real, readable Python source from it.

Recover every first-party module from a compiled binary:

```bash
python nuitka_decompiler.py --ndx --dev-only main.exe -o OUT
```

Or target specific modules only:

```bash
python nuitka_decompiler.py --ndx --only "pkg.*,__main__,__parents_main__" main.exe -o OUT
```

Key NDX options:

| Option | Meaning |
|---|---|
| `--dev-only` | First-party modules only — bundled libraries are skipped |
| `--only MODS` | Comma-separated glob patterns to filter modules |
| `-o / --output-dir` | Output directory (default `HEXPYRION_OUT`) |
| `--abi-only` | Static ABI discovery only, no source recovery |
| `--target-python X.Y` | Explicit runtime version when embedded metadata is unavailable |
| `--strict` | Exit with code 2 on detectable recovery gaps |
| `-v` | Verbose output |

### Classic pipeline

Whole-binary source emit:

```bash
python nuitka_decompiler.py --source app.dll \
    --emit-all-source out_source --only "pkg.*,__main__,__parents_main__"
```

Devirtualization:

```bash
python nuitka_decompiler.py --source app.exe --devirt --devirt-decompile
python nuitka_decompiler.py --source app.exe --trigger          # runtime blob
```

## Install

```
pip install -r requirements.txt
```

Core requirements: `pefile`, `capstone`, `zstandard`, `xdis`, `uncompyle6`, `decompyle3`. Most third-party imports degrade gracefully if missing; `pefile` is required for PE parsing. Optional: `pylingual` as an additional bytecode-to-source backend.

## Legal

This project is published for interoperability, source recovery, and defensive security research. Reverse engineering for malware analysis and security research serves a protective purpose; do not use it to violate licenses or laws that apply to the binaries you inspect.

## License

MIT — see [LICENSE](LICENSE).
