# Plugins

Una skill es un archivo de instrucciones. Un plugin es un paquete que se instala y puede traer skills,
agentes, comandos y conexiones (MCP). Los plugins no se copian a este repositorio: se reinstalan con
estos comandos.

## Instalados en Claude Code (esta PC)

Desde la tienda oficial de Anthropic:

```
claude plugin marketplace add anthropics/claude-plugins-official
claude plugin install frontend-design@claude-plugins-official
claude plugin install feature-dev@claude-plugins-official
claude plugin install claude-security@claude-plugins-official
```

| Plugin | Para qué |
|---|---|
| `frontend-design` | Dirección visual de interfaces: tipografía, color y composición con criterio propio |
| `feature-dev` | Explorar el código, diseñar la arquitectura y revisar una función nueva |
| `claude-security` | Escanear el código o un cambio en busca de vulnerabilidades y proponer parches |

## Desde la cuenta de Claude (app de escritorio / claude.ai)

Estos llegan con la cuenta, no se instalan en la PC:

| Plugin | Para qué | Nota |
|---|---|---|
| `design` | Crítica de diseño, accesibilidad (WCAG), design system, textos de interfaz, handoff | |
| `figma` | Leer y generar diseños en Figma | Requiere conectar Figma en los conectores de claude.ai |
| `anthropic-skills` | `ui-ux-pro-max`, `modern-web-design`, `frontend-design`, documentos (docx, pdf, xlsx, pptx) | |
| `marketing` | Campañas, SEO, contenido | Algunos conectores requieren autorización |

## Cómo usarlos en un proyecto

- Antes de diseñar una pantalla: `frontend-design` o `ui-ux-pro-max`.
- Al terminar: `design:design-critique` y `design:accessibility-review`.
- Antes de subir: `security-review` o `claude-security`.
