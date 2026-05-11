# little-coder Docker Model Runner Sandbox Kit

Docker Sandboxes kit for running [`little-coder`](https://github.com/itayinbarr/little-coder) against a local Docker Model Runner endpoint.

The default model is `llamacpp/qwen3.6-35b-a3b`, backed by a Docker Model Runner model tagged as `qwen3.6-35b-a3b`.

The kit sets `PI_OFFLINE=1` to suppress pi's startup update checks. Inference still goes to Docker Model Runner at `host.docker.internal:12434`.

## Host Setup

Enable Docker Model Runner's TCP endpoint:

```bash
docker desktop enable model-runner --tcp
```

Pull or tag a compatible model. If you already have the Unsloth Qwen3.6-35B-A3B GGUF model, tag it with the short name little-coder expects:

```bash
docker model tag \
  huggingface.co/unsloth/qwen3.6-35b-a3b-gguf:UD-Q6_K_XL \
  qwen3.6-35b-a3b
```

Check that Docker Model Runner resolves it:

```bash
curl http://localhost:12434/v1/models/qwen3.6-35b-a3b
```

## Run From GitHub

Create and run a sandbox in the current directory:

```bash
sbx run --kit git+https://github.com/shelajev/little-coder-sbx-kit.git little-coder-model-runner .
```

Create a named sandbox:

```bash
sbx create --name little-coder-current \
  --kit git+https://github.com/shelajev/little-coder-sbx-kit.git \
  little-coder-model-runner .
```

Run that named sandbox later:

```bash
sbx run --kit git+https://github.com/shelajev/little-coder-sbx-kit.git little-coder-current
```

For local custom agent kits, pass `--kit` again when running an existing sandbox. `sbx` records the custom agent name on the sandbox, but `sbx run <sandbox>` may not know how to resolve a local or Git kit-defined agent unless the kit is supplied again.

## Smoke Test

For a non-interactive `sbx exec` smoke test, close or pipe stdin so pi does not wait for more input:

```bash
sbx exec little-coder-current -- sh -lc \
  'little-coder --no-update-check --model llamacpp/qwen3.6-35b-a3b --thinking off --no-tools --no-session -p "Reply with exactly: sbx-ok" < /dev/null'
```

Expected output includes:

```text
sbx-ok
```

## Local Wrapper

If you clone this repo, `run.sh` runs the named sandbox with the local kit path:

```bash
./run.sh
```

Pass a different sandbox name as the first argument:

```bash
./run.sh my-sandbox
```

## Model Notes

Known useful Docker Model Runner candidates:

- `qwen3.6-35b-a3b`: best default for little-coder; MoE, 34.66B parameters.
- `qwen3.6-27b`: dense fallback; 26.90B parameters.
- Gemma 4 GGUF models can run in Docker Model Runner, but little-coder does not currently ship tuned provider profiles for them.
