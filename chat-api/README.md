# chat-api (Tiledesk Server) – Easypanel (sin Compose)

## Instrucciones (Easypanel → New App → Dockerfile)
1) Selecciona **Dockerfile** y pega este Dockerfile (o referencia tu repo).
2) En **Environment** pega las variables de `.env.example` (ajusta valores reales).
3) **Internal Port**: 3000
4) Asigna dominio **https://chat-api.topestrellas.com** y activa **SSL**.
5) Deploy. Comprueba `/` o `/status` (si disponible).

## Notas
- El contenedor expone **3000 interno**. El puerto externo final lo gestiona el proxy de Easypanel (HTTPS 443).
- Asegúrate de que `MONGODB_URI` y `REDIS_HOST/PORT` apunten a tus servicios existentes.
- Si quieres fijar versión del server, cambia `TILEDESK_SERVER_REF` por un tag estable (p. ej. `vX.Y.Z`).

