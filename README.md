# sp-engine

![Docker Pulls](https://img.shields.io/docker/pulls/karlender/sp-engine)
![Docker Image Size](https://img.shields.io/docker/image-size/karlender/sp-engine/latest)
![GitHub](https://img.shields.io/github/license/lucas-weiselowski/sp-engine)

A batteries-included Python environment for Service Provider network automation.
Built for engineers working with IOS-XR, JunOS, NX-OS and everything in between.

Comes in three flavours depending on what you need:

| Tag | What's inside | Size |
|---|---|---|
| `latest` / `final` | Python + Ansible + NETCONF + gNMI + AI | ~276MB |
| `slim` | ansible-core + SP collections only | ~88MB |
| `dev` | Everything + pyATS + linting + testing tools | ~600MB |

---

## Quickstart

```bash
# Pull
docker pull karlender/sp-engine:latest

# Run interactively
docker run -it --rm karlender/sp-engine:latest

# Mount your playbooks
docker run -it --rm \
  -v $(pwd):/workspace \
  karlender/sp-engine:latest \
  ansible-playbook playbooks/deploy.yml
```

## Set an alias

Drop this in your `.bashrc` or `.zshrc` and forget Docker is even involved:

```bash
# Full image
alias sp='docker run -it --rm -v $(pwd):/workspace karlender/sp-engine:latest'

# Slim – for quick Ansible runs
alias sp-slim='docker run -it --rm -v $(pwd):/workspace karlender/sp-engine:slim'

# Dev – with all the tools
alias sp-dev='docker run -it --rm -v $(pwd):/workspace karlender/sp-engine:dev'
```

Then just:

```bash
sp ansible-playbook playbooks/deploy_bgp.yml
sp python3 scripts/check_peers.py
sp-slim ansible all -m ping
```

## GitLab CI

```yaml
default:
  image: karlender/sp-engine:latest

deploy:bgp:
  script:
    - ansible-playbook playbooks/deploy_bgp.yml
```

## GitHub Actions

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    container:
      image: karlender/sp-engine:latest
    steps:
      - uses: actions/checkout@v4
      - run: ansible-playbook playbooks/deploy_bgp.yml
```

## What's included

**Connectivity**
- netmiko, napalm, paramiko, ncclient, scrapli
- pygnmi, grpcio – for gNMI/gRPC streaming telemetry
- RESTCONF via httpx + YANG tooling (pyang, pyangbind)

**Ansible**
- ansible-core with cisco.ios, cisco.iosxr, cisco.nxos
- junipernetworks.junos, ansible.netcommon, ansible.utils
- community.network, netbox.netbox

**AI / Parsing** (final + dev only)
- Ollama client + LangChain for local LLM workflows
- FastAPI for building automation APIs
- ChromaDB for RAG-based config analysis

**Dev** (dev only)
- pyATS full, ansible-lint, pytest, black, ruff, mypy
- ipython, pre-commit, commitizen

---

## Transparency

This image was built with the assistance of Claude (Anthropic) for Dockerfile structure and documentation. All configurations have been reviewed and
tested manually.

---

Built with ❤️ for SP engineers tired of setting up Python environments. \
Source: [github.com/lucas-weiselowski/sp-engine](https://github.com/lucas-weiselowski/sp-engine)
Docker Hub: [karlender/sp-engine](https://hub.docker.com/r/karlender/sp-engine)