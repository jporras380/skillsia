# Skills globales

60 skills instaladas en `~/.claude/skills/`. Disponibles en **todos los proyectos**, presentes y futuros.

Instaladas el 2026-08-04.

---

## Animación y 3D

| Skill | Fuente | Instalaciones |
|---|---|---|
| `review-animations` | emilkowalski | 78.5K |
| `animation-vocabulary` | emilkowalski | 66.6K |
| `improve-animations` | emilkowalski | 51.2K |
| `find-animation-opportunities` | emilkowalski | 40.2K |
| `gsap-core` | **greensock (oficial)** | 42.4K |
| `gsap-scrolltrigger` | **greensock (oficial)** | 40.6K |
| `gsap-performance` | **greensock (oficial)** | 39.2K |
| `gsap-timeline` | **greensock (oficial)** | 39K |
| `threejs-fundamentals` | cloudai-x | 8.5K |
| `threejs-animation` | cloudai-x | 11K |
| `threejs-interaction` | cloudai-x | 6.8K |
| `motion-design` | **lottiefiles (oficial)** | 6.7K |
| `modern-web-guidance` | **googlechrome (oficial)** | 14.3K |

---

## Decisiones

| Skill | Para qué |
|---|---|
| `decision-eval` | Comparar opciones con criterios ponderados y recomendar |
| `judge-with-debate` | Debate adversarial entre posturas antes de decidir |
| `tree-of-thoughts` | Explorar ramas de razonamiento en paralelo |
| `brainstorming` | Generar opciones antes de evaluarlas |
| `analyse-problem` | Definir bien el problema antes de buscar solución |
| `plan-do-check-act` | Ciclo de mejora iterativa |

## Depuración

| Skill | Para qué |
|---|---|
| `systematic-debugging` | Método ordenado para encontrar fallos |
| `root-cause-tracing` | Rastrear del síntoma al disparador original |
| `five-whys` | Cinco porqués hasta la causa raíz |
| `cause-and-effect` | Diagrama causa-efecto (Ishikawa) |

## Pruebas

| Skill | Para qué |
|---|---|
| `test-driven-development` | Prueba primero, implementación después |
| `design-testing-strategy` | Qué probar y a qué nivel |
| `write-tests` | Escribir pruebas |
| `fix-tests` | Arreglar pruebas rotas |
| `test-coverage` | Analizar cobertura |
| `webapp-testing` | Probar apps web con Playwright |
| `verification-before-completion` | Verificar antes de dar algo por terminado |

## Revisión de código

| Skill | Para qué |
|---|---|
| `review-local-changes` | Revisar el diff de trabajo |
| `review-pr` | Revisar un pull request |
| `triage-review` | Priorizar hallazgos de revisión |
| `requesting-code-review` | Pedir revisión bien planteada |
| `receiving-code-review` | Procesar comentarios recibidos |

## Git

| Skill | Para qué |
|---|---|
| `git-commit` | Commits con buen mensaje |
| `git-create-pr` | Crear pull requests |
| `git-analyze-issue` | Analizar un issue antes de trabajarlo |
| `git-load-pr-comments` | Traer comentarios de un PR |
| `git-resolve-pr-comments` | Resolver comentarios ya corregidos |
| `using-git-worktrees` | Worktrees aislados |
| `finishing-a-development-branch` | Cerrar una rama correctamente |
| `changelog-generator` | Changelog desde el historial |

## Backend *(escritas a medida)*

| Skill | Para qué |
|---|---|
| `backend-api` | Contratos, validación, auth, errores, paginación |
| `mongodb` | Modelado, consultas, índices, agregaciones |
| `python-fastapi` | Estructura, routers, Pydantic, async, dependencias |
| `ci-cd-docker` | Dockerfile, compose, GitHub Actions, secretos |

## Planificación

| Skill | Para qué |
|---|---|
| `writing-plans` | Escribir planes de implementación |
| `executing-plans` | Ejecutar un plan paso a paso |

## Orquestación de subagentes

| Skill | Para qué |
|---|---|
| `multi-agent-patterns` | Patrones de coordinación multi-agente |
| `subagent-driven-development` | Desarrollo por subagentes con checkpoints |
| `launch-sub-agent` | Lanzar un subagente |
| `do-in-parallel` | Trabajo en paralelo |

## Documentación y contexto

| Skill | Para qué |
|---|---|
| `update-docs` | Mantener documentación al día |
| `write-concisely` | Escribir sin relleno |
| `context-engineering` | Gestión del contexto |
| `prompt-engineering` | Técnicas de prompting |

## Meta

| Skill | Para qué |
|---|---|
| `find-skills` | **Buscar e instalar skills del registro de Vercel** (skills.sh) |
| `skill-creator` | Crear skills nuevas (oficial de Anthropic) |
| `build-mcp` | Construir servidores MCP |

---

## Fuentes

| Origen | Cantidad |
|---|---|
| [obra/superpowers](https://github.com/obra/superpowers) | 11 |
| [NeoLabHQ/context-engineering-kit](https://github.com/NeoLabHQ/context-engineering-kit) | 27 |
| [ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills) | 3 |
| [vercel-labs/skills](https://github.com/vercel-labs/skills) | 1 |
| Escritas a medida | 5 |

## Añadir más

```
npx skills add <owner/repo>
```

O pide directamente: *"busca una skill para X"* — `find-skills` consulta el registro de Vercel.

## Notas

- Las skills de `superpowers` traían referencias al prefijo de plugin `superpowers:`; fueron limpiadas para funcionar como skills sueltas.
- Varias de `context-engineering-kit` se renombraron con prefijo `git-` para evitar nombres genéricos (`commit`, `create-pr`).
- Los plugins `ddd` y `tech-stack` de context-engineering-kit son *rules*, no skills — no se instalaron.
