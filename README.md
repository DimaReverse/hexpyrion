# HEXPYRION

**Static Nuitka source recovery** — translate Nuitka's native output back into Python. No runtime hooks, no injection; static mode is the default.

By **dimareverse** — devirt / onefile-unpack / native reconstruction.

> ⚠️ **Authorized use only.** Use this tool only on software you own or are authorized to inspect. It is intended for recovering your own lost source, security research, and malware analysis (Nuitka and Nuitka Commercial are widely used to pack malware precisely because they hamper analysis).

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

### Whole-binary source emit
```bash
py nuitka_decompiler.py --source app.dll \
    --emit-all-source out_source --only "pkg.*,__main__,__parents_main__"
```

### Typical devirtualization
```bash
py nuitka_decompiler.py --source app.exe --devirt --devirt-decompile
py nuitka_decompiler.py --source app.exe --trigger          # runtime blob
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
