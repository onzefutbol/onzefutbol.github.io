# Onze — Design System

Sistema de diseño de la landing de Onze. Estos son los tokens y componentes que definen la identidad visual del producto. Usarlos consistentemente en todas las piezas.

---

## 1 · Paleta de colores

Todos los colores están en formato **oklch** (percepción uniforme, mejor gradación). Cuando la herramienta no soporte oklch, usar los equivalentes HEX indicados.

### Primarios

| Rol | oklch | HEX aprox | Uso |
|-----|-------|-----------|-----|
| Lima principal | `oklch(0.75 0.19 130)` | `#B3F000` | CTAs, botones, acentos, links en dark |
| Lima hover | `oklch(0.68 0.19 130)` | `#9FD800` | Estado hover de botones lima |
| Lima oscuro (headings) | `oklch(0.5 0.16 130)` | `#6A8D00` | Títulos de sección sobre fondo blanco, links en light |
| Lima muy claro | `oklch(0.85 0.03 130)` | `#D8E5B8` | Números decorativos, elementos suaves |
| Lima 15% opacidad | `oklch(0.75 0.19 130 / 0.15)` | rgba(179,240,0,0.15) | Fondos de badges, halos suaves |

### Neutros oscuros (fondo dark)

| Rol | oklch | HEX aprox | Uso |
|-----|-------|-----------|-----|
| Dark base | `oklch(0.15 0.01 80)` | `#242220` | Fondo principal de secciones dark, footer, nav |
| Dark card | `oklch(0.12 0.01 80)` | `#1D1B1A` | Cards, modales |
| Dark deep | `oklch(0.08 0.01 80)` | `#141312` | Inputs sobre dark, sección más profunda |
| Dark elevated | `oklch(0.19 0.01 80)` | `#2C2A28` | Waitlist card sobre hero |
| Dark elevated 2 | `oklch(0.22 0.01 80)` | `#332F2D` | Avatares oscuros de testimonios |

### Neutros claros y grises

| Rol | oklch | HEX aprox | Uso |
|-----|-------|-----------|-----|
| Blanco | `white` | `#FFFFFF` | Fondo de secciones light, texto sobre dark |
| Off-white | `oklch(0.97 0 0)` | `#F5F5F5` | Fondo de sección testimonios |
| Gris texto oscuro | `oklch(0.3 0.01 80)` | `#4A4745` | Cuerpos de texto sobre fondos claros |
| Gris texto medio | `oklch(0.5 0.01 80)` | `#7A7673` | Metadata, footers de card |
| Gris texto suave | `oklch(0.55 0.01 80)` / `oklch(0.6 0.01 80)` | `#8A8683` | Textos secundarios sobre dark |
| Gris texto light | `oklch(0.65 0.01 80)` / `oklch(0.72 0.01 80)` | `#A6A29F` | Descripciones sobre dark |
| Gris borde dark | `oklch(0.25 0.01 80)` / `oklch(0.3 0.01 80)` | `#3E3B39` | Bordes sobre fondos oscuros |

### Acento amarillo (rating)

| Rol | oklch | HEX aprox | Uso |
|-----|-------|-----------|-----|
| Amarillo estrellas | `oklch(0.7 0.19 90)` | `#D4B300` | Solo para el rating de 5 estrellas en testimonios |

---

## 2 · Tipografía

### Familias

- **Space Grotesk** (Google Fonts, pesos 500 y 700) — headings, títulos de sección, botones, todo lo que va en **MAYÚSCULAS**. Geométrica, con carácter técnico/deportivo.
- **Manrope** (Google Fonts, pesos 500, 700 y 800) — cuerpos de texto, formularios, párrafos. Humanista, legible en tamaños chicos.

### Reglas de uso

- Space Grotesk **siempre** va en `text-transform: uppercase` con `letter-spacing: 0` o levemente cerrado (`-0.01em` en headings grandes).
- Manrope va en case normal (sin uppercase).
- Nunca mezclar dos pesos distintos en el mismo bloque de texto.
- Peso 700 es el default para Space Grotesk. Peso 500 solo para nav secundario.

### Escala tipográfica

| Rol | Familia | Tamaño | Peso |
|-----|---------|--------|------|
| Hero H1 | Space Grotesk | 56px | 700 |
| Números grandes (métricas) | Space Grotesk | 52px | 700 |
| Título footer CTA | Space Grotesk | 28px | 700 |
| Título de sección grande | Space Grotesk | 26px | 700 |
| Título modal | Space Grotesk | 22px | 700 |
| Título card / logo footer | Space Grotesk | 20-22px | 700 |
| Subtítulos / step titles | Space Grotesk | 18px | 700 |
| Nav logo | Space Grotesk | 17px | 700 |
| Sección label ("CÓMO FUNCIONA") | Space Grotesk | 14px + letter-spacing 0.1em | 700 |
| Texto hero / párrafos grandes | Manrope | 17px | 500 |
| Texto card / testimonios | Manrope | 15px | 500 |
| Texto general / nav links | Manrope | 14px | 700 |
| Botones, meta | Manrope | 13px | 700 |
| Labels de formulario | Manrope | 12px + letter-spacing 0.06em | 700 |
| Badges | Manrope | 11px + letter-spacing 0.08em | 700 |

---

## 3 · Escala de spacing

Basada en múltiplos de 4px. Usar estos valores, no valores arbitrarios.

`4 · 8 · 12 · 16 · 20 · 24 · 28 · 32 · 40 · 48 · 56 · 72 · 90 · 100 px`

### Padding típico por contexto

- Sección grande (hero, footer): `90-100px` vertical, `40px` horizontal
- Sección split (caminos): `72px` vertical, `48px` horizontal
- Card / modal: `28-40px` interior
- Botones: `12-14px` vertical, `20-26px` horizontal
- Form inputs: `11-13px` vertical, `14-16px` horizontal

---

## 4 · Border radius

| Valor | Uso |
|-------|-----|
| 8px | Botones, pills, form inputs, links del nav |
| 10px | Botones grandes de CTA, inputs destacados |
| 14px | Icon boxes cuadradas |
| 16px | Cards de testimonio, waitlist card |
| 20px | Modales |
| 999px | Avatares circulares, badges pill |

---

## 5 · Componentes clave

### 5.1 · Botón principal (CTA)

```
background: oklch(0.75 0.19 130)   [lima]
color: oklch(0.15 0.01 80)         [dark base]
padding: 12px 22px  (o 14px si es full-width)
border-radius: 8px  (o 10px si es grande)
font: Manrope 13px 700 uppercase, letter-spacing 0.05em
hover: background oklch(0.68 0.19 130)
```

### 5.2 · Botón secundario (dark sobre lima)

```
background: oklch(0.15 0.01 80)
color: white
resto igual al CTA
```

### 5.3 · Pill activa / inactiva

Pill inactive:
```
background: oklch(0.15 0.01 80)
color: oklch(0.65 0.01 80)
border: 1px solid oklch(0.3 0.01 80)
padding: 9px, border-radius: 8px
```

Pill active:
```
background: oklch(0.75 0.19 130)
color: oklch(0.15 0.01 80)
border: none
```

### 5.4 · Badge

```
padding: 4px 12px
border-radius: 999px
font: Manrope 11px 700 uppercase, letter-spacing 0.08em
```

Variantes:
- **Badge lima**: `background: oklch(0.75 0.19 130 / 0.15)` + `color: oklch(0.75 0.19 130)`
- **Badge dark**: `background: oklch(0.15 0.01 80)` + `color: oklch(0.75 0.19 130)` + `border: 1px solid oklch(0.3 0.01 80)`

### 5.5 · Card de testimonio

```
background: white
border-radius: 16px
padding: 32px
border: 1px solid oklch(0.92 0 0)
gap interno: 16px
```

Estructura: avatar circular 44×44 (iniciales sobre fondo dark o lima) + nombre + zona + cita en italic + 5 estrellas amarillas.

### 5.6 · Patrón de rayas diagonales

Fondo decorativo secundario:
```
background: repeating-linear-gradient(
  45deg,
  oklch(0.2 0.01 80), oklch(0.2 0.01 80) 10px,
  oklch(0.26 0.01 80) 10px, oklch(0.26 0.01 80) 20px
);
```

Usar para: placeholders de imagen, superficies decorativas, backgrounds de placas donde va la foto real.

---

## 6 · Logo

Doble chevron geométrico, dos paths vectoriales:

- **Chevron izquierdo (blanco/oscuro)**: apunta hacia abajo-derecha
- **Chevron derecho (lima)**: apunta hacia arriba-derecha (espejado)

Los dos chevrons no se tocan — hay un espacio entre ellos. Simbolizan dos entidades (jugador y equipo) que se conectan.

Archivo vectorial: `onze-logo.svg` (en el Escritorio).

**Variantes de uso:**
- Sobre fondo dark: chevron izquierdo blanco + chevron derecho lima
- Sobre fondo lima: chevron izquierdo dark + chevron derecho blanco (no usado aún, pero disponible)
- Sobre fondo blanco: chevron izquierdo dark + chevron derecho lima oscuro `oklch(0.5 0.16 130)`

**Tamaños mínimos:**
- Nav: 28px
- Footer: 40px
- Hero decorativo: 320px con 8% opacidad
- Placas sociales: 60-120px según formato

---

## 7 · Iconografía complementaria

La landing usa emojis puntuales (📍 🕐 ✓ ✕ ★) como iconos rápidos. Para placas de social se recomienda:
- Mantener los emojis como recurso rápido, **o**
- Reemplazar por icons de línea 1.5px stroke, minimalistas, en color lima o blanco según fondo.

No mezclar emojis con icons ilustrados en la misma pieza.
