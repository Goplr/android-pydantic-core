# 📱 Android Pydantic Core

[![Build & Release](https://img.shields.io/github/actions/workflow/status/Goplr/android-pydantic-core/build_wheels.yml?label=Build)](https://github.com/Goplr/android-pydantic-core/actions/build_wheels.yml)
[![Python Versions](https://img.shields.io/badge/python-3.9%20%7C%203.10%20%7C%203.11%20%7C%203.12%20%7C%203.13%20%7C%203.14-blue)](https://github.com/Goplr/android-pydantic-core/releases)
[![Architectures](https://img.shields.io/badge/arch-arm64%20%7C%20armv7%20%7C%20x86%20%7C%20x86__64-orange)](https://github.com/Goplr/android-pydantic-core/releases)

**Automated builds of `pydantic-core` optimized for Android (Termux).**

Compiling `pydantic-core` on Android requires a Rust toolchain and takes ~15 minutes (or fails due to memory). This repository provides pre-built wheels that install instantly via `pip`.

## 📦 Supported Targets

| Architecture | Device Type | Status |
|--------------|-------------|--------|
| `aarch64` | Modern Smartphones | ✅ Supported |
| `armv7` | Older Devices | ✅ Supported |
| `x86_64` | Emulators / Chromebooks | ✅ Supported |
| `x86` | Old Emulators | ✅ Supported |

> **Python Versions:** 3.9 – 3.14 candidates, auto-pruned per release — see [Python version support](#-python-version-support)

---

## 🚀 Installation

### ⚡ Option 1: Quick Install (Script)
Use this if you want the installer to **auto-detect** your architecture and Python version.

```bash
curl -sL https://raw.githubusercontent.com/Goplr/android-pydantic-core/main/install_pydantic_core.sh | bash
```

### 🐍 Option 2: Pip (Standard)
Best for requirements files or CI/CD.

```bash
pip install pydantic-core --extra-index-url https://Goplr.github.io/android-pydantic-core/
```

### 📦 Option 3: Manual Download
You can manually download the `.whl` files from the [Releases Page](https://github.com/Goplr/android-pydantic-core/releases).

1. Download the file matching your Python version (`cp312`) and Architecture (`aarch64`). On Python 3.13+ the filename also has an `.android_{api}_{abi}` segment — that's expected, see [Wheel tags & Android compatibility](#-wheel-tags--android-compatibility).
2. Install it:
   ```bash
   pip install pydantic_core-*.whl
   ```

---

## 🛠️ How it works

This repository uses **GitHub Actions** to cross-compile wheels using the Android NDK r25b.

0.  **Version selection:** by default the workflow builds whatever is latest on PyPI. To build a specific version instead (e.g. `2.46.5`), either set `TARGET_VERSION` in the `env:` block at the top of [`build_wheels.yml`](.github/workflows/build_wheels.yml), or run the workflow manually (`Actions` → `Android Wheels & Release` → `Run workflow`) and fill in the `target_version` input — the manual input always wins.
1.  **Checks PyPI** for new versions daily.
2.  **Resolves which Python versions to build** by checking the pinned `pydantic-core` release's own `requires_python` metadata on PyPI, and skipping any candidate Python version (from `ALL_PYTHON_VERSIONS` in `build_wheels.yml`) that release doesn't support — see [Python version support](#-python-version-support).
3.  **Cross-compiles** using `maturin` and patched linker flags:
    *   **RPATH Fix:** Hardcodes Termux library paths (`/data/data/com.termux/files/usr/lib`) so the linker finds `libpython`.
    *   **Force Needed:** Uses `--no-as-needed` to ensure `libpython` dependencies are correctly recorded.
4.  **Renames** artifacts to `linux_{arch}` for Termux compatibility (Python 3.13+ wheels also get a second `android_{api}_{abi}` tag — see [Wheel tags & Android compatibility](#-wheel-tags--android-compatibility) below).
5.  **Publishes** wheels to GitHub Releases and updates the PEP 503 Index.

## 🐍 Python version support

`pydantic-core` has periodically raised its minimum Python version — for example, `2.48.0` requires Python ≥3.10, dropping the Python 3.9 support that `2.46.5` still had. Building from source for a Python version a given `pydantic-core` release no longer supports just fails the job, so before building, the `check-version` job:

1. Reads the pinned/resolved `pydantic-core` version's `requires_python` field straight from PyPI's JSON API.
2. Filters `ALL_PYTHON_VERSIONS` (the candidate list defined in `build_wheels.yml`) down to whichever versions satisfy that floor.
3. Passes the filtered list into the build job's matrix, so unsupported Python versions are skipped entirely rather than failing.

You'll see this reflected in the `check-version` job's logs (`Building for: [...]` / `Skipping unsupported Python versions: ...`) and in each release's notes, which list the Python versions actually built. To add a new Python version once `pydantic-core` supports it, add it to `ALL_PYTHON_VERSIONS` at the top of `build_wheels.yml` — no other changes needed.

## 🏷️ Wheel tags & Android compatibility

`pip` decides whether a wheel is installable purely by matching its filename tags against what your interpreter reports as compatible — it doesn't inspect the binary. Older Termux Python builds self-identify as plain Linux, so a `linux_{arch}` tag was enough. Starting with Python 3.13, CPython has [official Android support](https://peps.python.org/pep-0738/) (PEP 738), and some Termux 3.13+ builds self-identify as Android instead, which makes `pip` reject a `linux_{arch}`\-only wheel with `... is not a supported wheel on this platform` (tracked as [issue #4](https://github.com/Eutalix/android-pydantic-core/issues/4)).

Wheel filenames can carry more than one platform tag separated by dots (the same mechanism `manylinux` wheels use), and `pip`/`uv` install the wheel if *any* tag matches. So for Python 3.13+ this project tags each wheel with both:

```
pydantic_core-2.46.5-cp313-cp313-linux_aarch64.android_24_arm64_v8a.whl
```

This is purely additive — it never removes the legacy `linux_{arch}` tag, so it can't break installs that already work, and it adds compatibility for Android-self-identifying builds. Python 3.9-3.12 predate PEP 738 entirely, so those wheels keep the single legacy `linux_{arch}` tag.

If you still see `is not a supported wheel on this platform`, check exactly what tag your `pip` wants:
```bash
python -c "import sysconfig; print(sysconfig.get_platform())"
```
and compare it against the tags in the release asset name.

## 🤝 Credits

- [pydantic](https://github.com/pydantic/pydantic-core)

License: MIT
