FROM mcr.microsoft.com/devcontainers/base:bookworm

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        ca-certificates \
        gh \
        jq \
        openssh-client \
        python3 \
        python3-pip \
        python3-venv \
        ripgrep \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

ENV GIT_TERMINAL_PROMPT=0

RUN mkdir -p -m 0700 /home/vscode/.ssh \
    && printf 'Host github.com\n  StrictHostKeyChecking accept-new\n' > /home/vscode/.ssh/config \
    && chown -R vscode:vscode /home/vscode/.ssh
