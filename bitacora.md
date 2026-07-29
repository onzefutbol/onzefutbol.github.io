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

## [2026-07-28] Publicación en GitHub Pages

**Qué hice:** Subí la landing a GitHub Pages y quedó accesible en la URL pública `https://onzefutbol.github.io`.

**Cómo lo hice:**

1. Creé cuenta de GitHub con username inicial `onzeappcba`, después renombrada a `onzefutbol` para que fuera más profesional.
2. Inicialicé el repositorio local con `git init` en la carpeta `onze-landing`.
3. Hice el primer commit con `git add index.html bitacora.md` + `git commit`.
4. Creé el repositorio remoto en GitHub con el nombre `onzefutbol.github.io` — cuando el nombre del repo coincide con `<username>.github.io`, Pages publica en la URL raíz sin path adicional.
5. Configuré el remote con token de acceso personal (PAT) para poder pushear sin autenticación interactiva.
6. `git push -u origin master` subió los archivos.
7. En Settings → Pages del repo activé la publicación desde la branch `master` en la raíz.

**Por qué:** Necesitábamos una URL pública para compartir la landing con terceros sin depender del servidor local. GitHub Pages es gratis, no requiere configuración de hosting, y la URL `onzefutbol.github.io` es lo suficientemente corta y memorable para compartir por WhatsApp, Instagram bio, etc.

---

## [2026-07-28] Extracción del logo como archivo SVG independiente

**Qué hice:** Creé `onze-logo.svg` en el Escritorio con el logo del doble chevron listo para abrir en Adobe Illustrator.

**Cómo lo hice:** El logo estaba embebido inline en varios lugares del `index.html` (nav, hero, footer). Lo extraje a un archivo SVG standalone de 200×200 con los dos paths vectoriales: chevron izquierdo oscuro y chevron derecho lima. Los colores están en HEX (no oklch) para máxima compatibilidad con software de diseño.

**Por qué:** El usuario necesita el logo vectorial para usarlo en Adobe Illustrator al crear placas para Instagram y TikTok. Al ser SVG, se puede escalar sin perder calidad y editar cada path por separado.

---

## [2026-07-28] Documentos de sistema de diseño, marca y brief social

**Qué hice:** Generé tres documentos markdown en la carpeta del proyecto para pasarle contexto completo a Claude Design y poder generar placas publicitarias consistentes:

- `design-system.md` — tokens técnicos: colores oklch, tipografías, escalas de spacing, componentes, radios.
- `brand-system.md` — identidad de marca: voz, tono, valores, público objetivo, do's & don'ts visuales.
- `social-brief.md` — brief específico para la primera tanda de 15 piezas (5 feed posts, 4 stories, 2 portadas TikTok, foto de perfil, 3 highlight covers).

**Cómo lo hice:** Los tres documentos son consumibles por Claude Design directamente. El design system extrae los valores exactos que ya están en el `index.html`. El brand system codifica decisiones que hasta ahora vivían solo en las conversaciones. El social brief detalla cada pieza con mensaje, layout sugerido, y assets a adjuntar.

**Por qué:** El usuario quiere generar placas de Instagram y TikTok con Claude Design. Sin un sistema de diseño y marca escrito, cada pieza generada podría interpretar la identidad de forma distinta. Con los tres documentos + el logo SVG + una captura de la landing, Claude Design tiene todo el contexto para producir piezas consistentes entre sí y con la landing.

Separé branding de design system (en vez de un solo doc) porque son reutilizables independientemente — el design system sirve para cualquier producto Onze (landing, app, email), y el brand system sirve para cualquier canal (redes, contenido, PR).

---

## [2026-07-28] Rediseño del hero, footer y formularios (matching pro)

**Qué hice:**

- **Hero rediseñado:** saqué el badge "Pre-lanzamiento" y el bloque de waitlist. En su lugar puse un grid de 4 cards clickeables (Fútbol 5, 7, 9 y 11), cada una con etiqueta de categorías disponibles (Masc · Fem · Mixto). Al clickear cualquier card se abre el modal de jugador con el tipo pre-seleccionado.
- **Nav actualizado:** el botón "Sumarme" ahora dice "Enterate primero" y hace scroll al bloque de suscripción en el footer.
- **Footer reescrito:** cambié el título "¿Le damos para adelante?" por "Enterate primero" y el subtítulo de validación de idea por una invitación a suscribirse a novedades. Reemplacé el link viejo por un mini-form inline (email + botón Suscribirme) con estado de confirmación.
- **Copy actualizado:** reemplacé "te mostramos", "te mandamos" y "te conectamos" por lenguaje de matching real ("hacemos match", "te matcheamos") en 5 lugares: paso 02 de "Cómo funciona", intro del modal jugador, intro del modal equipo, y ambos mensajes de éxito.
- **Nuevos campos en formularios (jugador y equipo):** agregué tres campos: `tipo_futbol` (select con Fútbol 5/7/9/11), `categoria` (select con Masculino/Femenino/Mixto), y `modalidad` (checkboxes múltiples: partido en el momento / próximos días / torneo).
- **submitForm() mejorado:** ahora concatena los valores múltiples de modalidad en un string separado por comas, y valida que al menos una modalidad esté marcada antes de enviar.
- **Función `openPlayerModalWithType()`:** nueva función que abre el modal de jugador y pre-selecciona el tipo de fútbol basándose en la card que clickeó el usuario.

**Cómo lo hice:** Todos los cambios en `index.html` (archivo único). Se reutilizaron las clases CSS existentes (`.form-group`, `.form-row`, `.form-input`, `.modal`, `.btn-submit`) para mantener consistencia visual. Solo agregué una clase nueva `.match-card:hover` en el CSS para el efecto hover de las cards del hero.

**Por qué:**

- Las cards de tipos de partido bajan la fricción de entrada: el usuario ve inmediatamente los formatos que Onze cubre y con un click ya está en el flow de registro con contexto.
- Sacar el mensaje de "ayudanos a decidir si construimos" corrige un tono de validación que ya no aplica — Onze ya está construido y funcionando, ahora comunicamos actividad y crecimiento.
- El copy de matching refleja con más precisión el modelo del producto: no es un tablón donde ves opciones y elegís, es un sistema que empareja según compatibilidad.
- Los 3 campos nuevos suben la calidad del matching: sin conocer tipo de fútbol y categoría, todas las conexiones eran ruido. La modalidad múltiple maximiza los matches posibles por jugador.

**Pendiente asociado (acción manual):** actualizar el Google Sheet agregando las columnas `tipo_futbol`, `categoria` y `modalidad`, y actualizar la función `doPost` del Apps Script para incluirlas en el `appendRow()`. Sin esto, los datos nuevos se envían pero no se guardan.

---

## [2026-07-29] Ajustes de nav y testimonial

**Qué hice:**

- **Nav — botón CTA:** renombré "Enterate primero" a "Sumarme" (más directo, mismo destino `#suscribite`).
- **Nav — logo clickeable:** envolví el logo SVG + texto "Onze" en un `<a href="#">` para que al clickear vuelva al inicio de la página.
- **Testimonio:** cambié el segundo testimonio de "Los Búfalos FC · Güemes" a "Anticresis · Villa Allende", con avatar "AN" en lugar de "LB".

**Por qué:** el botón "Sumarme" es más accionable y coincide con el lenguaje del resto del sitio. El logo sin link es un antipatrón de UX. El testimonio actualizado usa un nombre real de equipo y una zona geográfica de Córdoba más específica.

---

## [2026-07-29] Modales compactos + botón Cancelar

**Qué hice:**

- **Modal más chico:** reduje `max-width` de 500 a 440px, `padding` de 40 a 24/28px, `border-radius` de 20 a 16px. Agregué `max-height: calc(100vh - 40px)` con `overflow-y: auto` para que en pantallas bajas se pueda scrollear internamente en vez de cortar el formulario.
- **Escala tipográfica y de campos ajustada:** `h2` 22→19px, `p` 14→13px, `label` 12→11px, `form-input` padding 11/14→9/12px y font 14→13px, `form-group margin-bottom` 14→10px. La sensación general es un formulario más denso pero legible.
- **Botón Cancelar:** creé `.form-actions` (flex row) y `.btn-cancel` (transparente con borde) para que el submit y el cancelar convivan al pie de cada formulario. Los dos formularios (jugador y equipo) tienen el par de botones; "Cancelar" cierra el modal.
- **Textos de botones acortados:** "Enviar mi perfil" → "Enviar", "Publicar búsqueda" → "Publicar" (con menos ancho por card, textos más cortos leen mejor).

**Por qué:** el modal ocupaba casi toda la pantalla y forzaba a scroll dentro de una overlay incómoda; achicándolo se ve más liviano y profesional. La ausencia de un botón explícito de cancelar dejaba solamente la ✕ chica de la esquina, que muchos usuarios no ven — un botón "Cancelar" al lado del submit reduce la ansiedad de "estoy comprometido a enviar esto".

---

## [2026-07-29] Links reales de redes sociales en el footer

**Qué hice:** reemplacé los `<span>` estáticos del footer por `<a>` reales apuntando a Instagram (`https://www.instagram.com/onze_arg/`) y TikTok (`https://www.tiktok.com/@onzearg`). Ambos abren en pestaña nueva (`target="_blank" rel="noopener"`). Saqué el link de LinkedIn porque todavía no hay cuenta.

**Por qué:** ya existen las cuentas oficiales `@onze_arg` (IG) y `@onzearg` (TikTok), así que el footer ahora es funcional en vez de decorativo. Menos ítems y todos operativos = mejor UX y menos ruido.

---

## [2026-07-29] Google Analytics 4 integrado

**Qué hice:** agregué el snippet de GA4 (`gtag.js`) en el `<head>` del `index.html`, con el ID de medición `G-VZQWKQTS0B`. Creé la propiedad "Onze Landing" en Google Analytics con zona horaria Argentina y moneda ARS, y configuré el flujo de datos web apuntando a `https://onzefutbol.github.io`.

**Por qué:** hasta ahora sabíamos cuánta gente completaba el formulario (por el sheet), pero no cuánta *entraba* a la landing sin registrarse. GA4 nos da esa base para medir tasa de conversión real (visitas → registros), fuente de tráfico (Instagram, TikTok, directo), dispositivo, ubicación, y comportamiento. Es la primera pieza de analítica seria del proyecto.

**Cómo verificar:** entrar a analytics.google.com → Informes → En tiempo real → abrir onzefutbol.github.io en otra pestaña → tiene que aparecer 1 usuario activo.

---

## [2026-07-29] Fix responsive — landing rota en móvil

**Qué hice:**

- Agregué `overflow-x: hidden` al `body` como safety net contra scroll horizontal.
- Bloque `@media (max-width: 640px)` con reglas específicas:
  - Nav: `padding` 40→16px, `gap` 32→12px. Oculto los links del medio ("Cómo funciona", "Jugadores y equipos") con la clase `.nav-links` — dejo solo el logo y el CTA "Sumarme".
  - Todas las `<section>` bajan de 40 a 20px de padding horizontal.
  - Cards de "Para jugadores" y "Para equipos" (clase nueva `.caminos-card`): saco el `min-width:320px` que rompía en pantallas de 360-390px, y bajo padding de 48 a 24px.
  - `h1` del hero: 56 → 40px para no desbordar.

**Por qué:** en el iPhone del usuario, el nav se veía cortado con "Sumarme" fuera de pantalla y toda la landing tenía scroll horizontal. Las cards de caminos con `min-width:320px + padding:48px` sumaban 416px, más ancho que cualquier celu estándar → esto era la causa raíz del arrastre horizontal. El fix se limita a `@media (max-width: 640px)` para no afectar desktop.

---

## [2026-07-29] Responsive completo — mobile, tablet, desktop

**Qué hice:**

- **Reset defensivo:** `html, body` con `overflow-x: hidden`, `max-width: 100%`, `margin/padding: 0`. `section, footer, nav` con `width: 100%; max-width: 100%`. Esto evita cualquier scroll horizontal accidental sea cual sea la causa.
- **Sistema de 3 breakpoints coherente:**
  - `< 1024px` (tablet + mobile): padding lateral de secciones baja a 32px, hero un poco menos alto, H1 de 56→48px.
  - `641px–1024px` (tablet específico): match cards a 2 por fila, caminos apilados a 100% (ya que un split 50/50 quedaba apretado), testimonios a 2 por fila.
  - `< 640px` (mobile): 1 columna en todo, nav simplificado, tipografía más chica, paddings mínimos.
- **Clases nuevas para targeting limpio:** `.hero`, `.cf-grid`, `.testi-grid`, `.testi-card`, `.num-cell`, `.num-big`, `.card-num`, `.footer-title`, `.newsletter-form`. Se agregaron a los elementos existentes sin tocar la estructura HTML.
- **Reglas específicas del mobile:**
  - Nav: 12px de padding, botón "Sumarme" más chico (8×14, 12px de fuente).
  - Hero H1: 34px con letter-spacing -0.02em para que "JUGADORES" quepa.
  - Match cards: 2 por fila (`flex: 1 1 calc(50% - 8px)`).
  - Testimonios y caminos: 100% de ancho cada uno.
  - Números: grid 2×2 con bordes redistribuidos (derecho solo en columna impar, inferior en filas superiores).
  - Newsletter form: apila email + botón verticalmente.
  - Modales: padding 20px, overlay con padding 10px, `form-row` apila los campos.

**Por qué:** las reglas anteriores eran parches específicos que dejaban muchos casos sin cubrir (tablet, botón grande del nav apretado en mobile, testimonios con `min-width:260px` que no bajaban, números en 4 columnas horrorosas en 400px). Este bloque es un sistema completo y consistente, más fácil de mantener.

---

## [2026-07-29] Fix desalineamiento formularios ↔ Google Sheet

**Qué hice (parte landing):**

- Renombré el campo `disponibilidad` (jugador) y `horarios` (equipo) a un único nombre común `horario`, que matchea la columna del sheet.

**Qué queda pendiente (acción manual del usuario en Google Sheet + Apps Script):**

1. Agregar 3 columnas nuevas al sheet: `TIPO_FUTBOL`, `CATEGORIA`, `MODALIDAD`.
2. Reemplazar el código del Apps Script por una versión que mapea por nombre de columna (no por orden), lo que elimina el desalineamiento estructural.
3. Redesplegar como "New version" del deployment existente (mantiene la URL, no hay que tocar la landing).

**Por qué:** el problema raíz era que el `doPost` viejo apilaba los valores en orden fijo, entonces cuando la forma del payload cambiaba (más campos, distinto orden), todo se corría. El nuevo lee los headers y mapea por nombre → cualquier campo del payload cae en la columna correcta sin importar el orden. Beneficio adicional: en el futuro se puede agregar/reordenar columnas del sheet sin tocar el script ni la landing.

---

## [2026-07-29] Modales móvil compactos + flujo de éxito arreglado

**Qué hice:**

- **Modal más compacto en mobile:** bajé padding de 20 a 18/16, `h2` a 17px, `p` a 12px, `form-group` margin a 8px, `label` a 10px, `form-input` padding a 8/10 con font 13px, botones más chicos (11px padding, 12px font), close (✕) más chico y arriba. Además `align-items: flex-start` en el overlay para que el modal empiece pegado arriba en vez de centrado — así en pantallas cortas queda más natural con el scroll interno.
- **Card de éxito compacta en mobile:** check circle de 56 → 48px, h3 de 20 → 17px, párrafo a 13px, padding 16 en vez de 20.
- **Flujo de éxito nuevo:** en vez de mostrar el mensaje 4 segundos y re-abrir el formulario (que dejaba al usuario mirando otra vez el mismo form, confuso), ahora:
  1. Se muestra el mensaje de éxito durante 2.5 segundos.
  2. Auto-cierra el modal completo.
  3. Reset del form en background 300ms después (para que la próxima apertura arranque limpia).
- **Scroll del modal al top** al cambiar a la vista de éxito, así el usuario no queda mirando el pie del formulario cuando aparece el mensaje.

**Por qué:** en pantallas de celular el modal ocupaba casi toda la altura y quedaba con scroll interno, y encima al terminar el flujo se re-mostraba el formulario vacío por 4 seg más — el usuario no entendía qué estaba pasando. El auto-cierre es más natural: enviaste, viste el "listo", el modal se va, seguís navegando la landing. Corta la fricción y no te "traba" con un formulario que reaparece.

---
