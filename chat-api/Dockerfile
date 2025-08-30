# Tiledesk Server (chat-api) – build from source
# Use Node.js 16.x (LTS version compatible with Tiledesk)
FROM node:16-alpine AS build

ARG TILEDESK_SERVER_REF=master

WORKDIR /app
RUN apk add --no-cache git python3 make g++ \
    && apk add --no-cache libvips

# Clone official repo (pin with TILEDESK_SERVER_REF for stability)
RUN git clone --depth=1 --branch ${TILEDESK_SERVER_REF} https://github.com/Tiledesk/tiledesk-server.git .

# Install production deps
RUN npm ci

# ---- Runtime image ----
FROM node:16-alpine AS runtime
WORKDIR /app

# Create non-root user
RUN addgroup -S nodegroup && adduser -S nodeuser -G nodegroup

COPY --from=build /app /app

# Internal port used by Tiledesk Server
EXPOSE 3000

# Healthcheck (optional)
HEALTHCHECK --interval=30s --timeout=5s --retries=5 CMD wget -qO- http://localhost:3000/ || exit 1

USER nodeuser
CMD ["npm", "start"]
