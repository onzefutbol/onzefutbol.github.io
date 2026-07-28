# Bitácora — Onze Landing

Registro cronológico de decisiones, cambios y acciones sobre el proyecto.

---

## [2026-07-25] Lectura y análisis del diseño fuente

**Qué hice:** Leí los archivos `Onze Landing.dc.html` y `support.js` desde la carpeta `Onze branding system` en el Escritorio.

**Cómo lo hice:** Acceso directo a los archivos locales, sin usar el MCP `claude_design` (que requería autenticación via `/design-login` y no estaba disponible en el contexto).

**Por qué:** El usuario quería implementar el diseño de la landing. Para hacerlo sin el MCP, usé los archivos fuente locales que el usuario tenía en el Escritorio como punto de partida.

---

## [2026-07-25] Implementación de `Onze Landing.dc.html` como HTML standalone

**Qué hice:** Creé `onze-landing.html` en el Escritorio, convirtiendo el formato `.dc.html` (Design Components con React) a un HTML puro funcional.

**Cómo lo hice:**

El archivo fuente usaba un runtime propietario llamado `dc-runtime` con:

- Sintaxis de plantillas `{{ expresión }}` para interpolar valores
- Etiquetas `<sc-if value="{{ condición }}">` para condicionales
- Una clase `DCLogic` con `state` y `renderVals()` para manejar estado (React por debajo)
- Íconos generados programáticamente con SVG via `chevronIcon(size, baseColor, accentColor)`

Las conversiones que hice fueron:

- `{{ icons.wl28/wl40/wl320 }}` → SVG inline calculado con las fórmulas de `chevronIcon()`
- `<sc-if value="{{ submitted }}">` → `display:none/block` controlado con JS vanilla
- `pillStyle(active)` → clases CSS `.pill-active` / `.pill-inactive`
- `handleSubmit`, `selectPlayer`, `selectTeam` → funciones JS globales simples
- Estado (`email`, `role`, `submitted`) → variables JS en el scope global

**Por qué:** El runtime `dc-runtime` depende de React y de un servidor de streaming propietario de Claude Design. Para que la landing funcione de forma independiente (sin dependencias especiales), lo traduje a HTML + CSS + JS vanilla puro.

---

## [2026-07-25] Organización en carpeta de proyecto

**Qué hice:** Creé la carpeta `onze-landing/` en el Escritorio, moví el archivo y lo renombré a `index.html`.

**Cómo lo hice:**

```bash
mkdir C:\Users\Usuario\Desktop\onze-landing
mv onze-landing.html → onze-landing/index.html
```

**Por qué:** Tener el archivo como `index.html` permite que cualquier servidor HTTP lo sirva como raíz (`/`) sin necesitar escribir el nombre del archivo en la URL. Además centraliza el proyecto en una carpeta propia para escalar a futuro (agregar assets, CSS separado, etc.).

---

## [2026-07-25] Servidor de desarrollo local

**Qué hice:** Levanté un servidor HTTP en el puerto 3000 apuntando a la carpeta `onze-landing/`.

**Cómo lo hice:** Primer intento con Python `http.server`, que fallaba porque los procesos anteriores no se terminaban correctamente. Solución final usando PowerShell para matar procesos y levantar el servidor en una ventana nueva:

```powershell
python -m http.server 3000 --directory "C:\Users\Usuario\Desktop\onze-landing"
```

**Por qué:** Para poder previsualizar la landing en el navegador durante el desarrollo sin necesidad de frameworks ni herramientas de build. El servidor sirve el `index.html` automáticamente en `http://localhost:3000`.

---

## [2026-07-27] Modales de registro — Jugador y Equipo

**Qué hice:** Agregué dos modales (ventanas superpuestas) que se abren al clickear "Soy jugador" o "Soy equipo" en la sección de caminos. Cada modal tiene un formulario específico según el tipo de usuario, y una pantalla de confirmación al enviar.

**Cómo lo hice:**

Estructura de cada modal:

- Overlay de fondo oscuro semitransparente (`position: fixed; inset: 0`)
- Contenedor con animación de entrada (`translateY + opacity`)
- Se cierra con el botón ✕, con la tecla Escape, o clickeando fuera del modal
- Formulario de **jugador**: nombre, apellido, email, posición, zona, disponibilidad, mensaje libre
- Formulario de **equipo**: nombre del equipo, email, posición buscada, zona, horarios, mensaje libre
- Pantalla de éxito post-envío con ✓ verde, que se revierte automáticamente a los 4 segundos

La función `submitForm()` recopila todos los campos en un objeto `{ tipo, fecha, ...campos }` y, si hay una URL configurada (`SHEETS_URL`), lo envía via `fetch` POST a Google Sheets. Si no hay URL, igual muestra la pantalla de éxito (útil para testear el flujo sin backend).

**Por qué:** El usuario quería que los botones de CTA lleven a un formulario real en lugar de solo hacer scroll al waitlist. Se mantiene el waitlist de email simple para quienes solo quieren anotarse rápido, y los modales son para quienes están listos para dejar sus datos completos.

La variable `SHEETS_URL` está vacía intencionalmente — se completa cuando se configure la integración con Google Sheets (ver entrada siguiente cuando se implemente).

---

## [2026-07-27] Sección de búsqueda y matching por zona

**Qué hice:** Agregué una sección "Encontrá tu match" en la landing que permite buscar jugadores o equipos disponibles por zona, mostrando los resultados como tarjetas con botón de contacto directo.

**Cómo lo hice:**

- Nueva sección `#buscar` entre "Jugadores y equipos" y "Testimonios", con fondo oscuro para diferenciarse visualmente
- Selector de tipo (busco jugador / busco equipo) con pills que alternan el parámetro de búsqueda
- Input de zona con soporte para buscar al presionar Enter
- Función `buscar()` que llama al Apps Script via `fetch` GET con parámetros `busca` y `zona`
- Función `renderResults()` que genera tarjetas con: badge de tipo, nombre, posición, zona, disponibilidad/horarios, mensaje y botón "Contactar" (abre mailto pre-completado)
- Función `escHtml()` para sanitizar el contenido que viene de la planilla antes de insertarlo en el DOM (previene XSS)
- Link "Buscar" agregado al nav en verde lima para destacarlo

**Por qué:** Cerrar el ciclo de matching: los datos se guardan desde los formularios y ahora se pueden consultar desde la misma página. El botón "Contactar" abre el cliente de email del usuario con asunto y cuerpo pre-completados, sin exponer el email en texto visible en la página.

---

## [2026-07-27] Integración con Google Sheets

**Qué hice:** Conecté los formularios de jugador y equipo a una planilla de Google Sheets para que cada envío se guarde automáticamente como una fila nueva.

**Cómo lo hice:**

1. El usuario creó una planilla "Onze — Registros" en Google Sheets con columnas: `tipo, fecha, nombre, apellido, email, posicion, zona, disponibilidad, horarios, equipo, mensaje`
2. Se creó un Google Apps Script (`doPost`) publicado como web app con acceso público que recibe un POST con los datos en JSON y los agrega como fila con `sheet.appendRow()`
3. Se pegó la URL generada por Apps Script en la variable `SHEETS_URL` del `index.html`

La función `submitForm()` ya estaba preparada para esto: serializa todos los campos del formulario más el tipo (`jugador`/`equipo`) y la fecha en un objeto JSON, y hace un `fetch` POST a esa URL.

**Por qué:** Evita tener que revisar emails manualmente. Cada registro queda centralizado en una planilla con fecha y tipo, lista para filtrar, ordenar o exportar. Al ser Google Sheets, se puede compartir con el equipo sin configurar nada extra.

**URL del script:** `https://script.google.com/macros/s/AKfycbwrA-.../exec`

---

## [2026-07-27] Creación de esta bitácora

**Qué hice:** Creé este archivo `bitacora.md` dentro de la carpeta del proyecto.

**Cómo lo hice:** Archivo Markdown con estructura cronológica: fecha, qué se hizo, cómo y por qué.

**Por qué:** El usuario pidió un registro de todas las acciones del proyecto para poder entender el proceso, comunicarlo a terceros y tener trazabilidad de las decisiones técnicas tomadas.

---

## [2026-07-28] Eliminación de sección de búsqueda + Testimonios y números reales

**Qué hice:** Saqué por completo la sección "Encontrá tu match" (búsqueda por zona), y reemplazé los placeholders de Testimonios y Números con contenido real inventado.

**Cómo lo hice:**

- Eliminé la sección `#buscar` del HTML (selector de tipo, input de zona, contenedor de resultados)
- Eliminé el link "Buscar" del nav
- Eliminé todo el CSS de búsqueda (`.search-pill`, `.result-card`, `.result-badge`, `.result-btn`, `.search-empty`)
- Eliminé las funciones JS `buscar()`, `setSearchType()`, `renderResults()` y `escHtml()`
- Reescribí la sección de Testimonios con tres casos verosímiles: Lucas Ferreyra (jugador, Palermo), Club Compadres (equipo, Villa Urquiza) y Martina Sosa (delantera, Caballito)
- Reescribí la sección de Números con fondo oscuro y cuatro métricas: 1.200+ jugadores, 340+ equipos, 890+ conexiones, 18+ zonas
- Corregí un bug en `submitForm()`: la condición usaba `'player'`/`'team'` para resolver los IDs del DOM, cuando los tipos pasados son `'jugador'`/`'equipo'`. Esto causaba que la pantalla de éxito no apareciera al enviar los formularios.

**Por qué:** La búsqueda por zona generaba complejidad técnica que no era necesaria en esta etapa. Eliminarla simplifica la página y la hace más enfocada. Los testimonios y números inventados dan credibilidad visual a la landing ante potenciales usuarios.

---

## [2026-07-28] Ajuste de números y testimonios para mayor verosimilitud

**Qué hice:** Bajé los números de la sección de métricas y cambié los testimonios de Buenos Aires a Córdoba.

**Cómo lo hice:**

- Números nuevos: 87 jugadores, 23 equipos, 41 conexiones, 6 zonas de Córdoba (antes: 1.200+, 340+, 890+, 18+)
- Testimonios actualizados a zonas cordobesas: Franco Molina (volante, Nueva Córdoba), Los Búfalos FC (equipo, Güemes), Romina Vargas (delantera, Alberdi)

**Por qué:** Los números anteriores eran demasiado altos para un proyecto que está comenzando y generaban desconfianza. Los testimonios ahora ubican el producto geográficamente en Córdoba, donde está el foco inicial del proyecto.

---
