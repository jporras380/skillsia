---
name: ci-cd-docker
description: Empaqueta aplicaciones con Docker y automatiza pruebas y despliegue con CI/CD (GitHub Actions). Úsala al escribir un Dockerfile o docker-compose, al crear o arreglar un workflow de CI, al preparar una aplicación para producción, al gestionar variables de entorno y secretos entre entornos, o cuando algo funcione en local pero falle en el servidor.
---

# Docker y CI/CD

## 1. Dockerfile

Plantilla para Python. Los principios aplican a cualquier lenguaje:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

# Dependencias primero: esta capa se reutiliza si no cambia requirements.txt
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# El código después: cambia a menudo, invalida solo esta capa
COPY . .

# Usuario sin privilegios
RUN useradd -m app && chown -R app:app /app
USER app

EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

Cuatro reglas que resuelven la mayoría de problemas:

**Dependencias antes que código.** Si copias todo de una vez, cada cambio de una línea reinstala todo. Esta separación convierte builds de minutos en builds de segundos.

**Nunca como root.** Un contenedor comprometido corriendo como root es mucho peor.

**`--host 0.0.0.0`.** Con `127.0.0.1` el servicio solo escucha dentro del contenedor y no responde desde fuera. Es la causa más frecuente de "el contenedor arranca pero no puedo conectarme".

**Imagen base concreta.** `python:3.12-slim`, no `python:latest`. `latest` cambia bajo tus pies y rompe builds sin que toques nada.

### `.dockerignore`

Sin esto, `COPY . .` mete `.git`, `.venv`, `__pycache__`, `node_modules` y tus `.env`:

```
.git
.venv
__pycache__
*.pyc
.env
node_modules
uploads/
```

Copiar un `.env` dentro de la imagen deja los secretos ahí para siempre, incluso si borras el archivo en una capa posterior.

## 2. docker-compose para desarrollo

```yaml
services:
  api:
    build: .
    ports: ["8000:8000"]
    environment:
      MONGO_URI: mongodb://db:27017/miapp
    env_file: .env
    depends_on:
      db: { condition: service_healthy }
    volumes:
      - .:/app          # recarga en caliente; quítalo en producción

  db:
    image: mongo:7
    volumes: [mongo_data:/data/db]
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 10s
      retries: 5

volumes:
  mongo_data:
```

**Servicios se hablan por nombre**, no por `localhost`. Desde `api`, la base de datos es `db:27017`. Dentro de un contenedor, `localhost` es ese contenedor.

**Volumen con nombre para los datos.** Sin él, `docker compose down` borra la base de datos.

**`depends_on` con healthcheck.** Sin la condición, `depends_on` solo espera a que el contenedor arranque, no a que el servicio esté listo — y la app falla al conectar.

## 3. CI con GitHub Actions

```yaml
name: CI
on:
  push: { branches: [main] }
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      mongo:
        image: mongo:7
        ports: ["27017:27017"]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: pip
      - run: pip install -r requirements.txt
      - run: ruff check .
      - run: pytest -v
```

Orden deliberado: **lo rápido primero**. Linter antes que pruebas — falla en segundos en lugar de minutos.

La CI debe fallar de verdad. Un workflow que siempre pasa en verde es peor que no tenerlo: da confianza falsa.

## 4. Secretos

| Entorno | Dónde van |
|---|---|
| Local | `.env`, en `.gitignore` |
| CI | Secrets del repositorio |
| Producción | Variables del proveedor o gestor de secretos |

Nunca: en el Dockerfile, en el compose, ni en el repositorio.

Si un secreto llegó a git, **rótalo**. Borrarlo del historial no basta — ya está en clones, forks y cachés.

## 5. Antes de considerar algo listo para producción

- [ ] `.dockerignore` existe y excluye `.env`, `.git`, `.venv`
- [ ] El contenedor no corre como root
- [ ] Imagen base con versión fija
- [ ] Ningún secreto en la imagen ni en el repositorio
- [ ] Datos persistentes en volumen con nombre
- [ ] Sin `--reload` ni `debug=True`
- [ ] Healthcheck definido
- [ ] Política de reinicio (`restart: unless-stopped`)
- [ ] Logs a stdout/stderr, no a un archivo dentro del contenedor
- [ ] La CI corre las pruebas y puede fallar

## 6. Diagnóstico

| Síntoma | Causa habitual |
|---|---|
| Arranca pero no responde | Escucha en `127.0.0.1`, no en `0.0.0.0` |
| No conecta a la base | Usa `localhost` en vez del nombre del servicio |
| Datos se pierden al reiniciar | Falta volumen con nombre |
| Build lentísimo | Código copiado antes que dependencias |
| Funciona local, falla en CI | Depende de un archivo ignorado, o de estado local |
| Imagen enorme | Falta `.dockerignore`, o base no `slim` |

Comandos: `docker compose logs -f api` · `docker compose exec api sh` · `docker compose ps`

## Relación con otras skills

- Preparar la app antes de empaquetar: `python-fastapi`, `backend-api`
- Qué se ejecuta en CI: `design-testing-strategy`, `test-coverage`
- Fallos en el pipeline: `systematic-debugging`, `root-cause-tracing`
- Decidir proveedor u orquestador: `decision-eval`
