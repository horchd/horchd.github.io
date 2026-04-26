# horchd

A native Linux daemon that listens to the system microphone, detects
user-defined wakewords in parallel, and broadcasts a D-Bus `Detected`
signal the moment any of them fires. Other apps subscribe and react —
Home Assistant bridges, custom scripts, the bundled `horchctl monitor`,
the (optional) tray app, anything that speaks the session bus.

## Highlights

- **Native** — single Rust binary, no Python at runtime
- **Multi-wakeword** — N openWakeWord-compatible `.onnx` models, run on the same audio
- **Cheap** — ~12 fps inference on a single core; fits on small devices
- **D-Bus first** — no custom protocol, no HTTP listener, no cloud
- **systemd user unit** — no root, no system-bus policy file
- **Trainer-agnostic** — bring `.onnx` from [Lyna](https://github.com/horchd/lyna)
  or any openWakeWord-compatible trainer

## How it works

```
cpal mic 16 kHz mono
  → 80 ms / 1280-sample frames
  → melspectrogram.onnx                  (universal)
  → embedding_model.onnx                 (universal, 96-dim per 80 ms)
  → sliding window of last 16 embeddings (1.28 s receptive field)
  → fan-out to per-wakeword classifier   (input (1, 16, 96), output f32 in [0,1])
  → threshold + cooldown state machine
  → D-Bus Detected(name, score, timestamp_us) signal
```

The 3-stage pipeline (melspec → embedding → classifier) is exactly what
Python `openwakeword.Model.predict()` does internally — horchd ports
that pipeline to Rust + ONNX Runtime so it can run as a tiny daemon
instead of a Python process.

[Install →](install.md) · [Quick start →](quickstart.md) · [D-Bus API →](dbus-api.md)
