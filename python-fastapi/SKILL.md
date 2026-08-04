---
name: python-fastapi
description: Construye y mantiene servicios en Python con FastAPI — estructura del proyecto, routers, modelos Pydantic, inyección de dependencias, async correcto, autenticación, manejo de errores y arranque con uvicorn. Úsala al crear o modificar rutas de FastAPI, al organizar un proyecto Python que ha crecido en archivos sueltos, al añadir dependencias compartidas como conexión a base de datos o usuario actual, o cuando aparezcan bloqueos, errores 500 o problemas de importación circular.
---

# FastAPI

## 1. Estructura

Un `main.py` con todo dentro deja de escalar rápido. Estructura mínima que aguanta crecimiento:

```
app/
  main.py            # crea la app, monta routers, middleware
  config.py          # settings desde variables de entorno
  db.py              # conexión, una sola instancia
  dependencies.py    # dependencias compartidas (usuario actual, db)
  routers/
    piezas.py
    ventas.py
    auth.py
  models/            # esquemas Pydantic (entrada/salida)
  services/          # lógica de negocio, sin saber de HTTP
```

Regla que evita la mayoría de los líos: **los routers no contienen lógica de negocio**. Un router valida, delega a un servicio y da forma a la respuesta. Si un endpoint pasa de ~20 líneas, la lógica va a `services/`.

Los servicios **no importan nada de FastAPI**. Así se prueban sin levantar servidor y se reutilizan desde scripts.

## 2. Routers

```python
# routers/piezas.py
from fastapi import APIRouter, Depends, HTTPException
router = APIRouter(prefix="/v1/piezas", tags=["piezas"])

@router.get("/{pieza_id}", response_model=PiezaSalida)
async def obtener(pieza_id: str, usuario=Depends(usuario_actual)):
    pieza = await servicio.obtener(pieza_id, usuario)
    if not pieza:
        raise HTTPException(404, "no encontrada")
    return pieza
```

```python
# main.py
app.include_router(piezas.router)
```

**`response_model` siempre.** No es decorativo: filtra los campos de salida. Sin él, devuelves el documento completo — incluidos hashes y campos internos.

## 3. Pydantic: separa entrada de salida

```python
class PiezaEntrada(BaseModel):
    nombre: str = Field(min_length=1, max_length=200)
    precio: Decimal = Field(gt=0)

class PiezaSalida(BaseModel):
    id: str
    nombre: str
    precio: Decimal
    # sin costo_interno, sin proveedor
```

Usar el mismo modelo para entrada y salida es la vía más rápida a una fuga de datos: el cliente puede enviar campos que no debería poder fijar, y recibe campos que no debería ver.

Pydantic valida en el borde. No repitas esas comprobaciones a mano dentro del endpoint.

## 4. Dependencias

Para lo que se repite en muchas rutas:

```python
# dependencies.py
async def usuario_actual(token: str = Depends(oauth2_scheme)):
    usuario = await verificar(token)
    if not usuario:
        raise HTTPException(401, "credenciales inválidas")
    return usuario

def requiere_rol(rol: str):
    async def check(usuario=Depends(usuario_actual)):
        if rol not in usuario["roles"]:
            raise HTTPException(403, "sin permiso")
        return usuario
    return check
```

Uso: `@router.delete("/{id}", dependencies=[Depends(requiere_rol("admin"))])`

Las dependencias se cachean por petición: `usuario_actual` usado tres veces en la misma request se ejecuta una sola vez.

## 5. Async: el error que más cuesta

**`async def` con una llamada bloqueante dentro congela todo el servidor.** No una petición — todas.

```python
@router.get("/")
async def listar():
    return db.piezas.find()          # PyMongo es SÍNCRONO → bloquea el loop
```

Dos salidas correctas:

```python
# A) usa el driver async
from motor.motor_asyncio import AsyncIOMotorClient
async def listar():
    return await db.piezas.find().to_list(100)

# B) o declara la ruta síncrona — FastAPI la corre en un hilo aparte
def listar():
    return list(db.piezas.find())
```

**Con PyMongo, `requests`, o cualquier librería síncrona: usa `def`, no `async def`.** FastAPI la ejecutará en un threadpool y no bloqueará nada. Mezclar `async def` con librerías síncronas es la causa número uno de "el servidor se cuelga bajo carga".

## 6. Configuración

```python
# config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    mongo_uri: str
    secret_key: str
    debug: bool = False
    class Config:
        env_file = ".env"

settings = Settings()
```

Nada de credenciales en el código. `.env` va en `.gitignore`; deja un `.env.example` sin valores reales.

## 7. Errores

```python
@app.exception_handler(Exception)
async def error_general(request, exc):
    logger.exception("fallo no controlado")
    return JSONResponse(500, {"error": {"code": "interno", "message": "error interno"}})
```

`HTTPException` para errores esperados. El handler global atrapa el resto, **registra la traza en el servidor y devuelve un mensaje genérico**. Nunca envíes la traza al cliente.

**`debug=True` jamás en producción**: expone código fuente y variables en las páginas de error.

## 8. Detalles operativos

**CORS** — solo los orígenes que necesites. `allow_origins=["*"]` junto a `allow_credentials=True` es una combinación insegura y además el navegador la rechaza.

**Conexión a base de datos** — una sola instancia global, creada al arrancar. Abrir un cliente por petición agota conexiones.

**Arranque** — `uvicorn app.main:app --reload` en desarrollo. En producción, sin `--reload`, con varios workers y detrás de un proxy inverso.

**Documentación** — `/docs` es automática. Considera desactivarla en producción si la API no es pública.

**Importación circular** — si `routers/` importa de `main.py`, tendrás ciclos. El flujo va en una dirección: `main` importa routers, los routers importan servicios, los servicios importan modelos y db.

## Relación con otras skills

- Diseño del contrato y seguridad: `backend-api`
- Consultas, índices y modelado: `mongodb`
- Antes de implementar: `test-driven-development`, `design-testing-strategy`
- Al terminar: `review-local-changes`, `verification-before-completion`
- Si se cuelga o falla: `systematic-debugging`, `root-cause-tracing`
- Despliegue: `ci-cd-docker`
