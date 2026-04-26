# Quick start

You've installed horchd (see [Install](install.md)) and `systemctl
--user status horchd` says it's `active (running)`. Now register your
first wakeword.

## Option A — use a pretrained openWakeWord model

```bash
# Pull a pretrained model into the user models dir
mkdir -p ~/.local/share/horchd/models
oww=$(python -c 'import openwakeword, pathlib; print(pathlib.Path(openwakeword.__file__).parent / "resources/models")')
cp "$oww/hey_jarvis_v0.1.onnx" ~/.local/share/horchd/models/

# Register it with the daemon
horchctl add hey_jarvis --model ~/.local/share/horchd/models/hey_jarvis_v0.1.onnx
horchctl status
```

## Option B — train your own with Lyna

[Lyna](https://github.com/horchd/lyna) is the companion trainer/studio.
Record samples, pick voices, run training, get back `<name>.onnx`. Then:

```bash
cp ~/Downloads/<name>.onnx ~/.local/share/horchd/models/
horchctl add <name> --model ~/.local/share/horchd/models/<name>.onnx --threshold 0.5
```

## Verify

```bash
horchctl monitor
# (speak the wakeword)
```

Each fire prints one line:

```
0.831423   hey_jarvis              ts=12894301527
```

## React to fires from your own script

Subscribe to `xyz.horchd.Daemon1.Detected` on the session bus from any
language that speaks D-Bus. See the [Bash](examples/bash.md),
[Python](examples/python.md), and [Rust](examples/rust.md) subscriber
examples.

## Day-to-day

```bash
horchctl list                                     # tabular view
horchctl threshold hey_jarvis 0.6 --save          # tweak + persist
horchctl disable hey_jarvis --save                # mute without unloading
horchctl enable  hey_jarvis --save
horchctl remove  hey_jarvis                       # keeps the .onnx on disk
horchctl reload                                   # re-read config.toml
journalctl --user -fu horchd                      # live logs
```
