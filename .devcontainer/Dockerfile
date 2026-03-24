FROM python:3.12-slim-bookworm

LABEL maintainer="devops"
LABEL description="Ansible + Proxmox dev container for playbook development and execution"

# ── System dependencies ────────────────────────────────────────────────────
RUN apt-get update && apt-get install -y --no-install-recommends \
      openssh-client \
      git \
      curl \
      jq \
      vim \
      bash-completion \
      gnupg2 \
    && rm -rf /var/lib/apt/lists/*

# ── Python dependencies ────────────────────────────────────────────────────
COPY requirements.txt /tmp/requirements.txt
RUN pip install --no-cache-dir -r /tmp/requirements.txt

# ── Ansible collections ────────────────────────────────────────────────────
COPY requirements.yml /tmp/requirements.yml
RUN ansible-galaxy collection install -r /tmp/requirements.yml

# ── Non-root user ──────────────────────────────────────────────────────────
RUN useradd -ms /bin/bash ansible
USER ansible
WORKDIR /workspace

# ── Shell autocomplete ─────────────────────────────────────────────────────
RUN echo 'source /usr/share/bash-completion/bash_completion' >> ~/.bashrc && \
    echo 'complete -W "$(ansible-playbook --list-tasks 2>/dev/null)" ansible-playbook' >> ~/.bashrc

CMD ["/bin/bash"]
