---
name: decision-eval
description: Evalúa una decisión técnica o de producto comparando opciones contra criterios ponderados explícitos, y produce una recomendación con su justificación y condiciones de reversión. Úsala cuando haya que elegir entre alternativas (librería, arquitectura, base de datos, proveedor, enfoque de implementación, orden de trabajo), cuando el usuario pregunte "¿cuál conviene?", "¿qué opción es mejor?", "¿debería usar X o Y?", o cuando una decisión sea costosa de revertir.
---

# Evaluación de decisiones

Convierte una elección difusa en una decisión defendible y registrada.

## Cuándo usarla

Úsala cuando **al menos una** sea cierta:
- Hay 2 o más opciones reales sobre la mesa
- La decisión es cara de revertir (esquema de datos, framework, proveedor, contrato de API)
- El equipo no está de acuerdo, o el usuario duda
- La decisión condiciona trabajo posterior

**No la uses** para elecciones triviales o fácilmente reversibles. Si revertir cuesta menos que evaluar, decide y sigue.

## Procedimiento

### 1. Enuncia la decisión en una frase

Formato: *"Elegir X para lograr Y, dado Z."*

Si no puedes escribirla así, el problema aún no está definido — para y clarifícalo primero.

### 2. Nombra las opciones reales

Mínimo 2, máximo 5. Incluye siempre:
- **No hacer nada** / mantener lo actual — es una opción legítima y suele ganar
- La opción "aburrida" — la tecnología conocida y madura

Descarta explícitamente las que no consideras, con una línea de por qué. Una opción omitida sin justificar es un punto ciego.

### 3. Define los criterios ANTES de puntuar

Esto es lo que separa una evaluación de una racionalización. Si eliges los criterios después de mirar las opciones, estás justificando una decisión ya tomada.

Criterios habituales — elige 3 a 6, no más:

| Criterio | Pregunta que responde |
|---|---|
| Costo de reversión | ¿Qué cuesta deshacer esto en 6 meses? |
| Tiempo hasta funcionar | ¿Cuándo tenemos algo en producción? |
| Carga de mantenimiento | ¿Quién lo cuida y cuánto duele? |
| Ajuste al equipo | ¿Sabemos hacerlo o hay que aprender? |
| Costo operativo | ¿Cuánto cuesta al mes en escala real? |
| Riesgo de proveedor | ¿Qué pasa si desaparece o sube precios? |
| Madurez | ¿Está probado o somos los conejillos? |

Asigna un peso a cada criterio (1-5). **Los pesos se justifican con el contexto del proyecto**, no con preferencias. Escribe la justificación de cada peso en una línea.

### 4. Puntúa

Matriz: opciones en filas, criterios en columnas, puntaje 1-5.

```
                   Reversión  Tiempo  Mantto.  Equipo   TOTAL
                   (peso 5)   (p 3)   (p 4)    (p 4)
Opción A              4        2        3        5       63
Opción B              2        5        4        2       49
No hacer nada         5        5        2        5       68
```

Cada celda necesita **una frase de evidencia**, no solo un número. Un puntaje sin evidencia es una opinión disfrazada de dato.

### 5. Aplica la prueba de inversión

Antes de aceptar el resultado, pregunta: *"Si el ganador hubiera sacado 5 puntos menos, ¿cambiaría mi recomendación?"*

- **No cambia** → el resultado es robusto
- **Sí cambia** → las opciones están empatadas. Dilo así, y decide por el criterio que más importa, no por el total.

Nunca presentes un empate técnico como si fuera un ganador claro.

### 6. Entrega la recomendación

Formato de salida:

```markdown
## Decisión
[Una frase]

## Recomendación
**[Opción]** — [dos o tres frases de por qué]

## Por qué no las otras
- **[Opción B]**: [razón concreta, no genérica]
- **[Opción C]**: [razón concreta]

## Qué haría cambiar esta decisión
- Si [condición observable], reconsiderar hacia [opción]
- Si [condición observable], reconsiderar hacia [opción]

## Costo de revertir
[Qué habría que deshacer, cuánto trabajo, y en qué punto se vuelve irreversible]
```

La sección **"qué haría cambiar esta decisión"** no es opcional. Una decisión sin condiciones de reversión es un dogma.

## Errores a evitar

- **Criterios elegidos a posteriori** — invalida toda la evaluación
- **Solo dos opciones cuando hay más** — el falso dilema es el sesgo más común
- **Omitir "no hacer nada"** — a menudo es la respuesta correcta
- **Puntajes sin evidencia** — números inventados con apariencia de rigor
- **Presentar empates como ganadores** — deshonesto y frágil
- **Ignorar el costo de reversión** — es el criterio que más se subestima

## Relación con otras skills

- Antes de decidir, si el problema no está claro: `analyse-problem`
- Para explorar opciones antes de evaluarlas: `brainstorming`, `tree-of-thoughts`
- Para someter la recomendación a debate adversarial: `judge-with-debate`
- Para buscar riesgos en la opción ganadora: `cause-and-effect`
- Tras decidir, registrar el plan: `writing-plans`
