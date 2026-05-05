# ============================================================
# STAGE 1 – Builder Slim
# ============================================================
FROM python:3.12-slim AS builder-slim

COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv

RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    apt-get update && apt-get upgrade -y && apt-get install -y --no-install-recommends \
    build-essential \
    libssl-dev \
    libffi-dev

RUN --mount=type=cache,target=/root/.cache/uv \
    uv pip install --prefix=/install --system \
        "ansible-core>=2.17" \
        "paramiko>=3.4" \
        "netaddr>=1.3" \
        "netutils>=1.9" \
        "jmespath>=1.0" \
        "passlib>=1.7" \
        "bcrypt>=4.1" \
        "cryptography>=42.0" \
        "PyYAML>=6.0" \
        "Jinja2>=3.1" \
        "resolvelib>=1.0"


# ============================================================
# STAGE 2 – Builder Full (no AI)
# ============================================================
FROM python:3.12-slim AS builder-full

ENV PIP_DISABLE_PIP_VERSION_CHECK=1

RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    apt-get update && apt-get upgrade -y && apt-get install -y --no-install-recommends \
    build-essential \
    libssl-dev \
    libffi-dev \
    libxml2-dev \
    libxslt1-dev \
    git

COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv

WORKDIR /build

COPY requirements/ requirements/

RUN --mount=type=cache,target=/root/.cache/uv \
    uv pip install --prefix=/install --system \
        -r requirements/base.txt \
        -r requirements/netconf.txt


# ============================================================
# STAGE 3 – Builder Validate
# ============================================================
FROM python:3.12-slim AS builder-validate

COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv

WORKDIR /build

COPY requirements/validate.txt requirements/validate.txt

RUN --mount=type=cache,target=/root/.cache/uv \
    uv pip install --prefix=/install --system \
        -r requirements/validate.txt


# ============================================================
# STAGE 4 – Builder AI
# ============================================================
FROM python:3.12-slim AS builder-ai

COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv

RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    apt-get update && apt-get upgrade -y && apt-get install -y --no-install-recommends \
    build-essential

WORKDIR /build

COPY requirements/ai.txt requirements/ai.txt

RUN --mount=type=cache,target=/root/.cache/uv \
    uv pip install --prefix=/install --system \
        -r requirements/ai.txt


# ============================================================
# STAGE 5 – Builder OpenTofu
# ============================================================
FROM debian:bookworm-slim AS builder-tofu

ARG TOFU_VERSION=1.11.6
ARG TARGETARCH

RUN apt-get update && apt-get upgrade -y && apt-get install -y --no-install-recommends \
    curl \
    ca-certificates \
    unzip \
    && rm -rf /var/lib/apt/lists/*

RUN curl -fsSL \
    "https://github.com/opentofu/opentofu/releases/download/v${TOFU_VERSION}/tofu_${TOFU_VERSION}_linux_${TARGETARCH}.zip" \
    -o /tmp/tofu.zip \
    && unzip /tmp/tofu.zip -d /tmp/tofu \
    && install -m 0755 /tmp/tofu/tofu /usr/local/bin/tofu \
    && rm -rf /tmp/tofu*

RUN test -x /usr/local/bin/tofu && echo "tofu OK"


# ============================================================
# STAGE 6 – Builder Ollama
# ============================================================
FROM debian:bookworm-slim AS builder-ollama

ARG OLLAMA_VERSION=0.23.0
ARG TARGETARCH

RUN apt-get update && apt-get upgrade -y && apt-get install -y --no-install-recommends \
    curl \
    ca-certificates \
    zstd \
    tar \
    && rm -rf /var/lib/apt/lists/*

RUN curl -fsSL \
    "https://github.com/ollama/ollama/releases/download/v${OLLAMA_VERSION}/ollama-linux-${TARGETARCH}.tar.zst" \
    -o /tmp/ollama.tar.zst \
    && mkdir -p /tmp/ollama-extract \
    && tar -xf /tmp/ollama.tar.zst -C /tmp/ollama-extract \
    && install -m 0755 /tmp/ollama-extract/bin/ollama /usr/local/bin/ollama \
    && rm -rf /tmp/ollama*

RUN test -x /usr/local/bin/ollama && echo "ollama OK"


# ============================================================
# STAGE 7 – Slim
# ============================================================
FROM python:3.12-slim AS slim

LABEL maintainer="lucas-weiselowski"
LABEL org.opencontainers.image.title="sp-engine-slim"
LABEL org.opencontainers.image.source="https://github.com/lucas-weiselowski/sp-engine"

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    ANSIBLE_HOST_KEY_CHECKING=False \
    ANSIBLE_COLLECTIONS_PATHS=/usr/share/ansible/collections

RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    apt-get update && apt-get upgrade -y && apt-get install -y --no-install-recommends \
    openssh-client \
    sshpass \
    libssl3 \
    libffi8

COPY --from=builder-slim /install /usr/local
COPY --from=builder-tofu /usr/local/bin/tofu /usr/local/bin/tofu

COPY collections.yml /collections.yml
RUN ansible-galaxy collection install -r /collections.yml \
    -p /usr/share/ansible/collections \
    && rm /collections.yml

WORKDIR /workspace
CMD ["bash"]


# ============================================================
# STAGE 8 – Final (Prod)
# ============================================================
FROM python:3.12-slim AS final

LABEL maintainer="lucas-weiselowski"
LABEL org.opencontainers.image.title="sp-engine"
LABEL org.opencontainers.image.source="https://github.com/lucas-weiselowski/sp-engine"
LABEL org.opencontainers.image.licenses="MIT"

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    ANSIBLE_HOST_KEY_CHECKING=False \
    ANSIBLE_COLLECTIONS_PATHS=/usr/share/ansible/collections

RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    apt-get update && apt-get upgrade -y && apt-get install -y --no-install-recommends \
    openssh-client \
    sshpass \
    libssl3 \
    libffi8 \
    libxml2 \
    libxslt1.1 \
    curl \
    git \
    iputils-ping \
    iproute2 \
    traceroute

COPY --from=builder-full /install /usr/local
COPY --from=builder-tofu /usr/local/bin/tofu /usr/local/bin/tofu

COPY collections.yml /collections.yml
RUN ansible-galaxy collection install -r /collections.yml \
    -p /usr/share/ansible/collections \
    && rm /collections.yml

WORKDIR /workspace
CMD ["bash"]


# ============================================================
# STAGE 9 – Validate
# ============================================================
FROM final AS validate

LABEL org.opencontainers.image.title="sp-engine-validate"

COPY --from=builder-validate /install /usr/local

WORKDIR /workspace
CMD ["bash"]


# ============================================================
# STAGE 10 – AI
# ============================================================
FROM final AS ai

LABEL org.opencontainers.image.title="sp-engine-ai"

ENV OLLAMA_MODELS=/workspace/models \
    OLLAMA_HOST=0.0.0.0:11434

COPY --from=builder-ai /install /usr/local
COPY --from=builder-ollama /usr/local/bin/ollama /usr/local/bin/ollama

WORKDIR /workspace
CMD ["bash"]


# ============================================================
# STAGE 11 – Dev (Everything)
# ============================================================
FROM python:3.12-slim AS dev

LABEL org.opencontainers.image.title="sp-engine-dev"

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    ANSIBLE_HOST_KEY_CHECKING=False \
    ANSIBLE_COLLECTIONS_PATHS=/usr/share/ansible/collections \
    OLLAMA_MODELS=/workspace/models \
    OLLAMA_HOST=0.0.0.0:11434

RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    apt-get update && apt-get upgrade -y && apt-get install -y --no-install-recommends \
    openssh-client \
    sshpass \
    libssl3 \
    libffi8 \
    libxml2 \
    libxslt1.1 \
    curl \
    git \
    vim \
    iputils-ping \
    iproute2 \
    traceroute

COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv
COPY --from=builder-full /install /usr/local
COPY --from=builder-validate /install /usr/local
COPY --from=builder-ai /install /usr/local
COPY --from=builder-ollama /usr/local/bin/ollama /usr/local/bin/ollama
COPY --from=builder-tofu /usr/local/bin/tofu /usr/local/bin/tofu

COPY collections.yml /collections.yml
RUN ansible-galaxy collection install -r /collections.yml \
    -p /usr/share/ansible/collections \
    && rm /collections.yml

COPY requirements/ requirements/

RUN --mount=type=cache,target=/root/.cache/uv \
    uv pip install --system \
        -r requirements/dev.txt \
    && rm -rf requirements/

WORKDIR /workspace
CMD ["bash"]