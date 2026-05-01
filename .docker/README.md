# .docker

This directory contains the Dockerfiles for sp-engine.

---

## Files

| File | Description |
|---|---|
| `Dockerfile` | Standard build using pip |
| `Dockerfile.uv` | Faster build using uv – recommended |

---

## pip vs uv

[uv](https://github.com/astral-sh/uv) is a drop-in replacement for pip written in Rust.
Same commands, same requirements files – just dramatically faster.

| | pip | uv |
|---|---|---|
| Resolver | Python | Rust |
| First build | ~10 min | ~2–3 min |
| Cached build | ~2 min | ~30 sec |
| Lock file support | ❌ | ✅ |

---

## Build

```bash
# Standard
docker build --target final -t karlender/sp-engine:latest -f .docker/Dockerfile .

# With uv – recommended
docker build --target final -t karlender/sp-engine:latest -f .docker/Dockerfile.uv .
```

---

## Multi-Platform with buildx

By default Docker builds for your local architecture only (`amd64` or `arm64`).
To build for multiple platforms at once – for example when building on Apple Silicon
but running on a Linux server – use `buildx`.

### Setup (once)

```bash
# Create a new builder with multi-platform support
docker buildx create --name sp-builder --use

# Verify
docker buildx inspect --bootstrap
```

### Build & Push multi-platform

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --target final \
  -t karlender/sp-engine:latest \
  -t karlender/sp-engine:1.0.0-final \
  -f .docker/Dockerfile.uv \
  --push \
  .
```

> `--push` is required for multi-platform builds – `--load` only supports single platform.

### Build targets

```bash
# Slim – ansible-core + SP collections only
docker buildx build --platform linux/amd64,linux/arm64 --target slim \
  -t karlender/sp-engine:slim -f .docker/Dockerfile.uv --push .

# Final – full image
docker buildx build --platform linux/amd64,linux/arm64 --target final \
  -t karlender/sp-engine:latest -f .docker/Dockerfile.uv --push .

# Dev – everything included
docker buildx build --platform linux/amd64,linux/arm64 --target dev \
  -t karlender/sp-engine:dev -f .docker/Dockerfile.uv --push .
```

---

## Further Reading

- [buildx documentation](https://docs.docker.com/buildx/working-with-buildx/)
- [uv documentation](https://docs.astral.sh/uv/)
- [multi-platform images](https://docs.docker.com/build/building/multi-platform/)

---

## Transparency

This image was built with the assistance of Claude (Anthropic) for Dockerfile structure and documentation. All configurations have been reviewed and
tested manually.
