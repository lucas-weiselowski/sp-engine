# sp-engine

![Docker Pulls](https://img.shields.io/docker/pulls/karlender/sp-engine)
![Docker Image Size](https://img.shields.io/docker/image-size/karlender/sp-engine/latest)
![GitHub](https://img.shields.io/github/license/lucas-weiselowski/sp-engine)

A batteries-included Python environment for Service Provider network automation.
Built for engineers working with IOS-XR, JunOS, NX-OS and everything in between.

| Tag | What's inside | Use case |
|---|---|---|
| `latest` / `final` | Ansible + NETCONF + gNMI + RESTCONF/YANG + Parsing | Production automation |
| `slim` | ansible-core + SSH + Collections | Quick runs, CI/CD |
| `validate` | final + ruff, flake8, bandit, ansible-lint, yamllint | Pipeline linting & security scanning |
| `ai` | final + Ollama + LangChain + OpenAI + FastMCP + ChromaDB | LLM agent workflows |
| `dev` | Everything + pyATS full + ipython + testing tools | Local development |

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

Drop this in your `.bashrc` or `.zshrc`:

```bash
alias sp='docker run -it --rm -v $(pwd):/workspace karlender/sp-engine:latest'
alias sp-slim='docker run -it --rm -v $(pwd):/workspace karlender/sp-engine:slim'
alias sp-validate='docker run -it --rm -v $(pwd):/workspace karlender/sp-engine:validate'
alias sp-ai='docker run -it --rm -v $(pwd):/workspace karlender/sp-engine:ai'
alias sp-dev='docker run -it --rm -v $(pwd):/workspace karlender/sp-engine:dev'
```

Then just:

```bash
sp ansible-playbook playbooks/deploy_bgp.yml
sp python3 scripts/check_peers.py
sp-slim ansible all -m ping
sp-validate ruff check .
sp-validate bandit -r .
```

## GitLab CI

```yaml
default:
  image: karlender/sp-engine:latest

lint:
  image: karlender/sp-engine:validate
  script:
    - ruff check .
    - bandit -r .
    - ansible-lint playbooks/

deploy:bgp:
  script:
    - ansible-playbook playbooks/deploy_bgp.yml
```

## GitHub Actions

```yaml
jobs:
  validate:
    runs-on: ubuntu-latest
    container:
      image: karlender/sp-engine:validate
    steps:
      - uses: actions/checkout@v4
      - run: ruff check .
      - run: bandit -r .
      - run: ansible-lint playbooks/

  deploy:
    runs-on: ubuntu-latest
    container:
      image: karlender/sp-engine:latest
    steps:
      - uses: actions/checkout@v4
      - run: ansible-playbook playbooks/deploy_bgp.yml
```

## What's included

**Connectivity** (all images)
- netmiko, napalm, paramiko, ncclient, scrapli, scrapli-netconf
- pygnmi, grpcio – gNMI/gRPC streaming telemetry
- pyang, pyangbind – RESTCONF/YANG tooling
- httpx, lxml, xmltodict – RESTCONF & XML parsing

**Ansible** (all images)
- ansible-core with cisco.ios, cisco.iosxr, cisco.nxos
- junipernetworks.junos, ansible.netcommon, ansible.utils
- community.network, netbox.netbox

**Parsing** (final, validate, ai, dev)
- ttp, textfsm, ntc-templates – template-based parsing
- genie, pyats – Cisco model-driven parsing

**Validate** (validate, dev)
- ruff, flake8, bandit, ansible-lint, yamllint
- mypy, black, pre-commit, commitizen

**AI** (ai, dev)
- Ollama + LangChain + langchain-openai + langchain-anthropic
- OpenAI + Anthropic SDK
- FastMCP – MCP server/client
- ChromaDB – RAG vector store
- FastAPI + uvicorn – automation APIs

**Dev** (dev only)
- pyATS full, pytest, ipython, ipdb
- All validate tools included

**Infrastructure** (all images)
- OpenTofu – Infrastructure as Code
- Nornir + nornir-netmiko + nornir-napalm

---

## Transparency

This image was built with the assistance of Claude (Anthropic) for Dockerfile structure and documentation. All configurations have been reviewed and tested manually.

---

Built with ❤️ for SP engineers tired of setting up Python environments.  
Source: [github.com/lucas-weiselowski/sp-engine](https://github.com/lucas-weiselowski/sp-engine)  
Docker Hub: [karlender/sp-engine](https://hub.docker.com/r/karlender/sp-engine)