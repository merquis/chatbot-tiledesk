# Tiledesk Server (chat-api) – build from source
# Use Node.js 16 as required by Tiledesk to avoid Sharp compatibility issues
FROM node:16-alpine AS build

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
    giflib-dev \
    pkgconfig

# Clone official repo
RUN git clone --depth=1 --branch ${TILEDESK_SERVER_REF} https://github.com/Tiledesk/tiledesk-server.git . || \
    git clone --depth=1 https://github.com/Tiledesk/tiledesk-server.git .

# Install dependencies without Sharp to avoid compilation issues
RUN npm install --omit=dev --no-audit --no-fund --ignore-scripts --legacy-peer-deps

# Install Sharp separately with pre-built binaries for alpine
RUN npm install sharp@0.30.7 --platform=linux --arch=x64

# Install other native modules that might need building
RUN npm rebuild bcrypt node-sass 2>/dev/null || true

# Clean npm cache
RUN npm cache clean --force

# ---- Runtime image ----
FROM node:16-alpine AS runtime

WORKDIR /app

# Install only runtime dependencies
RUN apk add --no-cache \
    vips \
    cairo \
    jpeg \
    pango \
    giflib \
    tini

# Create non-root user with proper home directory
RUN addgroup -S tiledesk && adduser -S tiledesk -G tiledesk -h /home/tiledesk

# Copy application files with proper ownership
COPY --from=build --chown=tiledesk:tiledesk /app /app

# Make sure all files have proper permissions
RUN chown -R tiledesk:tiledesk /app && \
    chmod -R 755 /app

# Create logs directory
RUN mkdir -p /app/logs && chown tiledesk:tiledesk /app/logs

# Internal port used by Tiledesk Server
EXPOSE 3010

# Environment variables (these will be overridden by your hosting)
ENV NODE_ENV=production \
    PORT=3010 \
    LOG_LEVEL=info \
    NODE_OPTIONS="--max-old-space-size=2048"

# Enhanced healthcheck
HEALTHCHECK --interval=30s --timeout=10s --start-period=120s --retries=3 \
    CMD node -e "require('http').get('http://localhost:3010/healthcheck', {timeout: 5000}, (r) => {process.exit(r.statusCode === 200 ? 0 : 1)}).on('error', () => process.exit(1))" || exit 1

# Switch to non-root user
USER tiledesk

# Use tini for proper signal handling and zombie reaping
ENTRYPOINT ["/sbin/tini", "--"]

# Start the application
CMD ["node", "app.js"]