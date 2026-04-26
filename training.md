# Train a wakeword

horchd does not train models. Training is the trainer's job — horchd
loads `.onnx` files and runs them. Two paths:

## Lyna (recommended)

[Lyna](https://github.com/horchd/lyna) is the companion trainer + studio
built specifically for the horchd ecosystem. Record samples, pick TTS
voices for synthetic training data, run training, and export
`<name>.onnx` (plus optional `<name>.onnx.data` external weights file).

Drop the resulting file under `~/.local/share/horchd/models/`, then:

```bash
horchctl add <name> --model ~/.local/share/horchd/models/<name>.onnx
```

## Upstream openWakeWord

[`openwakeword`](https://github.com/dscripka/openWakeWord) (the Python
package horchd ports the inference path from) ships a training pipeline
of its own and a set of pretrained models. The pretrained ones live
inside the package install:

```bash
oww=$(python -c 'import openwakeword, pathlib; print(pathlib.Path(openwakeword.__file__).parent / "resources/models")')
ls "$oww"/*.onnx
```

You'll see `alexa_v0.1.onnx`, `hey_jarvis_v0.1.onnx`,
`hey_mycroft_v0.1.onnx`, `hey_rhasspy_v0.1.onnx`, `weather_v0.1.onnx`,
`timer_v0.1.onnx`. Each one drops straight into horchd:

```bash
cp "$oww/hey_jarvis_v0.1.onnx" ~/.local/share/horchd/models/
horchctl add hey_jarvis --model ~/.local/share/horchd/models/hey_jarvis_v0.1.onnx
```

For a custom wakeword via openwakeword's training pipeline see [its
training docs](https://github.com/dscripka/openWakeWord#training-new-models).
The output is a `.onnx` file with input shape `(N, 16, 96)` and output
`(N, 1)` — exactly what horchd's classifier loader expects.

## Validation

When you `horchctl add ...`, the daemon loads the `.onnx` and validates
the shape before accepting it. If you see something like:

```
Error: classifier "lyna" at /…/lyna.onnx has shape [N, 12, 96] -> [N, 1],
expected (N, 16, 96) -> (N, 1) — was this model trained for openWakeWord?
```

then the model isn't a 16-frame openWakeWord classifier. Re-export it
from your trainer with the right window size.
