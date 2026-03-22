FROM node:22

WORKDIR /app/ai-toolkit/ui

# Copy UI dependencies from the correct location
COPY ui/package*.json ./

RUN npm install

# Copy the rest of the UI
COPY ui/ ./

# Bring in start script (this is what broke before)
COPY start.sh /start.sh

RUN chmod +x /start.sh

# Build
RUN npx tsc -p tsconfig.worker.json \
    && npm run build \
    \
    && if [ -f dist/cron/cron/worker.js ]; then \
        echo "Fixing nested cron folder..." && \
        mkdir -p dist/cron && \
        mv dist/cron/cron/* dist/cron/ && \
        rm -rf dist/cron/cron ; \
    fi \
    \
    && if [ -d dist/cron/src ]; then \
        echo "Fixing src path..." && \
        mkdir -p dist && \
        mv dist/cron/src dist/src ; \
    fi \
    \
    && echo "==== FINAL DIST STRUCTURE ====" \
    && ls -R dist || true \
    && echo "=============================" \
    \
    && npm run update_db

CMD ["/start.sh"]
