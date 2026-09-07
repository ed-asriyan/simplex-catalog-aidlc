FROM mcr.microsoft.com/devcontainers/base:bookworm

ARG USERNAME=vscode

ENV BUN_INSTALL=/home/${USERNAME}/.bun
ENV PATH=${BUN_INSTALL}/bin:${PATH}

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        unzip \
    && rm -rf /var/lib/apt/lists/*

USER ${USERNAME}

RUN curl -fsSL https://bun.sh/install | bash
