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

## Install

This uses the standard Arch/AUR workflow — `git clone` the package dir, `cd`
into it, `makepkg -si`. The cloned dir IS the build dir, so `src/` and `pkg/`
land inside it, not in your home folder.

```sh
# prebuilt (recommended, no compiler needed):
git clone --depth 1 -b packaging https://github.com/max-kepler/edit.git msedit-mod-bin && \
  cd msedit-mod-bin/msedit-mod-bin && makepkg -si

# from source (compiles with Rust):
git clone --depth 1 -b packaging https://github.com/max-kepler/edit.git msedit-mod-git && \
  cd msedit-mod-git/msedit-mod-git && makepkg -si
```

If you have an AUR helper, the last step can be `yay -Bi .` (or `paru -Ui .`)
instead of `makepkg -si` — it wraps makepkg and resolves deps for you.

Prereqs on a fresh Manjaro: `sudo pacman -S --needed base-devel git icu zstd`
(`rust` too for the `-git` variant).

## Update

Keep the cloned dir around and refresh it in place:

```sh
cd msedit-mod-bin/msedit-mod-bin   # (or msedit-mod-git/msedit-mod-git)
git pull                            # pull fresh PKGBUILD from the packaging branch
makepkg -f                          # bin: re-fetches the latest `nightly` asset;
                                    # git: re-clones `mk` and rebuilds
sudo pacman -U msedit-mod-*.pkg.tar.zst
```

(`makepkg -si` also works in place of the last two lines.)

`msedit-mod-bin`'s `sha256sums` is `SKIP` for rolling robustness — the `nightly`
asset is rebuilt automatically by CI on every push to `mk`. See the PKGBUILD
header for enabling strict verification.

Remove: `sudo pacman -R msedit-mod-bin` (or `msedit-mod-git`).

## How the prebuilt binary is produced

Push to `mk` triggers `.github/workflows/release.yml` in the fork: it builds
`cargo build --release --locked` on `ubuntu-latest`, packs the binary + manpage
+ `.desktop` + icon + license into `msedit-x86_64.tar.zst`, computes sha256,
and (re)publishes the rolling `nightly` GitHub Release. The `-bin` package
fetches that asset, so no compiler is needed on the target machine.