FROM nvidia/cuda:12.8.1-devel-ubuntu24.04

ENV DEBIAN_FRONTEND=noninteractive
ENV PYTHONDONTWRITEBYTECODE=1
ENV PIP_NO_CACHE_DIR=1
ENV VENV_PATH=/opt/venv
ENV PATH="/opt/venv/bin:${PATH}"

ARG AITK_REPO=https://github.com/vegsla/ai-toolkit-big-daddy.git
ARG AITK_REF=main

# System deps
RUN apt-get update && apt-get install --no-install-recommends -y \
    git \
    curl \
    build-essential \
    cmake \
    python3.12 \
    python3-pip \
    python3-dev \
    python3-venv \
    ffmpeg \
    tmux \
    htop \
    nvtop \
    python3-opencv \
    openssh-client \
    openssh-server \
    openssl \
    rsync \
    unzip \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Node
RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
    && apt-get update \
    && apt-get install --no-install-recommends -y nodejs \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Python venv
RUN python3 -m venv "${VENV_PATH}" \
    && python -m pip install --upgrade pip setuptools wheel

# Clone repo
RUN git clone --depth 1 --branch "${AITK_REF}" "${AITK_REPO}" /app/ai-toolkit

WORKDIR /app/ai-toolkit

# Install torch FIRST
RUN python -m pip install --no-cache-dir \
    torch==2.7.0 \
    torchvision==0.22.0 \
    torchaudio==2.7.0 \
    --index-url https://download.pytorch.org/whl/cu126

# Install rest WITHOUT dependency resolver chaos
RUN python -m pip install --no-cache-dir --no-deps -r requirements.txt

# ---------- UI BUILD ----------
WORKDIR /app/ai-toolkit/ui

RUN npm install \
    \
    # 🔥 FORCE TYPESCRIPT BUILD (this is the key fix)
    && npx tsc -p tsconfig.worker.json \
    \
    # Build Next.js
    && npm run build \
    \
    # Fix possible nested cron folder
    && if [ -f dist/cron/cron/worker.js ]; then \
         echo "Fixing nested cron folder..." && \
         mkdir -p dist/cron && \
         mv dist/cron/cron/* dist/cron/ && \
         rm -rf dist/cron/cron ; \
       fi \
    \
    # 🔍 HARD DEBUG (we will KNOW what's there)
    && echo "==== FINAL DIST STRUCTURE ====" \
    && ls -R dist || true \
    && echo "=============================" \
    \
    # DB setup
    && npm run update_db

# ---------- START ----------
WORKDIR /

COPY start.sh /start.sh
RUN chmod +x /start.sh

EXPOSE 22 8675

CMD ["/start.sh"]
