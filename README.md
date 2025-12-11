# pe-signgen

**Cross-version binary signatures and RVA offsets for Windows PE functions**

---

## Overview

`pe-signgen` is a tool for reverse engineers and security researchers that automatically generates:

* **Binary signatures** (byte patterns with wildcards) for **unexported and exported** functions
* **RVA and file offsets** for locating those functions directly in binaries
* Cross-build signatures that work across **many Windows versions**
* Support for multiple **architectures**: x64, ARM64, WoW64

The **core idea of the project** is to provide a **systematic, robust way to access unexported functions** across many Windows 10/11 builds. It leverages:

* [Winbindex](https://github.com/m417z/winbindex) for Windows build metadata
* Microsoft’s public symbol servers for PDBs
* Your local cache for reproducible, offline-friendly workflows

> ⚠️ **Windows version support**
> `pe-signgen` supports **Windows 10 and Windows 11 only**.
> This is a deliberate design choice: Winbindex does not provide complete data for older versions (`< Windows 10`), so this tool does not attempt to support them.

---

## Use Cases

* **Unexported windows internals**
  Generate signatures for functions like `LdrpInitializeTls`, `RtlpInsertInvertedFunctionTableEntry`, etc.
* **Malware analysis**
  Identify API usage in packed or obfuscated binaries.
* **Game hacking / anti-cheat research**
  Generate stable signatures that survive updates.
* **Security research**
  Locate security-critical routines across Windows builds.
* **Automation**
  Scriptable signature and offset generation for entire sets of internal APIs.

---

## Features

### Core Capabilities

* Automatic signature generation with wildcard masking
* Multi-build analysis across Windows 10/11
* Relocation-aware signature generation
* Symbol resolution via PDB + exports
* Smart caching system
* Parallel processing for fast execution
* Cross-platform tooling

### Output Formats

* **Text** – Human-readable console output
* **JSON** – Structured for automation
* **Binary (WSIG/WOFF)** – Compact runtime-ready format
* **C header** – Include-ready output for C/C++

### Architecture Support

* x64 (AMD64)
* ARM64 (AArch64)
* WoW64 (x86 on x64)

---

## Installation

### Prerequisites

* Python 3.8+
* Git
* Internet connection
* Sufficient disk space (first run may download several GB)


### Install from Pip 

```bash
pip install pe-signgen
```



### Install from Source

```bash
git clone https://github.com/forentfraps/pe-signgen.git
cd pe-signgen
pip install -r requirements.txt
pip install -e .
```

### Dependencies

**Python packages**

* `pefile` – PE file parsing
* `winpdb-rs` – PDB symbol resolution
* `requests` – HTTP downloads
* `tqdm` *(optional)* – Progress bars

**External tools**

* `git` – Winbindex updates

---

## Quick Start

### Generate a Signature

```bash
pe-signgen --signature ntdll!LdrLoadDll
```

Example output:

```text
Signature: 4C 8B DC 49 89 5B 08 49 89 6B 10 49 89 73 18 ...
Builds (1247): 10240.16384, 10586.0, ...
```

### Generate Offsets

```bash
pe-signgen --signature kernel32!CreateFileW --offsets
```

### Save as JSON / Binary / Header

```bash
pe-signgen --signature ntdll!NtCreateFile -o out.json --output-format json
pe-signgen --signature ntdll!NtCreateFile -o out.wsig --output-format binary
pe-signgen --signature ntdll!NtCreateFile -o out.h --output-format cheader
```

---

## Usage

### Basic Syntax

```bash
pe-signgen --signature DLL!FUNCTION [OPTIONS]
```

### Architecture

```bash
--arch x64   # default
--arch arm64
--arch wow64
```

### Version Filtering

```bash
--os-version win10
--os-version win11
--min-version 10.0
--max-version 11.0
```

### Signature Length Control

```bash
--min-length 32
--max-length 64
```

### Performance Controls

```bash
--workers 16
--no-cache
--no-git-update
```

### Output Controls

```bash
--verbose
--quiet
--no-progress
```

---

## Examples

### Example: NtCreateFile (x64)

```bash
pe-signgen --signature ntdll!NtCreateFile -o ntcreatefile.json --output-format json
```

Example JSON:

```json
{
  "dll": "ntdll",
  "function": "NtCreateFile",
  "arch": "x64",
  "signatures": {
    "4C 8B DC 49 89 5B 08 49 89 6B 10": [
      "10240.16384",
      "10586.0",
      "14393.0"
    ]
  }
}
```

---

## How It Works

### 1. Symbol Resolution

```text
User Input → Export lookup → PDB lookup → RVA/File offset
```

### 2. Signature Generation

```text
Extract bytes → Apply relocations → Wildcard relocated bytes → Test uniqueness
```

### 3. Multi-Build Processing

```text
Winbindex → Filter builds → Download DLL/PDB → Generate signatures → Group by pattern
```

---

## Caching

### Cache Layout

```text
~/.cache/pe-signgen/
│
├── dlls/
├── pdbs/
├── signatures/
└── winbindex_data/
```

### Cache Control

```bash
--no-cache
rm -rf ~/.cache/pe-signgen
```

### Custom Cache Location

```bash
export PE_SIGNGEN_CACHE=/custom/path
```

---

## Performance

**Example performance (12-core CPU, 100 Mbps):**

| Operation                         | Time        |
| --------------------------------- | ----------- |
| First run (no cache)              | 5–10 min    |
| Cached run                        | < 1 sec     |
| Per-build analysis                | 0.1–0.5 sec |
| Full run (1000 builds, 8 workers) | 2–4 min     |

**Resources:**

* Disk: ~10 GB for full DLL/PDB cache
* Memory: ~500 MB peak
* Network: several GB on first run

---

## Development

```bash
git clone https://github.com/forentfraps/pe-signgen.git
cd pe-signgen
pip install -e .
pip install black mypy
black pe_signgen/
mypy pe_signgen/
```

---

## Known Limitations

* **Windows version coverage:** Only Windows **10 and 11** supported (Winbindex limitation).
* **Build availability:** Not every Win10/11 build exists in Winbindex.
* **Symbol availability:** Missing PDBs may cause skipped builds.
* **Legacy formats:** No support for pre-Win10 systems.

---

## License

MIT License – see [LICENSE](LICENSE).

---

## Credits

* Winbindex – build metadata
* pefile – PE parsing
* winpdb-rs – My PDB parsing python bindings

Inspired by the need for robust signature generation for internal Windows APIs.

---
