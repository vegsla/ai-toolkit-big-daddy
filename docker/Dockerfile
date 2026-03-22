FROM nvidia/cuda:12.8.1-devel-ubuntu24.04

LABEL authors="vegsla"

ENV DEBIAN_FRONTEND=noninteractive
ENV PYTHONDONTWRITEBYTECODE=1
ENV PIP_NO_CACHE_DIR=1
ENV TORCH_CUDA_ARCH_LIST="8.0 8.6 8.9 9.0 10.0 12.0"

# System dependencies
RUN apt-get update && apt-get install --no-install-recommends -y \
    git \
    curl \
    build-essential \
    cmake \
    wget \
    python3.12 \
    python3-pip \
    python3-dev \
    python3-setuptools \
    python3-wheel \
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

# Node.js (for UI build)
WORKDIR /tmp
RUN curl -sL https://deb.nodesource.com/setup_23.x -o nodesource_setup.sh && \
    bash nodesource_setup.sh && \
    apt-get update && \
    apt-get install -y nodejs && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Make python3 default
RUN ln -s /usr/bin/python3 /usr/bin/python

# Install PyTorch (CUDA 12.8 compatible build)
RUN pip install --no-cache-dir \
    torch==2.9.1 torchvision==0.24.1 torchaudio==2.9.1 \
    --index-url https://download.pytorch.org/whl/cu128 \
    --break-system-packages

# Clone your repo
ARG AITK_REPO=https://github.com/vegsla/ai-toolkit-big-daddy.git
ARG AITK_REF=main

RUN git clone --depth 1 --branch ${AITK_REF} ${AITK_REPO} /app/ai-toolkit

WORKDIR /app/ai-toolkit

# Install Python dependencies
COPY requirements.txt /app/ai-toolkit/requirements.txt

RUN pip install --no-cache-dir --use-deprecated=legacy-resolver -r requirements.txt && \
    pip install setuptools==69.5.1 --no-cache-dir --break-system-packages

# Build UI
WORKDIR /app/ai-toolkit/ui
RUN npm install && \
    npm run build && \
    npm run update_db

# Expose RunPod port
EXPOSE 8675

WORKDIR /

# Copy startup script (MUST EXIST in your repo)
COPY docker/start.sh /start.sh
RUN chmod +x /start.sh

# Start the app
CMD ["/start.sh"]
