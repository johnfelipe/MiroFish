# MiroFish — Guía de despliegue

Este documento describe cómo instalar, configurar y desplegar MiroFish.
La aplicación tiene dos componentes:

| Componente | Tecnología | Puerto local | Carpeta |
|------------|-----------|--------------|---------|
| Backend (API) | Flask + camel-ai / oasis | `5001` | `backend/` |
| Frontend (SPA) | Vue 3 + Vite | `3000` | `frontend/` |

El frontend habla con el backend a través de `/api/*`. En desarrollo Vite hace
proxy a `http://localhost:5001`; en producción el frontend usa la variable
`VITE_API_BASE_URL` (ver `frontend/src/api/index.js`).

---

## 1. Requisitos

| Herramienta | Versión |
|-------------|---------|
| Node.js | 18+ |
| Python | ≥ 3.11, < 3.13 |
| uv | última |

## 2. Variables de entorno

Copia `.env.example` a `.env` en la raíz y completa:

```env
LLM_API_KEY=...                 # API key de un LLM compatible con el SDK de OpenAI
LLM_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
LLM_MODEL_NAME=qwen-plus
ZEP_API_KEY=...                 # https://app.getzep.com/
```

> El backend **no arranca** sin `LLM_API_KEY` y `ZEP_API_KEY` (ver `backend/app/config.py`).

## 3. Instalación y ejecución local

```bash
npm run setup:all        # instala dependencias (raíz + frontend + backend)
npm run dev              # arranca backend (:5001) y frontend (:3000)
```

---

## 4. Despliegue con Docker

La imagen del repositorio empaqueta ambos servicios:

```bash
cp .env.example .env     # y completa las claves
docker compose up -d     # expone :3000 (frontend) y :5001 (backend)
```

`docker-compose.yml` usa la imagen publicada en GHCR
(`ghcr.io/666ghj/mirofish:latest`), construida por
`.github/workflows/docker-image.yml` al crear un tag.

---

## 5. Despliegue en la nube (Render + Vercel)

### Opción A — Todo en Render (`render.yaml`)

El archivo [`render.yaml`](./render.yaml) define dos servicios:

1. **mirofish-backend** — servicio web Docker (`backend/Dockerfile`).
2. **mirofish-frontend** — sitio estático construido con Vite.

Pasos:

1. En Render: **New → Blueprint** y apunta a este repositorio.
2. Configura los secretos del backend: `LLM_API_KEY`, `ZEP_API_KEY`
   (y ajusta `LLM_BASE_URL` / `LLM_MODEL_NAME` si usas otro proveedor).
3. En el frontend, fija `VITE_API_BASE_URL` con la URL pública del backend
   (ej: `https://mirofish-backend.onrender.com`).

> ⚠️ El backend incluye `torch` y dependencias ML pesadas, por lo que **no cabe
> en el plan free (512 MB)**. Usa un plan con ≥ 2 GB de RAM.

### Opción B — Frontend en Vercel, backend en Render

1. Despliega solo el backend con `render.yaml` (o cualquier host Docker).
2. En Vercel importa el repo con **Root Directory = `frontend`**
   (usa [`frontend/vercel.json`](./frontend/vercel.json)).
3. Añade la variable de entorno `VITE_API_BASE_URL` con la URL del backend.

---

## 6. Notas de producción

- El backend arranca con el servidor de desarrollo de Flask; para alta carga,
  considera un WSGI server. Las simulaciones usan multiprocessing, por lo que el
  servicio debe ser persistente (no serverless).
- Render/Railway inyectan el puerto vía `PORT`; `backend/run.py` ya lo respeta.
- El frontend es estático y puede ir en cualquier CDN/host de sitios estáticos.
