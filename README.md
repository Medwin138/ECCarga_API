# EC-Carga API

[![CI](https://github.com/Medwin138/ECCarga_API/actions/workflows/ci.yml/badge.svg)](https://github.com/Medwin138/ECCarga_API/actions/workflows/ci.yml)
[![Licencia: MIT](https://img.shields.io/badge/licencia-MIT-blue.svg)](LICENSE)

Suite de pruebas automatizadas de API construida con Postman y ejecutada con Newman. Incluye reportes HTML e integración continua en GitHub Actions con pruebas funcionales y análisis de seguridad mediante OWASP ZAP.

Las pruebas se ejecutan contra la API pública de ejemplo [Swagger Petstore](https://petstore.swagger.io), por lo que no requieren credenciales ni infraestructura propia.

## Estructura del repositorio

| Ruta | Descripción |
| --- | --- |
| `newman/EC-Carga.postman_collection.json` | Colección Postman con cuatro peticiones y sus aserciones |
| `newman/Prueba.postman_environment.json` | Environment `Prueba` con las variables `Nombre`, `id` y `randomDogName` |
| `newman/DataPruebaDog.csv` | Conjunto de datos para pruebas data-driven (30 registros `id,Nombre`) |
| `newman/EC-Carga-*.html` | Reporte HTML de ejemplo generado por `newman-reporter-htmlextra` |
| `Comandos.txt` | Comandos de ejecución local |
| `.github/workflows/ci.yml` | Pipeline de pruebas con Newman |
| `.github/workflows/Api+Seguridadl.yml` | Pipeline de pruebas, escaneo ZAP y publicación en GitHub Pages |

## Alcance de las pruebas

| Petición | Método | Endpoint | Aserciones |
| --- | :---: | --- | :---: |
| `Crear` | `POST` | `/v2/pet` | 6 |
| `Consulta Por ID` | `GET` | `/v2/pet/{{id}}` | 7 |
| `Actualizar` | `PUT` | `/v2/pet` | 7 |
| `Consulta Por ID Copy` | `GET` | `/v2/pet/{{id}}` | 8 |

Las aserciones verifican el código de estado HTTP, la presencia y no nulidad de `id`, el contenido de `category.name`, el valor `status`, la longitud del nombre y la estructura general del JSON devuelto.

La petición `Crear` incluye un script de pre-request que selecciona un nombre de perro aleatorio y lo almacena en la variable de entorno `randomDogName`, de modo que cada ejecución envía datos distintos.

## Requisitos

- Node.js 18 o superior
- npm
- Opcional: Postman para editar la colección y Docker para ejecutar ZAP en local

## Ejecución local

Instalación de Newman y del reporter HTML:

```bash
npm install -g newman newman-reporter-htmlextra
```

Ejecución de la suite con el environment y los datos de prueba:

```bash
newman run newman/EC-Carga.postman_collection.json \
  -e newman/Prueba.postman_environment.json \
  -d newman/DataPruebaDog.csv \
  -r cli,htmlextra \
  --reporter-htmlextra-export newman/report.html
```

El reporte se genera en `newman/report.html`. Los mismos comandos están disponibles en `Comandos.txt`.

## Integración continua

### `ci.yml` — Run Postman Tests

Se ejecuta en cada push a `main`, de forma programada (todos los días a las 13:00 UTC) y mediante ejecución manual.

1. Instala Node.js 18, Newman y `newman-reporter-htmlextra`.
2. Ejecuta la colección con el environment y el archivo CSV.
3. Publica la carpeta `newman` como artefacto `postman-report`.

### `Api+Seguridadl.yml` — CI Pipeline

Se compone de tres jobs:

| Job | Función |
| --- | --- |
| `run-tests` | Ejecuta las pruebas de API y prepara la carpeta `gh-pages` |
| `zap-scan` | Ejecuta un full scan de OWASP ZAP contra `petstore.swagger.io/v2/pet/findByStatus?status=available` y falla solo si detecta alertas de riesgo alto |
| `publish-pages` | Publica los reportes en GitHub Pages |

> **Nota sobre fallos:** el paso de Newman finaliza con `|| true`, por lo que una aserción fallida no detiene el pipeline. El estado de las pruebas debe revisarse en el log o en el reporte HTML del artefacto. Para que un fallo bloquee la ejecución, elimine `|| true` del paso correspondiente.

## Notificaciones por correo

El repositorio no configura un servidor SMTP propio. Las notificaciones las envía GitHub a la dirección de correo asociada a la cuenta, siempre que las notificaciones por correo estén habilitadas.

GitHub notifica las ejecuciones de dos formas:

- **Ejecuciones que usted inicia:** recibe un correo con el estado final (`success`, `failed`, `neutral` o `cancelled`). Es posible configurar que solo se notifiquen los fallos.
- **Ejecuciones programadas:** el aviso se envía al usuario que creó el workflow. Si otro usuario modifica el `cron`, las notificaciones pasan a ese usuario.

Para habilitarlas:

1. Abra **Settings → Notifications** en su cuenta de GitHub.
2. En la sección **Actions**, active las notificaciones por correo y, si lo desea, limite el envío a ejecuciones fallidas.
3. Verifique que la dirección de correo esté confirmada en **Settings → Emails**.
4. Opcionalmente, para recibir también la actividad de CI del repositorio, use **Watch → Custom** y marque **Actions**.

> **Limitación:** debido al uso de `|| true`, un fallo en las aserciones de Newman no marca el workflow como fallido y, por tanto, no genera notificación de error. Para que el correo refleje el resultado real de las pruebas, elimine `|| true` del paso `Run Postman Collection` en `ci.yml` y del paso equivalente en `Api+Seguridadl.yml`.

## Seguridad

- Escaneo automatizado con OWASP ZAP en cada ejecución del pipeline de seguridad.
- Reportes HTML y JSON publicados como artefactos descargables.
- El job de ZAP falla únicamente ante vulnerabilidades de riesgo alto.

## Uso con otra API

Las URL están definidas en cada petición de la colección. Para apuntar a otra API:

1. Reemplace `https://petstore.swagger.io/v2/pet` por el endpoint correspondiente en la colección.
2. Ajuste el cuerpo de las peticiones y las aserciones al contrato real de la API.
3. Actualice la URL del paso de ZAP en `.github/workflows/Api+Seguridadl.yml`.

## Privacidad

Los endpoints, nombres y respuestas utilizados corresponden a datos ficticios de la API de ejemplo. El repositorio no contiene credenciales ni información de usuarios reales.

## Licencia

Este proyecto se distribuye bajo la licencia MIT. Consulte [LICENSE](LICENSE).

Copyright (c) 2026 Medwin138
