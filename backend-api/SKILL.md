---
name: backend-api
description: Diseña e implementa APIs de backend correctas y seguras — modelado de recursos, validación de entrada, autenticación y sesiones, manejo de errores, paginación, idempotencia y versionado. Úsala al crear o modificar endpoints, rutas, controladores o servicios HTTP, al añadir login/registro/permisos, o cuando aparezcan problemas de autorización, fugas de datos o contratos de API inconsistentes.
---

# Diseño e implementación de APIs

Aplica esto **antes** de escribir el primer endpoint, y al revisar cualquier ruta existente.

## 1. Contrato antes que código

Define para cada endpoint, en una línea cada uno:
- **Recurso y acción** — `POST /piezas` crea una pieza
- **Entrada** — campos, tipos, cuáles son obligatorios
- **Salida** — forma exacta de la respuesta en éxito
- **Errores** — qué códigos puede devolver y por qué
- **Quién puede llamarlo** — rol o permiso requerido

Si no puedes escribir esas cinco líneas, el endpoint no está diseñado.

## 2. Reglas no negociables

### Validar toda entrada en el borde
Nunca confíes en el cliente. Valida tipo, rango, longitud y formato **antes** de que el dato toque la lógica de negocio. Usa el validador del framework (Pydantic, Zod, class-validator) — no `if` sueltos.

### Nunca construyas queries por concatenación
Parámetros ligados, siempre. Esto aplica a SQL **y** a NoSQL — un objeto de consulta armado con entrada del usuario sin filtrar es inyección igual.

### Autorizar por objeto, no solo por ruta
Comprobar "está autenticado" no basta. Comprueba que **este** usuario puede tocar **este** recurso. El fallo más común en producción es una ruta protegida que devuelve el registro de otro usuario si cambias el ID.

### Nunca devuelvas más de lo necesario
Define explícitamente qué campos salen. Serializar el documento completo filtra hashes de contraseña, campos internos y datos de otros usuarios. Lista blanca, no lista negra.

### Secretos fuera del código
Variables de entorno, siempre. Nada de claves, tokens ni cadenas de conexión en el repositorio.

## 3. Errores

Devuelve el código correcto — el cliente decide su comportamiento con él:

| Código | Cuándo |
|---|---|
| 400 | Entrada mal formada |
| 401 | No autenticado (falta o expiró credencial) |
| 403 | Autenticado pero sin permiso |
| 404 | No existe — **o existe pero no es tuyo** |
| 409 | Conflicto de estado (duplicado, versión obsoleta) |
| 422 | Bien formado pero semánticamente inválido |
| 429 | Demasiadas peticiones |
| 500 | Fallo del servidor |

Dos reglas sobre el cuerpo del error:
- **Forma consistente** en toda la API: `{"error": {"code": "...", "message": "...", "details": [...]}}`
- **Nunca expongas trazas de pila ni mensajes de base de datos** al cliente. Registra el detalle en el servidor, devuelve un identificador de correlación.

En login, usa el **mismo mensaje** para usuario inexistente y contraseña incorrecta. Distinguirlos permite enumerar cuentas.

## 4. Autenticación y sesiones

- Contraseñas con **bcrypt o argon2**, nunca SHA/MD5, nunca sin sal
- Token o cookie de sesión con **expiración**; sin caducidad no es sesión, es una llave permanente
- Cookies de sesión: `HttpOnly`, `Secure`, `SameSite=Lax` como mínimo
- **Límite de intentos** en login y recuperación de contraseña
- Al cambiar contraseña o cerrar sesión, **invalida las sesiones existentes**
- Guarda el rol/permiso del lado servidor, nunca lo leas de un campo enviado por el cliente

## 5. Detalles que se olvidan

**Paginación** — toda lista puede crecer. Devuelve página y total, o cursor. Un endpoint que devuelve "todos los registros" es una bomba de tiempo.

**Idempotencia** — `PUT` y `DELETE` deben poder repetirse sin efecto adicional. En `POST` que cobran o crean, acepta una clave de idempotencia.

**Versionado** — prefijo `/v1/` desde el primer día. Añadirlo después obliga a romper clientes.

**Transacciones** — si una operación toca varios registros y un fallo a medias deja datos inconsistentes, necesita transacción.

**Zona horaria** — guarda en UTC, convierte al presentar. Mezclar husos en la base de datos es irreparable después.

## 6. Revisión de un endpoint existente

Recorre esta lista:

- [ ] ¿Valida toda la entrada?
- [ ] ¿Comprueba permiso sobre el objeto concreto, no solo autenticación?
- [ ] ¿Filtra los campos de salida con lista blanca?
- [ ] ¿Devuelve códigos de estado correctos?
- [ ] ¿Oculta detalles internos en los errores?
- [ ] ¿Está paginado si devuelve una lista?
- [ ] ¿Es transaccional si toca varios registros?
- [ ] ¿Tiene prueba del caso de fallo, no solo del feliz?

## Relación con otras skills

- Modelado de datos y consultas: `mongodb`
- Implementación concreta en Python: `python-fastapi`
- Antes de codificar: `test-driven-development`, `design-testing-strategy`
- Al terminar: `review-local-changes`, `verification-before-completion`
- Si algo falla en ejecución: `systematic-debugging`, `root-cause-tracing`
