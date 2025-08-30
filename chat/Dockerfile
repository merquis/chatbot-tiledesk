# Tiledesk Dashboard (chat) – build Angular app and serve via Nginx
FROM node:22-alpine AS build

ARG TILEDESK_DASHBOARD_REF=master

WORKDIR /app
RUN apk add --no-cache git python3 make g++

# Clone official repo
RUN git clone --depth=1 --branch ${TILEDESK_DASHBOARD_REF} https://github.com/Tiledesk/tiledesk-dashboard.git .

# Install deps & build
RUN npm ci
RUN npm run build

# ---- Runtime ----
FROM nginx:1.27-alpine AS runtime

# Copy build output
COPY --from=build /app/dist /usr/share/nginx/html

# Basic SPA config
RUN printf '%s\n'   'server {'   '  listen 80;'   '  server_name _;'   '  root /usr/share/nginx/html;'   '  index index.html;'   '  location / {'   '    try_files $uri $uri/ /index.html;'   '  }'   '}' > /etc/nginx/conf.d/default.conf

EXPOSE 80
