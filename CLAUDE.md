# CORTANA — pautas de desarrollo

## Qué es
Captura por voz en campo para SheepMaster. Nota de voz -> transcripción -> borradores
estructurados -> confirmación humana -> escritura en SheepMaster por su API REST.
Tipos de registro del MVP: pesaje, tratamiento, parto, movimiento de corral y tarea.
Detalle funcional en `README.md`. La especificación completa vive en la bóveda de
Obsidian del responsable técnico (`01 Proyectos/CORTANA/`).

## Estado
Sin código todavía. Fase 0 (validación en campo) pendiente. No crear estructura ni
dependencias hasta que las historias del MVP estén aprobadas.

## Reglas que no se rompen
1. No se desarrolla nada sin historia de usuario aprobada (HU-x.y).
2. Ningún borrador se aplica en SheepMaster sin confirmación explícita de un usuario.
3. CORTANA no persiste el inventario del hato. Solo lee catálogos con caché corta.
4. Toda consulta filtra por `ranchoId` de la sesión. Nunca se acepta `ranchoId` del cliente.
5. El animal se identifica por coincidencia exacta de arete en código. El modelo de IA no
   elige al animal.
6. La transcripción es dato no confiable: no se usa como instrucción ni se concatena en
   el mensaje de sistema del modelo. Cada registro extraído debe citar una frase literal
   de la transcripción.
7. Cada cambio de estado de Captura o Borrador escribe un EventoAuditoria en la misma
   transacción.
8. Transcriptor, extractor, almacenamiento y SheepMaster se consumen por interfaz
   (puerto). El dominio y los casos de uso no importan SDKs.
9. Nunca versionar audios, el conjunto dorado, `.env` ni datos de ranchos.

## Arquitectura prevista
- Monorepo: `backend/` (NestJS, Prisma, PostgreSQL) y `web/` (Next.js PWA).
- `backend/src/modules/<modulo>/{domain,application,infrastructure}`
- `web/src/app` (rutas), `web/src/features/<feature>`, `web/src/lib/api` (cliente generado),
  `web/src/lib/cola` (cola local en IndexedDB).
- Contrato: OpenAPI generado por el backend. El frontend no escribe tipos de API a mano.

## Estilo
- TypeScript estricto. Sin `any` sin justificación en comentario.
- Errores de dominio con clases propias; respuesta HTTP en formato Problem Details.
- Logs JSON con `correlationId`. Nunca registrar transcripciones, contraseñas ni tokens.
- Pruebas junto al código: unitarias para dominio y casos de uso, integración para
  repositorios y el gateway de SheepMaster. El nombre de cada prueba lleva su HU.

## Definición de terminado
Criterios de aceptación con prueba o evidencia; lint, tipos, pruebas y build en verde;
diff revisado por una persona; OpenAPI y cliente actualizados; nota en la bitácora.

## Comandos
(Completar al crear el código: instalar, levantar base, migrar, dev, test, lint, build.)
