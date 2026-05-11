# little-coder Docker Model Runner Sandbox Kit

Docker Sandboxes kit for running [`little-coder`](https://github.com/itayinbarr/little-coder) against a local Docker Model Runner endpoint.

The kit does not guess which local model you have. You must pass a model explicitly:

```bash
-- --model llamacpp/<docker-model-tag>
```

For example:

```bash
-- --model llamacpp/qwen3.6-35b-a3b-64k
```

If you forget the model, the sandbox fails fast and prints the exact command shape to use.

## Host Setup

Enable Docker Model Runner's TCP endpoint:

```bash
docker desktop enable model-runner --tcp
```

Check your installed local models:

```bash
docker model list
```

Use the model tag from that list after the `llamacpp/` prefix. For example, a Docker Model Runner tag named `qwen3.6-27b` becomes:

```bash
--model llamacpp/qwen3.6-27b
```

## Optional: Qwen 64k Variant

If you already have the Unsloth Qwen3.6-35B-A3B GGUF model under its Hugging Face tag, give it a short local tag:

```bash
docker model tag \
  huggingface.co/unsloth/qwen3.6-35b-a3b-gguf:UD-Q6_K_XL \
  qwen3.6-35b-a3b
```

Create a lightweight 64k-context variant:

```bash
docker model package --from qwen3.6-35b-a3b \
  --context-size 64000 \
  qwen3.6-35b-a3b-64k
```

Verify Docker Model Runner can resolve it:

```bash
curl http://localhost:12434/v1/models/qwen3.6-35b-a3b-64k
```

Then run the kit with:

```bash
-- --model llamacpp/qwen3.6-35b-a3b-64k
```

## Run

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

For custom agent kits, pass `--kit` again when running an existing sandbox. `sbx` records the custom agent name on the sandbox, but `sbx run <sandbox>` may not resolve a Git or local kit-defined agent unless the kit is supplied again.

## Model Selection

The kit ships with a sentinel model value:

```bash
LITTLE_CODER_MODEL_ID=__REQUIRED_MODEL_ID__
```

That sentinel is intentional. Docker Sandboxes kits do not currently have an interactive "ask during install" parameter field, so the public, non-surprising path is to pass the model after the `sbx run ... --` separator.

Any Docker Model Runner tag can be used as a `llamacpp/` model:

```bash
sbx run --kit git+https://github.com/shelajev/little-coder-sbx-kit.git \
  little-coder-model-runner . -- --model llamacpp/<docker-model-tag>
```

The launcher registers unknown `llamacpp/*` model IDs with little-coder at runtime. The default local profile context is `64000`, matching the Qwen 64k packaging example.

## Smoke Test

For `sbx exec`, close or pipe stdin so pi does not wait for more input:

```bash
sbx exec little-coder-current -- sh -lc \
  'little-coder-dmr --model llamacpp/qwen3.6-35b-a3b-64k --thinking off --no-tools --no-session -p "Reply with exactly: sbx-ok" < /dev/null'
```

Expected output includes:

```text
sbx-ok
```

## Local Clone

If you clone this repo, `run.sh` runs a named sandbox with the local kit path. You still need to pass a model:

```bash
./run.sh little-coder-current -- --model llamacpp/qwen3.6-35b-a3b-64k
```

Use another sandbox name as the first argument:

```bash
./run.sh my-sandbox -- --model llamacpp/qwen3.6-27b
```

## Network Policy

The kit allows only:

- `registry.npmjs.org:443` to install `little-coder`
- `host.docker.internal:12434` to reach Docker Model Runner

It sets:

```bash
LLAMACPP_BASE_URL=http://host.docker.internal:12434/v1
LLAMACPP_API_KEY=docker-model-runner
PI_OFFLINE=1
```

`PI_OFFLINE=1` suppresses pi startup update checks. Inference still goes to Docker Model Runner.

## Useful Model Candidates

- `qwen3.6-35b-a3b-64k`: Qwen3.6-35B-A3B 64k Docker Model Runner variant.
- `qwen3.6-35b-a3b`: Qwen3.6-35B-A3B short local tag.
- `qwen3.6-27b`: Qwen3.6 dense fallback.

Use whatever tag is present in `docker model list`.
