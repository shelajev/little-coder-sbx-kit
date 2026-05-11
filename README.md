# little-coder Docker Model Runner Sandbox Kit

Docker Sandboxes kit for running [`little-coder`](https://github.com/itayinbarr/little-coder) against a local Docker Model Runner endpoint.

The kit does not assume a default local model. Pass the Docker Model Runner tag explicitly with `--model`, for example `llamacpp/qwen3.6-35b-a3b-64k`.

The kit sets `PI_OFFLINE=1` to suppress pi's startup update checks. Inference still goes to Docker Model Runner at `host.docker.internal:12434`.

## Host Setup

Enable Docker Model Runner's TCP endpoint:

```bash
docker desktop enable model-runner --tcp
```

Pull or tag a compatible model. If you already have the Unsloth Qwen3.6-35B-A3B GGUF model, tag it with a short name:

```bash
docker model tag \
  huggingface.co/unsloth/qwen3.6-35b-a3b-gguf:UD-Q6_K_XL \
  qwen3.6-35b-a3b
```

Create a 64k-context lightweight variant:

```bash
docker model package --from qwen3.6-35b-a3b \
  --context-size 64000 \
  qwen3.6-35b-a3b-64k
```

Check that Docker Model Runner resolves it:

```bash
curl http://localhost:12434/v1/models/qwen3.6-35b-a3b-64k
```

## Run From GitHub

Create and run a sandbox in the current directory:

```bash
sbx run --kit git+https://github.com/shelajev/little-coder-sbx-kit.git \
  little-coder-model-runner . -- --model llamacpp/qwen3.6-35b-a3b-64k
```

Create a named sandbox:

```bash
sbx create --name little-coder-current \
  --kit git+https://github.com/shelajev/little-coder-sbx-kit.git \
  little-coder-model-runner .
```

Run that named sandbox later:

```bash
sbx run --kit git+https://github.com/shelajev/little-coder-sbx-kit.git \
  little-coder-current -- --model llamacpp/qwen3.6-35b-a3b-64k
```

For local custom agent kits, pass `--kit` again when running an existing sandbox. `sbx` records the custom agent name on the sandbox, but `sbx run <sandbox>` may not know how to resolve a local or Git kit-defined agent unless the kit is supplied again.

## Model Selection

The kit deliberately ships with a sentinel model value:

```bash
LITTLE_CODER_MODEL_ID=__REQUIRED_MODEL_ID__
LITTLE_CODER_CONTEXT_LIMIT=64000
```

If no model is provided, the sandbox fails fast with the exact command to run. Docker Sandboxes kits currently do not have an interactive "ask during install" parameter field, so the supported public path is to pass `--model` after the `sbx run ... --` separator.

Use any Docker Model Runner tag by passing it as a `llamacpp/` model:

```bash
sbx run --kit git+https://github.com/shelajev/little-coder-sbx-kit.git \
  little-coder-model-runner . -- --model llamacpp/<docker-model-tag>
```

The launcher registers unknown `llamacpp/*` model IDs with little-coder at runtime, using `LITTLE_CODER_CONTEXT_LIMIT` for the local profile context size.

## Smoke Test

For a non-interactive `sbx exec` smoke test, close or pipe stdin so pi does not wait for more input:

```bash
sbx exec little-coder-current -- sh -lc \
  'little-coder --no-update-check --model llamacpp/qwen3.6-35b-a3b-64k --thinking off --no-tools --no-session -p "Reply with exactly: sbx-ok" < /dev/null'
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

- `qwen3.6-35b-a3b-64k`: best default for this kit; MoE, 34.66B parameters, 64k Docker Model Runner context variant.
- `qwen3.6-35b-a3b`: original 32k little-coder profile; MoE, 34.66B parameters.
- `qwen3.6-27b`: dense fallback; 26.90B parameters.
- Gemma 4 GGUF models can run in Docker Model Runner, but little-coder does not currently ship tuned provider profiles for them.
