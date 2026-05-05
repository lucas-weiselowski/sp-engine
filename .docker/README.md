# .docker

This directory contains the Dockerfile for sp-engine.

---

## Files

| File | Description |
|---|---|
| `Dockerfile` | Multi-stage build using uv – recommended |

---

## Why uv?

[uv](https://github.com/astral-sh/uv) is a drop-in replacement for pip written in Rust.

| | pip | uv |
|---|---|---|
| Resolver | Python | Rust |
| First build | ~10 min | ~2–3 min |
| Cached build | ~2 min | ~30 sec |
| Lock file support | ❌ | ✅ |

---

## Build Targets

| Target | Description |
|---|---|
| `slim` | ansible-core + SSH + Collections |
| `final` | slim + NETCONF + gNMI + RESTCONF/YANG + Parsing |
| `validate` | final + linting + security scanning |
| `ai` | final + Ollama + LangChain + FastMCP + ChromaDB |
| `dev` | everything + pyATS full + testing tools |

---

## Build

```bash
# Single target
docker build --target final -t karlender/sp-engine:latest -f .docker/Dockerfile .
docker build --target slim -t karlender/sp-engine:slim -f .docker/Dockerfile .
docker build --target validate -t karlender/sp-engine:validate -f .docker/Dockerfile .
docker build --target ai -t karlender/sp-engine:ai -f .docker/Dockerfile .
docker build --target dev -t karlender/sp-engine:dev -f .docker/Dockerfile .
```

---

## Multi-Platform with buildx

### Setup (once)

```bash
docker buildx create --name sp-builder --use
docker buildx inspect --bootstrap
```

### Build & Push

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --target final \
  -t karlender/sp-engine:latest \
  -t karlender/sp-engine:1.0.1-final \
  -f .docker/Dockerfile \
  --push \
  .
```

> `--push` is required for multi-platform builds – `--load` only supports single platform.

---

## Further Reading

- [buildx documentation](https://docs.docker.com/buildx/working-with-buildx/)
- [uv documentation](https://docs.astral.sh/uv/)
- [multi-platform images](https://docs.docker.com/build/building/multi-platform/)

---

## Transparency

This image was built with the assistance of Claude (Anthropic) for Dockerfile structure and documentation. All configurations have been reviewed and tested manually.