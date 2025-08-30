# Tiledesk Server (chat-api)
FROM node:16-alpine AS build
ARG TILEDESK_SERVER_REF=v2.9.0
WORKDIR /app

# deps de build + vips-dev (sharp)
RUN apk add --no-cache \
  git python3 make g++ vips-dev build-base cairo-dev jpeg-dev pango-dev giflib-dev pkgconfig

# código
RUN git clone --depth=1 --branch ${TILEDESK_SERVER_REF} https://github.com/Tiledesk/tiledesk-server.git . \
 || git clone --depth=1 https://github.com/Tiledesk/tiledesk-server.git .

# deps (sin scripts) y sharp correcto para musl
RUN npm install --omit=dev --no-audit --no-fund --ignore-scripts --legacy-peer-deps
RUN npm_config_platform=linuxmusl npm_config_arch=x64 npm install sharp@0.30.7

# nativos opcionales
RUN npm rebuild bcrypt node-sass 2>/dev/null || true
RUN npm cache clean --force

# -------- runtime
FROM node:16-alpine AS runtime
WORKDIR /app

# libs runtime + curl + tini
RUN apk add --no-cache vips cairo jpeg pango giflib curl tini

# usuario no-root y permisos
RUN addgroup -S tiledesk && adduser -S tiledesk -G tiledesk -h /home/tiledesk
COPY --from=build --chown=tiledesk:tiledesk /app /app
RUN mkdir -p /app/logs && chown -R tiledesk:tiledesk /app && chmod -R 755 /app

EXPOSE 3010
ENV NODE_ENV=production \
    PORT=3010 \
    LOG_LEVEL=info \
    NODE_OPTIONS="--max-old-space-size=2048"

HEALTHCHECK --interval=30s --timeout=10s --start-period=120s --retries=3 \
  CMD curl -fsS http://localhost:3010/healthcheck || exit 1

USER tiledesk
ENTRYPOINT ["/sbin/tini","--"]
CMD ["node","app.js"]
