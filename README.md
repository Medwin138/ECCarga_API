# 🐶 EC-Carga API <sub>· pruebas de API sin drama</sub>

[![CI · Postman Tests](https://github.com/Medwin138/ECCarga_API/actions/workflows/ci.yml/badge.svg)](https://github.com/Medwin138/ECCarga_API/actions/workflows/ci.yml)
[![CI · API + Seguridad](https://github.com/Medwin138/ECCarga_API/actions/workflows/Api%2BSeguridadl.yml/badge.svg)](https://github.com/Medwin138/ECCarga_API/actions/workflows/Api%2BSeguridadl.yml)
![License](https://img.shields.io/badge/license-MIT-3DA639?style=flat-square)
![Node](https://img.shields.io/badge/node-18.x-339933?style=flat-square&logo=node.js&logoColor=white)
![Postman](https://img.shields.io/badge/postman-ready-FF6C37?style=flat-square&logo=postman&logoColor=white)
![OWASP ZAP](https://img.shields.io/badge/OWASP-ZAP-00549E?style=flat-square&logo=owasp&logoColor=white)

> 🧪 Suite de pruebas de API con **Postman + Newman**, reportes HTML bonitos y seguridad con **OWASP ZAP**,
> todo corriendo solito en **GitHub Actions**. Sin servidores, sin credenciales, sin excusas. 🚀

Se ejecuta contra la API pública de ejemplo [Swagger Petstore](https://petstore.swagger.io) 🐕, así que puedes
clonar el repo y verlo funcionar en menos de un minuto. Sí, en serio. ⏱️

---

## 📦 ¿Qué hay adentro?

| Archivo | ¿Qué es? |
| --- | --- |
| 🧾 `newman/EC-Carga.postman_collection.json` | La colección Postman con **4 peticiones** y sus aserciones |
| 🌱 `newman/Prueba.postman_environment.json` | Environment `Prueba` con las variables `Nombre`, `id` y `randomDogName` |
| 🎲 `newman/DataPruebaDog.csv` | Datos para pruebas data-driven: **30 perritos** (nombre + id) |
| 📊 `newman/EC-Carga-*.html` | Reporte HTML de ejemplo generado por `newman-reporter-htmlextra` |
| ⌨️ `Comandos.txt` | Los comandos de siempre, por si no quieres pensarlos |
| 🤖 `.github/workflows/ci.yml` | Pipeline de pruebas con Newman |
| 🛡️ `.github/workflows/Api+Seguridadl.yml` | Pruebas + escaneo ZAP + publicación en GitHub Pages |

---

## 🎯 Las peticiones (y qué validan)

| Petición | Método | Endpoint | Checks |
| :--- | :---: | :--- | :---: |
| ➕ `Crear` | `POST` | `/v2/pet` | **6** |
| 🔍 `Consulta Por ID` | `GET` | `/v2/pet/{{id}}` | **7** |
| ✏️ `Actualizar` | `PUT` | `/v2/pet` | **7** |
| 🔁 `Consulta Por ID Copy` | `GET` | `/v2/pet/{{id}}` | **8** |

Cada request valida:

- ✅ Código de estado HTTP correcto
- 🆔 `id` presente y no nulo
- 🏷️ `category.name` con contenido
- 🟢 `status` igual a `available`
- 🔤 Nombre con más de 2 caracteres
- 🧱 Estructura del JSON devuelto

Y el request `Crear` tiene un *pre-request script* que sortea un nombre de perro 🎲 y lo guarda en
`randomDogName`. Nunca dos corridas iguales. Bueno, casi nunca. 😄

---

## ⚡ Cómo correrlo en tu máquina

**Requisito único:** [Node.js](https://nodejs.org) 18+ 🟩

```bash
# 1️⃣ Instala Newman y el reporter HTML
npm install -g newman newman-reporter-htmlextra

# 2️⃣ Lanza la suite con environment y datos
newman run newman/EC-Carga.postman_collection.json \
  -e newman/Prueba.postman_environment.json \
  -d newman/DataPruebaDog.csv \
  -r cli,htmlextra \
  --reporter-htmlextra-export newman/report.html
```

📂 Al terminar, abre `newman/report.html` y presume tu reporte. 🕶️

> 💡 También puedes importar la colección en Postman y darle *Run* como todo el mundo.

---

## 🤖 Integración continua (la parte mágica)

### 1️⃣ `ci.yml` — *"Run Postman Tests"*

Se dispara con cada **push a `main`**, **cada día a las 13:00 UTC** ⏰ o **a mano** desde la pestaña Actions.

1. Instala Node 18 + Newman + reporter
2. Corre la colección con environment y CSV
3. Sube la carpeta `newman` como artefacto `postman-report` 📤

### 2️⃣ `Api+Seguridadl.yml` — *"CI Pipeline · API + Seguridad + Reports Web"*

Tres jobs en cadena:

| Job | Qué hace |
| --- | --- |
| 🧪 `run-tests` | Pruebas de API y preparación de `gh-pages` |
| 🛡️ `zap-scan` | OWASP ZAP full scan contra `petstore.swagger.io/v2/pet/findByStatus?status=available`; falla solo con alertas **High** 🔴 |
| 🌐 `publish-pages` | Publica los reportes en GitHub Pages |

> ⚠️ **Ojo:** el paso de Newman usa `|| true`, así que un check en rojo **no** tumba el pipeline.
> ¿Quieres que las pruebas bloqueen el merge? Borra ese `|| true`. Así de fácil. ✂️

---

## 🛡️ Seguridad sin sustos

- 🔍 Escaneo ZAP automático en cada corrida del pipeline
- 📄 Reportes HTML + JSON como artefactos descargables
- 🚨 El job falla únicamente con vulnerabilidades de riesgo **High**

---

## 🎯 ¿Y si quiero probar MI API?

La URL está fija en cada request. Para apuntar a la tuya:

1. 🔗 Cambia `https://petstore.swagger.io/v2/pet` por tu endpoint
2. 🧩 Ajusta el body y las aserciones al contrato real
3. 🛡️ Actualiza la URL del paso de ZAP en `Api+Seguridadl.yml`

---

## 🔒 Privacidad

Todo es dato ficticio de una API de ejemplo. **Cero credenciales, cero datos reales, cero problemas.** 🙈

---

## 📄 Licencia

![MIT](https://img.shields.io/badge/license-MIT-3DA639?style=for-the-badge)

Distribuido bajo **MIT** — usa, copia, modifica y comparte. Solo conserva el aviso de copyright. 🤝
Ver [LICENSE](LICENSE).

---

<p align="center">
  <b>Hecho con ☕ + 🐾 por <a href="https://github.com/Medwin138">Medwin138</a></b><br/>
  <i>Si te sirvió, déjale una ⭐ — alimenta al perrito.</i> 🐕‍🦺
</p>
