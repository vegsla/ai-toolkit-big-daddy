FROM node:22

WORKDIR /app/ai-toolkit/ui

COPY package*.json ./

RUN npm install

COPY . .

RUN npx tsc -p tsconfig.worker.json \
    && npm run build \
    \
    # ✅ Fix nested cron folder
    && if [ -f dist/cron/cron/worker.js ]; then \
        echo "Fixing nested cron folder..." && \
        mkdir -p dist/cron && \
        mv dist/cron/cron/* dist/cron/ && \
        rm -rf dist/cron/cron ; \
    fi \
    \
    # ✅ FIX WRONG SRC STRUCTURE
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

COPY start.sh /start.sh
RUN chmod +x /start.sh

CMD ["/start.sh"]
