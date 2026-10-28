# Bitácora — Sprint 3

Proyecto de Especialización en Herramientas de Testing — QualityTech Solutions.
Herramienta: **Postman + Newman**. Objetivo del sprint: automatizar los 15 casos del Sprint 2, generar reportes y correr todo en integración continua.

| Fecha | Integrante | Tarea | Decisión / problema | Resolución |
|---|---|---|---|---|
| 2026-10-27 | Basauri Augusto | Consolidar la colección y agregar chequeos globales | Los 15 casos de error repetían a mano las mismas verificaciones (código HTTP + `codigo`) y no había control de tiempo, `Content-Type` ni formato completo del error | Chequeos a nivel de colección (tiempo de respuesta < `maxMs`, `Content-Type` JSON y formato estándar `status/error/codigo/message` en respuestas ≥ 400) + aserciones de contenido nuevas (`expiresAt`, nombre/apellido, no exponer contraseña, número de pedido) |
| 2026-10-27 | Garnica German | Workflow de GitHub Actions | Ejecutar la suite sin depender del backend en cada push, y contra la API real solo cuando esté disponible | `tests.yml` con dos jobs: `suite-contra-mock` (siempre, contra `mock/server.js`) y `api-real` (se activa con la variable `BACKEND_REPO`, levanta MySQL + backend Spring Boot). Reportes como *artifacts* y resumen en la pestaña del workflow |
| 2026-10-27 | Medina Jorge | Mediciones antes/después y documento de optimización | Mostrar el impacto con números (aserciones, pasos manuales, repetibilidad) en lugar de solo describir los cambios | `docs/optimizacion.md` con tabla antes/después (35 → 90 aserciones, 0 → 19 chequeos de tiempo, 0 pasos manuales, estado del sistema restaurado) |
| 2026-10-27 | Vallejo Leandro | README, reportes y prueba desde cero | Que cualquier integrante (o el evaluador) pueda clonar y correr todo sin contexto previo | README con ejecución desde cero (`npm install`, `npm test`, `npm run test:ci`), `scripts/resumen.js` para el resumen en consola y en el workflow, y validación de instalación limpia contra el mock |

## Decisiones técnicas

- Credenciales y URL en archivos de entorno (`local` y `ci`) y no en la colección: `pm.variables.get` resuelve entorno + colección; `pm.collectionVariables.get` solo la colección. Así la misma colección corre en la máquina de cada integrante y en el pipeline.
- Los tokens e ids se guardan como variables de colección. No definirlos en el entorno: el valor del entorno tiene prioridad y pisaría lo que guardan los scripts (esto provocó tokens vacíos hasta entender la regla).
- Chequeos transversales (tiempo, formato, seguridad) como reglas globales de colección en lugar de repetirlos en cada caso: se escriben una vez y se aplican a los 19 requests.
- Ejecución contra mock en el pipeline para validar la suite sin depender del backend; la API real corre en el job `api-real`. El mock reproduce el contrato, no reemplaza las pruebas contra la API real.
- Umbral de tiempo (`maxMs`) e ids inexistentes (`idInexistente`) parametrizados en variables, sin valores fijos dentro de los casos.
- Paso Z1 de limpieza para que la suite sea repetible: el caso 10 deja al cadete en `DESCONECTADO` y Z1 lo restaura a `LIBRE`.

## Problemas resueltos

| Problema | Causa | Solución |
|---|---|---|
| Una segunda ejecución fallaba o partía de un estado distinto | El caso 10 cambiaba el estado del cadete a `DESCONECTADO` y no lo restauraba | Paso Z1 que restaura el estado a `LIBRE` al final; el sistema queda igual antes y después de correr |
| Un token guardado por un script aparecía vacío o pisado | Prioridad de variables en Postman: el valor del entorno pisa al de la colección | Tokens e ids solo como variables de colección, sin definirlos en el entorno |
| Los casos 08 y 14 tenían ids inexistentes escritos a mano | Ids fijos dentro del request | Variable `idInexistente` en la colección, reutilizada en ambos casos |
| Ejecutar y reportar a mano era lento y propenso a error | Postman manual + armado del reporte a mano | Newman con reporters `cli`, `htmlextra` y `json`; `npm test`/`npm run test:ci`; `scripts/resumen.js` |
| No había forma de validar la suite sin el backend disponible | Dependencia del backend real para correr | Mock del contrato (`mock/server.js`) que corre en el job `suite-contra-mock` |
