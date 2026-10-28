# Documento de optimización — Sprint 3

## 1. Punto de partida (colección del Sprint 2)

- 18 requests (15 casos + P1, P2, P3), 18 scripts de test.
- 35 aserciones en total; cada caso de error repetía a mano las mismas verificaciones.
- Credenciales y URL dentro de la colección; ids inexistentes escritos a mano en los casos 08 y 14.
- El caso 10 cambiaba el estado del cadete a `DESCONECTADO` y no lo restauraba: una segunda ejecución partía de un estado distinto.
- Sin límite de tiempo de respuesta ni verificación del formato de error en todos los requests.
- Ejecución manual desde Postman.

## 2. Cambios realizados

| # | Cambio | Justificación técnica |
|---|---|---|
| 1 | Chequeos a nivel de colección: tiempo de respuesta, `Content-Type` JSON y formato estándar de error (`status` coincide con el HTTP, `error`, `codigo`, `message`) | Se escriben una vez y se aplican a los 19 requests; antes no existían. Detecta regresiones de contrato sin tocar cada caso |
| 2 | Aserciones nuevas en casos existentes: `expiresAt` (01), nombre y apellido (04), mismo cadete (10), número de pedido (13) | Verifican el contenido de la respuesta, no solo el código HTTP |
| 3 | Chequeo de que la respuesta no expone la contraseña (04 y 07) | Control de seguridad barato sobre datos del cadete |
| 4 | Paso Z1 de limpieza: restaura el estado del cadete a `LIBRE` | La suite queda repetible; el pipeline no depende del estado que dejó la ejecución anterior |
| 5 | Variables `idInexistente` y `maxMs` | Sin valores fijos en los casos; el umbral de tiempo se ajusta por entorno |
| 6 | Credenciales y URL en archivos de entorno (`local` y `ci`) | La misma colección corre en la máquina de cada integrante y en el pipeline |
| 7 | Newman + reporters HTML y JSON, scripts `npm` y workflow de GitHub Actions | Ejecución y reportes sin intervención manual |
| 8 | Resumen automático (`scripts/resumen.js`) | Resultados visibles en la pestaña del workflow sin abrir el reporte |

## 3. Medición antes / después

Mediciones estáticas (conteo sobre la colección) y ejecución de validación contra el mock del contrato:

| Métrica | Antes | Después |
|---|---|---|
| Requests | 18 | 19 (se suma Z1) |
| Casos de prueba | 15 | 15 |
| Aserciones por ejecución | 35 | 90 |
| Chequeos de tiempo de respuesta | 0 | 19 |
| Verificaciones de formato de error | 0 (cada caso verificaba `codigo` a mano) | 10 (una por cada respuesta de error, definidas una sola vez) |
| Valores fijos en los casos | 2 ids + credenciales | 0 |
| Estado del sistema tras ejecutar | modificado (estado del cadete) | restaurado |
| Pasos manuales para ejecutar y reportar | abrir Postman, importar, correr, armar el reporte a mano | 0 (`npm test` o push al repositorio) |

**Ejecución medida** (1 corrida con Newman, 2026-10-27):

| Métrica | Antes (Postman manual) | Después (Newman) |
|---|---|---|
| Tiempo para ejecutar y obtener el reporte | ~10–15 min (importar, correr los 18 requests y armar el reporte a mano) | ~1,7 s de ejecución; los reportes HTML y JSON se generan solos |
| Tiempo de respuesta promedio de la API | no se registraba | 3 ms (mín 1 ms, máx 30 ms) |
| Casos aprobados / fallidos | 15/15 aprobados (revisión manual, sin reporte) | 15/15 aprobados · 90/90 aserciones · 0 fallos |

> Los valores de la columna **después** son reales y medidos contra el **mock del contrato** (19 requests, 90 aserciones, 0 fallos, 1685 ms en total). El mock valida que la suite y el pipeline funcionan, pero **no describe el rendimiento de la API real**: esos tiempos deben re-medirse contra el backend (job `api-real`) y promediar 3 corridas. Los valores de la columna **antes** son estimaciones del trabajo manual del Sprint 2 y conviene confirmarlos entre todos.

## 4. Evidencia

- Reporte HTML (`reportes/reporte.html`) y JSON (`reportes/reporte.json`) generados por Newman en la ejecución automática. En GitHub Actions quedan como *artifacts*: pestaña Actions → ejecución → Artifacts (`reportes-mock` y `reportes-api-real`).
- Resumen de consola de la ejecución validada (2026-10-27, contra el mock):

```text
## Resultado de la suite de pruebas (Newman)

| Métrica | Valor |
|---|---|
| Requests ejecutados | 19 |
| Aserciones | 90 OK / 0 fallidas |
| Duración total | 1685 ms |
| Tiempo de respuesta promedio | 3 ms |

Sin fallos.
```

- Capturas pendientes de adjuntar: [captura del workflow en verde, del resumen en la pestaña Actions y del reporte HTML].
