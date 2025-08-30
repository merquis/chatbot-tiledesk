# Tiledesk Server (chat-api) desde la imagen oficial
FROM tiledesk/tiledesk-server:v2.13.0

# Tiledesk expone 3000 en la imagen oficial
EXPOSE 3000

# Defaults sanos; EasyPanel los puede sobreescribir en runtime
ENV NODE_ENV=production \
    PORT=3000 \
    LOG_LEVEL=info

# Healthcheck sin curl (usa Node), compatible con cualquier base
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
  CMD node -e "const http=require('http');const p=process.env.PORT||3000;http.get({host:'127.0.0.1',port:p,path:'/healthcheck',timeout:5000},r=>process.exit(r.statusCode===200?0:1)).on('error',()=>process.exit(1))" || exit 1
