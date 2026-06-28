# msedit — max-kepler modified build (Arch/Manjaro packaging)

Personal modification of [Microsoft Edit](https://github.com/microsoft/edit)
living on the [`mk`](https://github.com/max-kepler/edit/tree/mk) branch of the
fork. Currently ships one patch on top of upstream:

- **Escape exits the editor** (with the standard save prompt if there are
  unsaved changes), without hijacking Esc from search, modals, menus, the
  file picker, or the status bar.

This `packaging` branch holds only Arch Linux `PKGBUILD`s — no source code.

## Packages

| Package | Source | Compiles? | Use when |
|---|---|---|---|
| `msedit-mod-bin` | prebuilt tarball from the rolling [`nightly`](https://github.com/max-kepler/edit/releases/tag/nightly) GitHub Release | no — installs in seconds | default, any x86_64 Arch/Manjaro |
| `msedit-mod-git` | `git` branch `mk` (compiled with Rust) | yes | when you want a source build / no prebuilt binary |

Both `provides=(msedit)` and conflict with each other and with upstream
`msedit` / `msedit-git` — install exactly one.

## Install (one line)

```sh
# prebuilt (recommended):
curl -fsSL https://raw.githubusercontent.com/max-kepler/edit/packaging/msedit-mod-bin/PKGBUILD -o PKGBUILD && makepkg -si

# from source:
curl -fsSL https://raw.githubusercontent.com/max-kepler/edit/packaging/msedit-mod-git/PKGBUILD -o PKGBUILD && makepkg -si
```

Prereqs on a fresh Manjaro: `sudo pacman -S --needed base-devel git icu zstd`
(`rust` too for the `-git` variant).

## Update

- `msedit-mod-bin`: `makepkg -f` (the `nightly` asset is rebuilt automatically
  by CI on every push to `mk`; `sha256sums` is `SKIP` for rolling robustness —
  see the PKGBUILD header for enabling strict verification).
- `msedit-mod-git`: `makepkg -f` (re-clones `mk` and rebuilds).

Remove: `sudo pacman -R msedit-mod-bin` (or `msedit-mod-git`).

## How the prebuilt binary is produced

Push to `mk` triggers `.github/workflows/release.yml` in the fork: it builds
`cargo build --release --locked` on `ubuntu-latest`, packs the binary + manpage
+ `.desktop` + icon + license into `msedit-x86_64.tar.zst`, computes sha256,
and (re)publishes the rolling `nightly` GitHub Release. The `-bin` package
fetches that asset, so no compiler is needed on the target machine.