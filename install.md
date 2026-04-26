# Install

## Source install (any distro with cargo)

```bash
git clone https://github.com/horchd/horchd
cd horchd

# Drop the universal preprocessing models from openwakeword
oww=$(python -c 'import openwakeword, pathlib; print(pathlib.Path(openwakeword.__file__).parent / "resources/models")')
cp "$oww"/{melspectrogram,embedding_model}.onnx shared-models/

./packaging/install.sh
```

`install.sh` builds release binaries, installs them to `/usr/local/bin/`,
copies the shared models to `/usr/local/share/horchd/`, scaffolds
`~/.config/horchd/config.toml` and the user data dir
`~/.local/share/horchd/models/`, then enables + starts the systemd user
unit.

Verify:

```bash
systemctl --user status horchd
horchctl status
```

## Arch / CachyOS (AUR)

A `PKGBUILD` ships under `packaging/arch/`. Once published to the AUR:

```bash
yay -S horchd                   # or paru / makepkg
```

The AUR package depends on `pipewire`. After install, fetch the shared
models the same way as the source path (the package leaves them up to
the user to keep the build hermetic):

```bash
sudo install -Dm644 "$oww/melspectrogram.onnx"  /usr/share/horchd/melspectrogram.onnx
sudo install -Dm644 "$oww/embedding_model.onnx" /usr/share/horchd/embedding_model.onnx
```

Then enable the unit and reload the config:

```bash
systemctl --user enable --now horchd
horchctl reload
```

## Build dependencies

If you build from source you need:

- `rustc` 1.85+ (Rust 2024 edition)
- `cargo`
- A working PipeWire (or PulseAudio) install — cpal handles both
- libc, glibc — already on every Linux desktop

`ort` will download a matching ONNX Runtime binary on first build (with
the `download-binaries` feature, which is on by default for horchd).
