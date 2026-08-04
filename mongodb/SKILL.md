---
name: mongodb
description: Modela colecciones, escribe consultas y agregaciones, y diseña índices en MongoDB con PyMongo, Motor o el driver de Node. Úsala al crear o cambiar el esquema de documentos, al escribir queries o pipelines de agregación, cuando una consulta esté lenta, al decidir entre incrustar y referenciar, o al manejar ObjectId, transacciones y migraciones de datos.
---

# MongoDB

## 1. La decisión central: incrustar o referenciar

Es la decisión de modelado que más cuesta revertir. Guíate por esto:

**Incrusta** cuando los datos:
- Se leen casi siempre junto al padre
- Pertenecen al padre y no tienen vida propia
- Son acotados — decenas, no miles
- Cambian a la vez que el padre

**Referencia** cuando los datos:
- Se consultan por sí solos
- Los comparten varios documentos
- Crecen sin límite (historial, movimientos, logs)
- Se actualizan con frecuencia distinta al padre

**Límite duro**: un documento no puede superar **16 MB**. Un array que crece sin techo dentro de un documento es un fallo garantizado a futuro, no una posibilidad.

Regla práctica: si dudas, referencia. Pasar de referencia a incrustación es fácil; al revés implica migrar datos en producción.

## 2. Consultas

### Proyecta siempre
```python
db.piezas.find({"activo": True}, {"nombre": 1, "precio": 1, "_id": 0})
```
Traer el documento completo cuando necesitas dos campos desperdicia red y memoria — y filtra datos que no deberían salir.

### Nunca pases entrada del usuario sin validar al objeto de consulta
```python
# MAL — inyección de operadores
db.usuarios.find_one({"usuario": datos["usuario"], "clave": datos["clave"]})
# si datos["clave"] es {"$ne": None}, entra sin contraseña

# BIEN — fuerza el tipo
db.usuarios.find_one({"usuario": str(datos["usuario"])})
# y compara el hash en Python, nunca la contraseña en la query
```
Esto es inyección NoSQL. Es tan real como la de SQL y se pasa por alto mucho más.

### ObjectId
El `_id` es un `ObjectId`, no una cadena. Convierte siempre en el borde, y captura el error:
```python
from bson import ObjectId
from bson.errors import InvalidId
try:
    oid = ObjectId(id_texto)
except InvalidId:
    raise HTTPException(400, "id inválido")
```
Un `ObjectId` inválido lanza excepción — si no la capturas, es un 500 en lugar de un 400.

El `ObjectId` **no es secreto ni aleatorio**: incluye marca de tiempo y es predecible. No lo uses como token ni como clave de acceso.

### Actualiza con operadores, no reemplazando
```python
db.piezas.update_one({"_id": oid}, {"$set": {"precio": 100}})   # BIEN
db.piezas.replace_one({"_id": oid}, {"precio": 100})            # borra el resto
```

Operadores útiles: `$set`, `$inc`, `$push`, `$pull`, `$addToSet`, `$unset`.

Para contadores y stock usa **`$inc`**, nunca leer-sumar-escribir: eso pierde actualizaciones concurrentes.

## 3. Índices

Sin índice, MongoDB recorre la colección entera. Funciona con 100 documentos y cae con 100.000.

**Indexa**: todo campo usado en filtro, orden o join (`$lookup`).

**Índice compuesto — regla ESR**, en este orden:
1. **E**quality — campos con igualdad exacta
2. **S**ort — campos de ordenamiento
3. **R**ange — campos con rango (`$gt`, `$lt`)

```python
db.ventas.create_index([("vendedor", 1), ("fecha", -1)])
```

**Único** para lo que no puede repetirse:
```python
db.usuarios.create_index("correo", unique=True)
```
La restricción va en la base de datos. Comprobar duplicados en código antes de insertar tiene una condición de carrera; el índice único no.

**Diagnóstico**: `db.coleccion.find(...).explain("executionStats")`. Si ves `COLLSCAN` en una consulta frecuente, falta un índice.

## 4. Agregaciones

Orden que importa por rendimiento: **filtra primero, transforma después**.

```python
pipeline = [
    {"$match": {"fecha": {"$gte": inicio}}},   # 1º reduce el conjunto
    {"$group": {"_id": "$vendedor", "total": {"$sum": "$monto"}}},
    {"$sort": {"total": -1}},
    {"$limit": 10},
]
```

Poner `$match` después de `$group` procesa todo para luego descartarlo. `$match` al inicio aprovecha índices; después del primer `$group`, ya no.

## 5. Cosas que muerden

**Transacciones** requieren replica set. En un `mongod` suelto no funcionan. Si necesitas atomicidad multi-documento, planifícalo desde el despliegue.

**Escritura atómica dentro de un documento** sí está garantizada siempre — a menudo eso basta y evita la transacción.

**Fechas**: guarda `datetime` en UTC, no cadenas. Las cadenas no se ordenan ni se filtran por rango correctamente.

**Decimales de dinero**: `float` pierde precisión. Usa `Decimal128` o guarda céntimos como entero.

**Esquema flexible no es ausencia de esquema.** Documenta la forma esperada de cada colección y valídala en la aplicación, o usa validación de esquema de MongoDB.

**`find_one` devuelve `None`** si no hay coincidencia. Comprueba antes de acceder a campos.

## 6. Migraciones

MongoDB no tiene migraciones integradas. Cuando cambies la forma de los documentos:
1. Escribe un script idempotente y versionado en el repositorio
2. Pruébalo contra una copia, nunca directo en producción
3. Haz respaldo antes de ejecutar
4. Soporta ambas formas en el código durante la transición

## Relación con otras skills

- Contratos y seguridad de endpoints: `backend-api`
- Implementación en Python: `python-fastapi`
- Consultas lentas o resultados raros: `systematic-debugging`, `root-cause-tracing`
- Antes de cambiar el modelo de datos: `decision-eval` (es caro de revertir)
