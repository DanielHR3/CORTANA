# CORTANA — pautas de desarrollo

## Qué es
Asistente personal de voz de un solo usuario (Daniel). Conversa por voz y texto, recuerda
entre conversaciones y actúa sobre Obsidian, Google Calendar, Gmail, GitHub y SheepMaster.
Toda acción que cambia algo fuera de CORTANA requiere confirmación de Daniel.
Detalle funcional en `README.md`. La especificación completa vive en la bóveda de Obsidian
del responsable técnico (`01 Proyectos/CORTANA/`).

## Estado
Sin código. Proyecto en pausa, solo documentación. Cuando se retome, el primer paso es
la Fase 0: prueba de concepto de la cascada de voz (desechable, fuera de este repositorio).
No crear estructura ni dependencias antes de cerrarla.

## Reglas que no se rompen
1. No se desarrolla nada sin historia de usuario aprobada (HU-x.y).
2. Cada herramienta se registra con clasificación `lectura` o `accion`. Sin clasificación,
   el registro falla al arrancar.
3. Una herramienta de `accion` nunca se ejecuta cuando el modelo la llama: crea una acción
   pendiente. Solo el módulo de acciones la ejecuta, tras confirmación desde la interfaz
   o desde el detector de afirmaciones por voz.
4. El modelo no tiene ninguna herramienta ni camino para confirmar acciones.
5. La confirmación por voz la decide código: una sola acción pendiente y una afirmación de
   la lista cerrada. Nunca se le pregunta al modelo si el usuario aceptó.
6. Lo ejecutado usa los argumentos guardados en la acción, sin volver a consultar al modelo.
7. Contenido de correos, notas, páginas y respuestas de APIs va delimitado como dato no
   confiable. Nunca se concatena en las instrucciones del sistema.
8. El historial enviado al modelo solo crece. Se guarda cada respuesta completa (incluidos
   bloques de pensamiento y herramientas). Información nueva entra como mensaje de sistema.
9. Credenciales solo en el backend, cifradas. Nunca en mensajes, memoria, logs ni el repo.
10. La memoria solo guarda lo que Daniel pide; nunca secretos.
11. El servidor MCP de Obsidian solo accede a la bóveda personal. Nada de información
    de trabajo institucional.
12. El repositorio es público: nada de `.env`, audios, conversaciones ni datos personales.

## Arquitectura prevista
- `backend/` NestJS + Prisma + PostgreSQL. Módulos en
  `src/modules/<modulo>/{domain,application,infrastructure}`.
- `backend/prompts/` instrucciones del sistema versionadas.
- `backend/eval/` conjunto de evaluación del agente (datos de prueba, nunca reales).
- `mcp/obsidian`, `mcp/sheepmaster`: servidores MCP propios; el backend es su cliente.
- `web/` Next.js PWA: conversación, tarjetas de confirmación, memoria, habilidades.
- Agente: SDK de TypeScript de Anthropic, Tool Runner con herramientas Zod, modelo
  `claude-opus-5-5`, pensamiento adaptativo, esfuerzo por tipo de petición, streaming.
- Voz: cascada voz a texto -> Claude -> texto a voz, todo en streaming.

## Estilo
- TypeScript estricto. Sin `any` sin justificación en comentario.
- Errores de dominio tipados; Problem Details en REST; eventos de error en WebSocket.
- Logs JSON con correlación. Nunca registrar texto de conversaciones, correos ni credenciales.
- Pruebas junto al código; el nombre de cada prueba lleva su HU.

## Definición de terminado
Criterios con prueba o evidencia; lint, tipos, pruebas y build en verde; evaluación del
agente pasando si se tocó al agente; diff revisado por Daniel; nota en la bitácora.

## Comandos
(Completar al crear el código.)

---

<!-- auditores:inicio -->
## 🛡️ Auditores del proyecto

Skills `auditor-*` en `~/.claude/skills/` (fuente: bóveda `01 Proyectos/Auditores/`; ciclo en `01 Proyectos/Marcos Metodologicos Freelance/Sistema de Auditoria y Calidad.md`). Los auditores **ponen a prueba, no corrigen**: exigen evidencia y emiten veredicto (Listo · Ajustes puntuales · Iteración profunda).

| Momento | Auditor | Qué entregarle |
|---|---|---|
| Antes de presentar propuesta, plan o reporte | `auditor-critico` | El documento |
| Antes de fijar logo, favicon o icono de la app | `auditor-logos` | Archivos, dónde aparecen y con qué marcas conviven |
| Al terminar una pantalla o flujo (escritorio + `390px` / dispositivo) | `auditor-ux` | URL o capturas, rol y tarea crítica; estados carga/vacío/error |
| Al exponer o consumir endpoints | `auditor-api` | URL base, contrato/OpenAPI y colección Postman con casos negativos (sin token, token ajeno, id inexistente, página fuera de rango) |
| Antes de producción (servidor y datos) | `auditor-seguridad` | Código, `.env.example`, permisos por ruta, migraciones, RLS/roles probados con usuario ajeno, respaldo **restaurado** (R5) |
| Al cambiar esquema o permisos de BD | `auditor-seguridad` + `auditor-api` | Migraciones versionadas, índices de paginación, `updated_at`/baja lógica, bitácora solo-inserción, seeds ficticios |

**Regla de trabajo para el agente**
- No declares nada «listo» sin evidencia (build, prueba, captura o petición ejecutada); lo no probado se reporta «no probado».
- Ante «audita …», entra en modo auditor: no reescribas código en ese paso; hallazgos con severidad, evidencia y pregunta crítica.
- Informe en la carpeta `Auditorias/` del proyecto en Obsidian: `AAAA-MM-DD - Auditoria {Tipo} - {Proyecto}.md`. Un hallazgo se cierra con evidencia (commit, prueba), no con «ya quedó».

**Obligatorio sí o sí antes de entregar** (bloqueante; se cierra con evidencia; lista completa en `01 Proyectos/Marcos Metodologicos Freelance/Obligatorios de Auditoria por Proyecto.md`):
- [ ] (Al retomar) Ninguna `accion` se ejecuta sin confirmación; el modelo no puede confirmar
- [ ] Correo/nota maliciosa no crea ni ejecuta acciones (caso en `eval/`)
- [ ] Repo público: escaneo de secretos del historial antes de cada push
- [ ] Credenciales cifradas, nunca en logs; MCP de Obsidian sin acceso a `05 Trabajo/`
<!-- auditores:fin -->
