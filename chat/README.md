# chat (Tiledesk Dashboard) – Easypanel (sin Compose)

## Instrucciones (Easypanel → New App → Dockerfile)
1) Selecciona **Dockerfile** y pega este Dockerfile (o referencia tu repo).
2) En **Environment** pega las variables de `.env.example` (ajusta valores reales).
3) **Internal Port**: 80
4) Asigna dominio **https://chat.topestrellas.com** y activa **SSL**.
5) Deploy. Accede al panel.

## Notas
- `SERVER_API` debe apuntar al **nombre interno** del servicio del server en la red de Easypanel (p. ej. `http://tiledesk-server:3000`). Si usas nombres distintos, actualízalo.
- Para fijar versión del dashboard, cambia `TILEDESK_DASHBOARD_REF` por un tag estable (p. ej. `vX.Y.Z`).
