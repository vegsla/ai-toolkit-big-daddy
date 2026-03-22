FROM node:22

WORKDIR /app/ai-toolkit/ui

# =========================
# Install dependencies
# =========================
COPY ui/package*.json ./
RUN npm install

# =========================
# Copy UI source
# =========================
COPY ui/ ./

# =========================
# Copy start script
# =========================
COPY start.sh /start.sh
RUN chmod +x /start.sh

# =========================
# Build process
# =========================
RUN npx tsc -p tsconfig.worker.json \
    && npm run build \
    \
    # ✅ Fix nested cron folder (if it exists)
    && if [ -f dist/cron/cron/worker.js ]; then \
        echo "Fixing nested cron folder..." && \
        mkdir -p dist/cron && \
        mv dist/cron/cron/* dist/cron/ && \
        rm -rf dist/cron/cron ; \
    fi \
    \
    # ✅ Fix src path structure (real bug you hit)
    && if [ -d dist/cron/src ]; then \
        echo "Fixing src path for runtime imports..." && \
        mkdir -p dist && \
        mv dist/cron/src dist/src ; \
    fi \
    \
    && echo "==== FINAL DIST STRUCTURE ====" \
    && ls -R dist || true \
    && echo "=============================" \
    \
    && npm run update_db

# =========================
# Runtime
# =========================
CMD ["/start.sh"]
