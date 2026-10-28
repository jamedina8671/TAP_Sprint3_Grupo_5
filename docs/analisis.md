# Análisis — Sprint 3

## ¿Qué mejoraron?

- **Más y mejores aserciones:** de 35 a 90 por ejecución. Antes cada caso validaba solo el código HTTP y el `codigo` de error; ahora también se verifica contenido (`expiresAt`, nombre/apellido, número de pedido) y que la respuesta **no expone la contraseña** del cadete (casos 04 y 07).
- **Chequeos que antes no existían y ahora se aplican a todos los requests:** tiempo de respuesta menor a `maxMs` (1500 ms por defecto), `Content-Type` JSON y, en las respuestas de error, el formato estándar `status / error / codigo / message`. Son 19 verificaciones de tiempo y 10 de formato definidas una sola vez.
- **Ejecución sin pasos manuales:** `npm test` o un push al repositorio dispara todo y genera los reportes HTML/JSON, más un resumen en consola. Antes había que abrir Postman, importar, correr y armar el reporte a mano.
- **Suite repetible:** el paso Z1 restaura el estado del cadete a `LIBRE`, de modo que el sistema queda igual antes y después de correr y el pipeline no depende del estado que dejó una ejecución anterior.
- **Parametrización y portabilidad:** ids y credenciales fuera de los casos; entornos separados (`local` y `ci`) para que la misma colección corra en la máquina de cada integrante y en GitHub Actions.
- **Integración continua:** el workflow corre en cada push a `main`, en cada pull request y a pedido; un fallo deja el pipeline en rojo y los reportes quedan como *artifacts* de cada ejecución.

## ¿Qué costó más?

- **Que la suite dependa de la carga de demostración y del orden de ejecución:** P1–P3 deben correr antes de los casos para guardar tokens e ids, y el caso 06 asume que la API admite una sola sesión por cadete (el token de P2 queda inválido al ejecutar el caso 01). Eso hace la suite frágil frente a cambios en los datos o en el orden.
- **Levantar el backend con MySQL dentro del pipeline** (job `api-real`): compilar con Maven, arrancar el servicio MySQL y esperar a que el backend responda. Es la parte más pesada de configurar y la más sensible a detalles del repositorio del backend (nombre del jar, propiedades de conexión).
- **La prioridad de variables en Postman:** una variable de entorno vacía pisa el valor que guarda un script. Costó tiempo de depuración entender por qué un token aparecía vacío; la solución fue guardar tokens e ids como variables de colección y no definirlos en el entorno.

## ¿Qué aprendieron de la automatización?

- **Probar a mano no es lo mismo que mantener infraestructura de pruebas.** La colección dejó de ser algo que se "corre una vez" y pasó a ser código: se versiona, se revisa y hay que mantenerla cuando cambia la API.
- **Qué automatizar y qué no.** Los chequeos transversales (tiempo, formato, seguridad) rinden más como reglas globales que repetidas en cada caso; y el orden de ejecución y los datos de prueba pasan a ser parte del diseño, no un detalle.
- **Un pipeline en rojo evita que un error llegue a producción.** Cada push y cada PR vuelve a ejecutar los 15 casos y bloquea la integración ante cualquier regresión, sin depender de que alguien recuerde correr las pruebas.
- **El valor de un mock para desacoplar la validación.** Poder validar la suite y el pipeline sin el backend real (`suite-contra-mock`) acelera el desarrollo y deja el job `api-real` para la verificación final del sistema.
