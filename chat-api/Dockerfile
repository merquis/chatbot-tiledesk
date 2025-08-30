# Tiledesk Server (chat-api) – build from source
# Use Node.js 18.x (recommended for latest Tiledesk)
FROM node:18-alpine AS build

# Use a specific stable version instead of master
ARG TILEDESK_SERVER_REF=v2.9.0

WORKDIR /app

# Install build dependencies including vips for image processing
RUN apk add --no-cache \
    git \
    python3 \
    make \
    g++ \
    vips-dev \
    build-base \
    cairo-dev \
    jpeg-dev \
    pango-dev \
    giflib-dev

# Clone official repo
RUN git clone --depth=1 --branch ${TILEDESK_SERVER_REF} https://github.com/Tiledesk/tiledesk-server.git . \
    || git clone --depth=1 https://github.com/Tiledesk/tiledesk-server.git .

# Install all dependencies (including dev for building native modules)
RUN npm ci || npm install

# ---- Runtime image ----
FROM node:18-alpine AS runtime

WORKDIR /app

# Install runtime dependencies
RUN apk add --no-cache \
    vips \
    cairo \
    jpeg \
    pango \
    giflib \
    && apk add --no-cache --virtual .gyp python3 make g++ \
    && apk del .gyp

# Create non-root user
RUN addgroup -S nodegroup && adduser -S nodeuser -G nodegroup

# Copy application from build stage
COPY --from=build --chown=nodeuser:nodegroup /app /app

# Copy node_modules separately to ensure permissions
COPY --from=build --chown=nodeuser:nodegroup /app/node_modules /app/node_modules

# Internal port used by Tiledesk Server
EXPOSE 3010

# Environment variables (these will be overridden by Easypanel)
ENV NODE_ENV=production \
    PORT=3010

# Healthcheck
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=5 \
    CMD node -e "require('http').get('http://localhost:3010/healthcheck', (r) => {r.statusCode === 200 ? process.exit(0) : process.exit(1)})" || exit 1

USER nodeuser

# Use node directly instead of npm start for better signal handling
CMD ["node", "app.js"]