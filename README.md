# EC-Carga API

Suite de pruebas automatizadas de API construida con **Postman** y ejecutada con **Newman**, con reportes HTML y un pipeline de integración continua en **GitHub Actions** que además ejecuta un escaneo de seguridad con **OWASP ZAP**.

Las pruebas se ejecutan contra la API pública de ejemplo [Swagger Petstore](https://petstore.swagger.io), por lo que no requieren credenciales ni infraestructura propia.

## Contenido

| Elemento | Descripción |
| --- | --- |
| `newman/EC-Carga.postman_collection.json` | Colección Postman con 4 peticiones y sus aserciones |
| `newman/Prueba.postman_environment.json` | Environment `Prueba` (variables `Nombre`, `id`, `randomDogName`) |
| `newman/DataPruebaDog.csv` | Datos para pruebas data-driven: 30 registros `id,Nombre` |
| `newman/EC-Carga-*.html` | Reporte HTML de ejemplo generado por `newman-reporter-htmlextra` |
| `Comandos.txt` | Comandos de ejecución local |
| `.github/workflows/ci.yml` | Pipeline de pruebas con Newman |
| `.github/workflows/Api+Seguridadl.yml` | Pipeline de pruebas + escaneo ZAP + publicación en GitHub Pages |

## Peticiones y validaciones

| Petición | Método | Endpoint | Checks |
| --- | --- | --- | --- |
| `Crear` | `POST` | `/v2/pet` | 6 |
| `Consulta Por ID` | `GET` | `/v2/pet/{{id}}` | 7 |
| `Actualizar` | `PUT` | `/v2/pet` | 7 |
| `Consulta Por ID Copy` | `GET` | `/v2/pet/{{id}}` | 8 |

Cada petición valida código de estado, presencia y no nulidad de `id`, contenido de `category.name`, `status`, longitud del nombre y estructura del JSON devuelto. La petición `Crear` incluye un script de pre-request que elige un nombre de perro aleatorio y lo guarda en la variable de entorno `randomDogName`.

## Requisitos

- [Node.js](https://nodejs.org) 18 o superior
- npm
- Opcional: Postman para editar la colección, Docker para ejecutar ZAP en local

## Ejecución local

Instala Newman y el reporter HTML:

```bash
npm install -g newman newman-reporter-htmlextra
```

Ejecuta la colección con el environment y los datos de prueba:

```bash
newman run newman/EC-Carga.postman_collection.json \
  -e newman/Prueba.postman_environment.json \
  -d newman/DataPruebaDog.csv \
  -r cli,htmlextra \
  --reporter-htmlextra-export newman/report.html
```

El reporte queda en `newman/report.html`. Los mismos comandos están en `Comandos.txt`.

## Integración continua

**`ci.yml`** se ejecuta en cada push a `main`, de forma programada (diariamente a las 13:00 UTC) o manualmente:

1. Instala Node.js 18, Newman y `newman-reporter-htmlextra`.
2. Ejecuta la colección con el environment y el CSV.
3. Publica la carpeta `newman` como artefacto `postman-report`.

**`Api+Seguridadl.yml`** añade dos jobs sobre el anterior:

1. `run-tests`: mismas pruebas de API y preparación de la carpeta `gh-pages`.
2. `zap-scan`: OWASP ZAP full scan contra `https://petstore.swagger.io/v2/pet/findByStatus?status=available`, con reportes HTML y JSON como artefactos; el job falla solo si ZAP reporta alertas de riesgo **High**.
3. `publish-pages`: publica los reportes en GitHub Pages.

> Nota: el paso de Newman usa `|| true`, de modo que un fallo en las aserciones no detiene el pipeline. Si quieres que las pruebas bloqueen el merge, elimina ese `|| true` del workflow.

## Ejecutar contra otra API

La URL está fijada en cada petición de la colección. Para apuntar a otra API:

1. Sustituye `https://petstore.swagger.io/v2/pet` por tu endpoint en Postman o en el JSON de la colección.
2. Ajusta el body y las aserciones al contrato real de tu API.
3. Actualiza la URL del paso de ZAP en `.github/workflows/Api+Seguridadl.yml`.

## Datos y privacidad

Los endpoints, nombres y respuestas son datos ficticios de la API pública de ejemplo. El repositorio no contiene credenciales ni información real de usuarios.

## Licencia

Distribuido bajo licencia **MIT**. Consulta [LICENSE](LICENSE).

Copyright (c) 2026 Medwin138
