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
