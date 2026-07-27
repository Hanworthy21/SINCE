# Estándares de front-end — proyectos con Julian Franco (Klerpson)

Reglas obligatorias al generar o modificar código en este proyecto y en los siguientes
del mismo tipo (landings y sitios estáticos). Nacen de las correcciones que se hicieron
sobre la entrega de la landing de Since.

La explicación larga, con ejemplos y referencias a líneas, está en `GUIA-ESTUDIO.md`.
Este archivo es solo la lista de reglas.

---

## 1. Estructura

- **Cero CSS en el HTML.** Ni atributos `style="..."` ni bloques `<style>`. Debe cumplirse
  siempre: `grep -c 'style="' index.html` → `0`.
- **Para un ajuste puntual, crear una clase utilitaria** con prefijo `u-`
  (`.u-mt-sm`, `.u-hidden`, `.u-center-2`), nunca un inline.
- **Nunca pegar parches al final de la hoja de estilos.** Prohibidos los bloques tipo
  `/* FIX: ... */` o `/* AJUSTES: ... */` al final del archivo. Toda regla nueva va
  **dentro de la sección del componente al que pertenece**. Un archivo que crece por el
  final es un archivo que nadie mantiene.
- **Un selector, un solo lugar.** Si `.nav-actions` ya existe, se edita ahí; no se
  redeclara 400 líneas después.
- **Orden de la hoja de estilos**, de lo general a lo específico:
  `tokens → base/reset → layout → componentes (en el orden de la página) → animaciones → responsive`.
- **Nunca `!important`.** Si hace falta, el problema es la especificidad o un estilo inline.
- **Una fuente de verdad para los datos repetidos** (teléfono, URL, correo): una constante
  en JS o un token en CSS, y el HTML solo declara lo que varía.

## 2. HTML semántico y SEO

- **Landmarks siempre:** `<header>`, `<main>`, `<section>`, `<aside>`, `<footer>`. Un `<div>`
  solo cuando de verdad no hay etiqueta con significado.
- **Un solo `<h1>` por página**, con la palabra clave real. Sin saltos de nivel
  (`h2 → h4` está mal). El tamaño se ajusta con CSS, nunca eligiendo otro nivel.
- **Cada `<section>` con `aria-labelledby`** apuntando al `id` de su encabezado.
- **Listas de verdad:** secuencias numeradas en `<ol>`, conjuntos en `<ul>`. No `<div>`
  simulando lista.
- **Formularios:** `<label for>` ligado al `id` del input en todos los campos. El
  `placeholder` nunca reemplaza al label. `type` correcto (`tel`, `email`, `number`) para
  que el móvil abra el teclado adecuado.
- **Imágenes** con `alt` descriptivo; `alt=""` solo si son decorativas. Iconos inline con
  `aria-hidden="true"`.
- **Metas obligatorias:** `<html lang>`, `<title>` ≤ 60 caracteres, `meta description`
  ≤ 155, `canonical`, Open Graph completo con imagen 1200×630, Twitter Card.
- **JSON-LD siempre**, con las entidades que apliquen (`Organization`, `Service`,
  `FAQPage`, `BreadcrumbList`), enlazadas por `@id` para no duplicar datos.
  **El JSON-LD debe decir exactamente lo mismo que la página visible** — declarar cosas que
  no están en pantalla se considera spam estructurado.
- **SVG repetidos** → sprite con `<symbol>` + `<use>`, definido una vez.

## 3. CSS

- **Todos los colores, sombras, radios, tipografías y anchos son tokens en `:root`.**
  Ningún valor hexadecimal suelto en el cuerpo de la hoja. Nombres semánticos (`--ink`,
  `--gold-deep`), no literales (`--negro`).
- **Variante de contraste obligatoria** para colores de marca que no pasan AA en texto
  pequeño (por eso existen `--gold` y `--gold-deep`).
- **`clamp()` para tipografía y espaciado**, no media queries. Los breakpoints se reservan
  para **reordenar el layout**, no para redimensionar texto.
- **Máximo dos breakpoints** (≈960px tablet, ≈640px móvil), salvo justificación real.
- **Grid con `gap`** para las cuadrículas. Nada de márgenes entre hijos.
- **Componentes como base + modificadores** (`.btn` / `.btn-primary` / `.btn-lg`), nunca
  clases monolíticas tipo `.boton-verde-grande`.
- **Propiedades lógicas** (`padding-inline`, `margin-inline`, `padding-block`) sobre
  `left/right/top/bottom`.
- **`@supports` para toda propiedad no universal** (`backdrop-filter`, `mask-image`) con
  un fallback sólido que mantenga legibilidad y contraste.
- **`prefers-reduced-motion` obligatorio.** Y al apagar las animaciones, forzar
  `opacity:1; transform:none` en los elementos animados: si no, el contenido queda
  invisible para siempre.
- **Fuentes en `.woff2` externo con `font-display:swap`**, no en base64. El base64 bloquea
  el render e infla el peso ~33%. Solo se embebe si el cliente lo exige por privacidad, y
  se documenta la razón.

## 4. Móvil (aquí es donde se rompe todo)

- **Cero scroll horizontal.** Probar quitando `overflow-x:hidden` temporalmente: si la
  página se arrastra de lado, hay un elemento desbordado que se arregla **de raíz**.
  `overflow-x:hidden` es red de seguridad, no solución.
- **`min-width:0` en los hijos de flex que contengan texto largo** (correos, URLs), junto a
  `overflow-wrap:anywhere`. Sin el `min-width:0` un hijo de flex no se encoge por debajo de
  su contenido y `overflow-wrap` no alcanza.
- **Áreas tocables ≥ 44×44px.**
- **Coordinar los elementos fijos** para que no se pisen entre sí, y añadir
  `padding-bottom` al `body` igual a la altura de la barra fija, o el footer queda tapado.
- **`env(safe-area-inset-bottom)`** en toda barra fija inferior (barra de gestos del iPhone).
- **`scroll-margin-top`** en las secciones con `id` cuando el header es `sticky`, o los
  anclajes quedan debajo del header.
- **Probar a 375px** (iPhone SE) y, antes de entregar, **en un celular físico**: el
  `safe-area`, el teclado y el `100vh` de Safari no se reproducen en el simulador.

## 5. Antes de dar algo por terminado

- [ ] `grep -c 'style="' index.html` → 0
- [ ] Ningún bloque `FIX`/`AJUSTE` al final del CSS
- [ ] Un solo `h1`, jerarquía sin saltos
- [ ] JSON-LD validado en <https://search.google.com/test/rich-results> y coherente con la página
- [ ] Sin scroll horizontal a 375px, probado sin `overflow-x:hidden`
- [ ] Contraste AA (4.5:1 normal, 3:1 grande)
- [ ] Con `prefers-reduced-motion` el contenido sigue visible
- [ ] Navegable solo con `Tab`, foco siempre visible
- [ ] Lighthouse ≥ 90 en Performance, Accesibilidad, SEO y Buenas prácticas

## 6. Convenciones del repo

- Ramas de trabajo: `claude/<descripción-corta>`.
- Commits y comentarios de código **en español**.
- No abrir Pull Request salvo que se pida explícitamente.
