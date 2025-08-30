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

# Install Sharp with correct musl binaries for alpine
RUN npm install --platform=linuxmusl --arch=x64 sharp@0.30.7

# Install other native modules that might need building
RUN npm rebuild bcrypt node-sass 2>/dev/null || true

# Clean npm cache
RUN npm cache clean --force

# ---- Runtime image ----
FROM node:16-alpine AS runtime

WORKDIR /app

# Install runtime dependencies + Redis (RabbitMQ will be handled differently)
RUN apk add --no-cache \
    vips \
    vips-dev \
    vips-cpp \
    cairo \
    jpeg \
    pango \
    giflib \
    tini \
    curl \
    redis

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

# Environment variables (will be overridden by EasyPanel)
# Removed Redis defaults since using external Redis container
ENV NODE_ENV=production \
    PORT=3010 \
    LOG_LEVEL=info \
    NODE_OPTIONS="--max-old-space-size=2048"

# Enhanced healthcheck using the Docker standard approach
HEALTHCHECK --interval=30s --timeout=10s --start-period=120s --retries=3 \
    CMD curl --silent --fail http://localhost:3010/healthcheck || exit 1

# Switch to non-root user
USER tiledesk

# Use tini for proper signal handling and zombie reaping
ENTRYPOINT ["/sbin/tini", "--"]

# Create supervisor config to manage multiple services
RUN mkdir -p /etc/supervisor.d && \
    echo '[supervisord]' > /etc/supervisor.d/supervisord.conf && \
    echo 'nodaemon=true' >> /etc/supervisor.d/supervisord.conf && \
    echo 'user=root' >> /etc/supervisor.d/supervisord.conf && \
    echo '' >> /etc/supervisor.d/supervisord.conf && \
    echo '[program:redis]' >> /etc/supervisor.d/supervisord.conf && \
    echo 'command=redis-server --port 6379 --bind 127.0.0.1 --save ""' >> /etc/supervisor.d/supervisord.conf && \
    echo 'autostart=true' >> /etc/supervisor.d/supervisord.conf && \
    echo 'autorestart=true' >> /etc/supervisor.d/supervisord.conf && \
    echo 'user=tiledesk' >> /etc/supervisor.d/supervisord.conf && \
    echo '' >> /etc/supervisor.d/supervisord.conf && \
    echo '[program:rabbitmq]' >> /etc/supervisor.d/supervisord.conf && \
    echo 'command=rabbitmq-server' >> /etc/supervisor.d/supervisord.conf && \
    echo 'autostart=true' >> /etc/supervisor.d/supervisord.conf && \
    echo 'autorestart=true' >> /etc/supervisor.d/supervisord.conf && \
    echo 'user=tiledesk' >> /etc/supervisor.d/supervisord.conf && \
    echo 'environment=RABBITMQ_NODENAME=rabbit@localhost' >> /etc/supervisor.d/supervisord.conf && \
    echo '' >> /etc/supervisor.d/supervisord.conf && \
    echo '[program:tiledesk]' >> /etc/supervisor.d/supervisord.conf && \
    echo 'command=node app.js' >> /etc/supervisor.d/supervisord.conf && \
    echo 'directory=/app' >> /etc/supervisor.d/supervisord.conf && \
    echo 'autostart=true' >> /etc/supervisor.d/supervisord.conf && \
    echo 'autorestart=true' >> /etc/supervisor.d/supervisord.conf && \
    echo 'user=tiledesk' >> /etc/supervisor.d/supervisord.conf && \
    echo 'environment=REDIS_URL="%(ENV_REDIS_URL)s",REDIS_HOST="%(ENV_REDIS_HOST)s",REDIS_PORT="%(ENV_REDIS_PORT)s"' >> /etc/supervisor.d/supervisord.conf

# Start the application through script
CMD ["/app/start.sh"]