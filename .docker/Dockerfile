# ============================================================
# Stage 1 – Builder Full
# ============================================================
FROM python:3.12-slim AS builder-full

ENV PIP_NO_CACHE_DIR=0 \
    PIP_DISABLE_PIP_VERSION_CHECK=1 \
    DOCKER_BUILDKIT=1

RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    apt-get update -y && apt-get install -y --no-install-recommends \
    build-essential \
    libssl-dev \
    libffi-dev \
    libxml2-dev \
    libxslt1-dev \
    git

WORKDIR /build

COPY requirements/ requirements/

RUN --mount=type=cache,target=/root/.cache/pip \
    pip install --prefix=/install \
        -r requirements/base.txt \
        -r requirements/netconf.txt \
        -r requirements/restconf.txt \
        -r requirements/ai.txt


# ============================================================
# Stage 2 – Builder Slim
# ============================================================
FROM python:3.12-slim AS builder-slim

ENV PIP_NO_CACHE_DIR=0 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    libssl-dev \
    libffi-dev

WORKDIR /build

RUN --mount=type=cache,target=/root/.cache/pip \
    pip install --prefix=/install \
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
# Stage 3 – Builder Tofu
# ============================================================
FROM debian:bookworm-slim AS builder-tofu

ARG TOFU_VERSION=1.9.0
ARG TARGETARCH

RUN apt-get update && apt-get install -y --no-install-recommends \
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

RUN test -x /usr/local/bin/tofu && echo "tofu binary OK"


# ============================================================
# Stage 4 – Builder Ollama
# ============================================================
FROM debian:bookworm-slim AS builder-ollama

ARG OLLAMA_VERSION=0.6.5
ARG TARGETARCH

RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    ca-certificates \
    tar \
    && rm -rf /var/lib/apt/lists/*

RUN mkdir -p /tmp/ollama-extract \
    && curl -fsSL \
    "https://github.com/ollama/ollama/releases/download/v${OLLAMA_VERSION}/ollama-linux-${TARGETARCH}.tgz" \
    -o /tmp/ollama.tgz \
    && tar -xzf /tmp/ollama.tgz -C /tmp/ollama-extract \
    && install -m 0755 /tmp/ollama-extract/bin/ollama /usr/local/bin/ollama \
    && rm -rf /tmp/ollama*

RUN test -x /usr/local/bin/ollama && echo "ollama binary OK"


# ============================================================
# Stage 5 – Slim
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
    apt-get update && apt-get install -y --no-install-recommends \
    openssh-client \
    sshpass

COPY --from=builder-slim /install /usr/local
COPY --from=builder-tofu /usr/local/bin/tofu /usr/local/bin/tofu

COPY collections.yml /collections.yml
RUN ansible-galaxy collection install -r /collections.yml \
    -p /usr/share/ansible/collections \
    && rm /collections.yml

WORKDIR /workspace
CMD ["bash"]


# ============================================================
# Stage 6 – Final
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
    apt-get update && apt-get install -y --no-install-recommends \
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
# Stage 7 – AI
# ============================================================
FROM final AS ai

LABEL org.opencontainers.image.title="sp-engine-ai"

COPY --from=builder-ollama /usr/local/bin/ollama /usr/local/bin/ollama

ENV OLLAMA_MODELS=/workspace/models \
    OLLAMA_HOST=0.0.0.0:11434

COPY requirements/ai.txt requirements/ai.txt

RUN --mount=type=cache,target=/root/.cache/pip \
    pip install --no-cache-dir \
        -r requirements/ai.txt \
    && rm -rf requirements/

WORKDIR /workspace
CMD ["bash"]


# ============================================================
# Stage 8 – Dev
# ============================================================
FROM final AS dev

LABEL org.opencontainers.image.title="sp-engine-dev"

COPY --from=builder-ollama /usr/local/bin/ollama /usr/local/bin/ollama

ENV OLLAMA_MODELS=/workspace/models \
    OLLAMA_HOST=0.0.0.0:11434

RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    apt-get update && apt-get install -y --no-install-recommends \
    vim \
    iputils-ping \
    iproute2 \
    traceroute

COPY requirements/ requirements/

RUN --mount=type=cache,target=/root/.cache/pip \
    pip install --no-cache-dir \
        -r requirements/dev.txt \
        -r requirements/testing.txt \
        -r requirements/ai.txt \
    && rm -rf requirements/

WORKDIR /workspace
CMD ["bash"]