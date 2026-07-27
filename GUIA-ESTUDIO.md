# Guía de estudio — cómo está construida esta landing

Guía para entender **cómo se organiza un proyecto de landing page profesional**, usando
este mismo repo como caso de estudio. Tres ejes:

1. **Estructura del proyecto** — qué archivo hace qué y por qué.
2. **HTML semántico y SEO** — cómo se marca el contenido para que Google y los lectores
   de pantalla lo entiendan.
3. **CSS** — funciones avanzadas y reúso de clases para no repetir código.

Al final hay un cuarto bloque sobre **móvil** (lo que más se rompe) y una lista honesta de
**deudas técnicas que este repo todavía tiene**, que es donde más se aprende.

Todas las referencias son a archivos y líneas reales. Ábrelas mientras lees.

---

## 0. Mapa del proyecto

```
index.html               Todo el contenido + el JS de la página
assets/css/styles.css    Toda la piel visual (tokens, componentes, responsive)
assets/css/noscript.css  Estilos para cuando el usuario tiene JS desactivado
assets/img/              Logo, isotipo, favicon, imagen para redes (og-image)
robots.txt               Le dice a los buscadores qué pueden rastrear
sitemap.xml              Lista de URLs para que Google las encuentre
site.webmanifest         Metadatos si alguien "instala" el sitio en su celular
netlify.toml             Configuración de despliegue
README.md                Cómo desplegarlo y qué falta por reemplazar
```

**Regla mental:** cada archivo responde una sola pregunta.

| Archivo | Pregunta que responde |
|---|---|
| `index.html` | ¿Qué dice la página? |
| `styles.css` | ¿Cómo se ve? |
| `<script>` | ¿Cómo se comporta? |
| `robots/sitemap/manifest` | ¿Cómo la entienden las máquinas? |

Cuando un archivo empieza a responder dos preguntas, el proyecto se vuelve difícil de
mantener. De ahí sale la regla que está escrita en la cabecera del CSS
(`assets/css/styles.css:1-6`):

```
- Nunca escribir CSS embebido en el HTML.
- Los tokens de marca estan en :root (colores, tipografia, sombras).
```

---

## 1. Estructura del proyecto

### 1.1 Separación de capas

El principio se llama **separación de responsabilidades**: contenido (HTML), presentación
(CSS) y comportamiento (JS) viven aparte.

En este repo se cumple para el CSS: **cero atributos `style="..."` en el HTML**. Puedes
verificarlo tú mismo:

```bash
grep -c 'style="' index.html   # → 0
```

Eso no fue gratis. Cuando necesitas un ajuste puntual (un margen, un `text-align`), la
tentación es escribir `style="margin-top:.8rem"` ahí mismo. La solución correcta es crear
una **clase utilitaria**, que es lo que hay en `assets/css/styles.css:527-543`:

```css
.u-mt-sm{margin-top:.8rem}
.u-center-2{text-align:center;margin-top:2.2rem}
.u-hidden{display:none}
.u-strike{text-decoration:line-through}
```

El prefijo `u-` (de *utility*) es una convención: al leer el HTML sabes de inmediato que
esa clase hace **una sola cosa** y no forma parte de un componente.

**Por qué importa de verdad:** un `style` inline gana casi siempre en la cascada. El día
que quieras cambiar todos los márgenes desde el CSS, los inline no te van a obedecer y vas
a terminar poniendo `!important`. Ese es el inicio de una hoja de estilos imposible.

### 1.2 Orden dentro del CSS

`styles.css` va de lo general a lo específico:

```
1-6      Cabecera con las reglas del proyecto
9-10     Fuentes embebidas
13-47    :root → tokens de diseño
51-65    Base (reset, body, tipografía, focus)
67-74    Layout y helpers globales (.wrap, .eyebrow)
77-106   Botones (el componente más reutilizado)
109-127  Header
129-139  Secciones
141-449  Componentes, en el mismo orden en que aparecen en la página
451-457  Animaciones
459-513  Responsive
```

Ese orden **no es decorativo**: el CSS se aplica en cascada, así que lo general debe ir
antes que lo específico para que lo específico pueda sobrescribirlo sin `!important`.

Y el orden de componentes espejea el orden de la página, así que cuando ves algo raro en
la sección de planes, sabes que el CSS está por la mitad-final del archivo, no lo buscas a
ciegas.

### 1.3 El JS: un solo IIFE, sin dependencias

Todo el comportamiento está en `index.html:595-759`, envuelto en:

```js
(function(){
  "use strict";
  ...
})();
```

Eso se llama **IIFE** (función que se ejecuta sola). Sirve para que ninguna de tus
variables (`WA`, `reduce`, `io`…) se escape al ámbito global y choque con otro script.

Adentro está dividido por comentarios, un bloque por responsabilidad:

- Deep links de WhatsApp (`601-607`)
- Año del footer (`609-610`)
- Duplicado del marquee (`612-614`)
- Sombra del header + botón volver arriba (`616-624`)
- Reveal al hacer scroll (`627-652`)
- Contadores animados (`653-668`)
- Acordeón del FAQ (`670-679`)
- Chat animado (`681-737`)
- Formulario (`739-757`)

**Detalle que vale oro** — los deep links de WhatsApp (`index.html:601-607`):

```js
document.querySelectorAll("[data-wa]").forEach(function(el){
  var msg = encodeURIComponent(el.getAttribute("data-wa") || "Hola, quiero más información 👋");
  el.setAttribute("href", "https://wa.me/" + WA + "?text=" + msg);
});
```

El número de teléfono está **una sola vez**, en la constante `WA` (`index.html:598`). En el
HTML cada uno de los 6 botones solo declara su mensaje:

```html
<a class="btn btn-wa" data-wa="Hola, quiero una *demo gratis*..." href="#">
```

Si el cliente cambia de número, cambias **una línea** en vez de buscar 6 enlaces. Esto es
el mismo principio del reúso de clases, aplicado a datos: **una fuente de verdad**.

---

## 2. HTML semántico y SEO

### 2.1 Qué es "semántico"

Semántico significa que la etiqueta **describe qué es el contenido**, no cómo se ve. Un
`<div>` no significa nada; un `<header>`, `<main>`, `<section>`, `<footer>` sí.

Esqueleto de esta página:

```
<header class="site-header">      index.html:111   — cabecera del sitio
<main>                            index.html:126   — el contenido principal
  <section class="hero">          index.html:128
  <section id="problema">         index.html:202
  <section id="como-funciona">    index.html:233
  <section id="beneficios">       index.html:285
  <section id="casos">            index.html:316
  <section id="resultados">       index.html:335
  <section id="testimonios">      index.html:374
  <section id="planes">           index.html:414
  <section id="faq">              index.html:479
  <section id="contacto">         index.html:497
    <aside class="contact-side">  index.html:530   — contenido secundario
</main>
<footer class="footer">           index.html:556
```

Esos elementos se llaman **landmarks**. Un usuario con lector de pantalla puede saltar
directo a `main` sin escuchar el menú, igual que tú usarías `Ctrl+F`. Google también los
usa para saber qué parte de la página es el contenido real y qué es adorno.

### 2.2 Jerarquía de encabezados

Regla: **un solo `<h1>` por página**, y no te saltes niveles.

```
h1  Un agente de IA que vende por ti las 24 horas   index.html:135
├── h2  Cada minuto sin responder…                  index.html:206
├── h2  Tu agente listo para vender en 4 pasos      index.html:237
│   └── h3  Diagnóstico y diseño / Entrenamiento…   index.html:241-244
├── h2  La diferencia entre atender… y vender       index.html:254
├── h2  Elige cómo quieres empezar a vender con IA  index.html:418
│   └── h3  Starter / Growth / Pro                  index.html:423,436,448
└── h2  Preguntas frecuentes                        index.html:483
```

El `h1` es el título de **la página**, no el nombre de la empresa. Debe contener la
palabra clave real por la que quieres posicionar: *"agente de IA"*, *"vende"*, *"Colombia"*.

Error clásico: usar `h3` porque "el `h2` se ve muy grande". El tamaño se arregla con CSS
(`assets/css/styles.css:136`), la jerarquía es de significado.

### 2.3 `aria-labelledby`: conectar sección con su título

Fíjate en el patrón que se repite en cada sección (`index.html:128`, `202`, `233`…):

```html
<section class="section alt" id="problema" aria-labelledby="problema-title">
  ...
  <h2 id="problema-title">Cada minuto sin responder es una venta…</h2>
```

`aria-labelledby` apunta al `id` del `h2`. Con eso, un lector de pantalla no anuncia
"región" sin más, sino **"región: Cada minuto sin responder…"**. Es dos atributos de
trabajo y cambia por completo la navegación para alguien ciego.

### 2.4 Formularios accesibles

`index.html:507-528`:

```html
<div class="field">
  <label for="nombre">Nombre*</label>
  <input id="nombre" name="nombre" type="text" required placeholder="Tu nombre">
</div>
```

Tres cosas que hay que hacer siempre:

- **`for` del label = `id` del input.** Así, al tocar la etiqueta, el cursor entra al
  campo. En móvil eso agranda el área tocable, que es justo donde la gente falla.
- **`placeholder` no reemplaza al `label`.** El placeholder desaparece al escribir; si era
  tu única etiqueta, el usuario ya no sabe qué campo está llenando.
- **`type` correcto.** `type="tel"` en el WhatsApp (`index.html:517`) hace que el celular
  abra el **teclado numérico**. `type="email"` abre el teclado con `@`. Es gratis y mejora
  la conversión.

El campo trampa para bots (`index.html:511`) usa la utilidad `.u-hidden`, no un
`style="display:none"`:

```html
<p class="u-hidden"><label>No llenar: <input name="bot-field"></label></p>
```

### 2.5 SEO técnico: las metas

`index.html:4-34`. Cada una tiene un trabajo:

| Etiqueta | Para qué sirve |
|---|---|
| `<html lang="es">` | Idioma. Google segmenta por él; los lectores de pantalla eligen la voz. |
| `<title>` | El texto azul del resultado de Google. **~60 caracteres.** |
| `meta description` | El párrafo gris debajo. **~155 caracteres.** No posiciona, pero decide si te hacen clic. |
| `link rel="canonical"` | La URL "oficial". Evita que Google vea contenido duplicado. |
| `og:*` | Cómo se ve al compartir en WhatsApp/Facebook. Sin esto, sale un link pelado. |
| `twitter:*` | Lo mismo en X. |
| `og:image` 1200×630 | Tamaño estándar. Otro tamaño se recorta feo. |

El `<title>` de aquí (`index.html:6`) está bien construido:

```
Agente de IA de Ventas 24/7 en Colombia | Since Marketing Digital
[--- qué es + dónde ---------------------] | [--- marca ---------]
```

Beneficio y ubicación primero, marca al final. Nadie busca "Since Marketing Digital";
buscan "agente de IA para ventas".

> `meta keywords` (`index.html:8`) ya **no** lo usa Google desde 2009. No hace daño, pero
> no esperes nada de él.

### 2.6 Datos estructurados (JSON-LD)

`index.html:36-104`. Es el bloque más valioso para SEO y el que casi nadie hace.

Es un JSON que le explica a Google **qué significa** tu contenido, en el vocabulario de
[schema.org](https://schema.org). Aquí se declaran cuatro entidades en un `@graph`:

- **`Organization`** (`41-65`) — nombre, teléfono, email, redes (`sameAs`). Alimenta el
  panel de la derecha en Google.
- **`Service`** (`66-83`) — el servicio, dónde se presta y los `offers` con precio.
- **`FAQPage`** (`84-94`) — las preguntas frecuentes. **Esta es la que da resultados
  enriquecidos**: Google despliega las preguntas directamente en el buscador.
- **`BreadcrumbList`** (`95-103`) — la miga de pan (Inicio › Servicios › Agente de IA).

Fíjate en el truco de `@id` (`43` y `70`):

```json
{ "@type": "Organization", "@id": "https://agenciasincemarketing.com/#org", ... }
...
{ "@type": "Service", "provider": {"@id": "https://agenciasincemarketing.com/#org"} }
```

En vez de repetir todos los datos de la organización dentro del servicio, se **referencia**
por `@id`. Otra vez el mismo principio: una fuente de verdad, cero duplicación.

**Regla crítica:** el JSON-LD tiene que decir lo mismo que la página visible. Las seis
preguntas del `FAQPage` (`86-91`) son exactamente las seis del acordeón (`486-491`). Si
declaras cosas que no están en pantalla, Google lo considera *spam estructurado* y te
puede penalizar.

Valida siempre en <https://search.google.com/test/rich-results>.

### 2.7 Los otros archivos de SEO

- **`robots.txt`** — permite el rastreo y apunta al sitemap.
- **`sitemap.xml`** — lista de URLs. En un sitio de una página es casi trivial, pero es el
  primer sitio donde Google mira.
- **`site.webmanifest`** — nombre, iconos y colores si alguien agrega el sitio a su
  pantalla de inicio.

---

## 3. CSS: funciones avanzadas y reúso de clases

Este es el bloque donde más se separa el código amateur del profesional.

### 3.1 Tokens de diseño con variables CSS

`assets/css/styles.css:13-47`. En vez de escribir `#ddb308` en 40 lugares:

```css
:root{
  --paper:#ffffff;
  --cream:#f8f6f2;
  --ink:#0d1117;          /* azul-noche: texto */
  --ink-soft:#565d69;     /* texto secundario */
  --gold:#ddb308;         /* dorado de marca */
  --gold-deep:#b6860a;    /* dorado oscuro, contraste AA */
  --shadow-md:0 12px 32px -14px rgba(13,17,23,.22);
  --radius:18px;
  --maxw:1160px;
  --pad:clamp(20px,5vw,40px);
  --font-display:'Bricolage Grotesque',system-ui,sans-serif;
}
```

Y luego, en todas partes, `var(--gold)`. El cliente cambia de dorado → **cambias una
línea**.

Dos cosas de nivel que hay aquí:

**Los nombres son semánticos, no literales.** Se llama `--ink` (tinta), no `--negro`. Si
mañana el texto pasa a azul oscuro, el nombre sigue teniendo sentido. Un token llamado
`--azul` que ahora es verde es peor que no tener token.

**Hay variantes por función, no por capricho.** `--gold` y `--gold-deep` existen porque el
dorado normal **no pasa contraste AA** sobre blanco en texto pequeño. Por eso las cifras
grandes usan `--gold-deep` (`styles.css:207`, `310`). Eso es una decisión de
accesibilidad codificada en el sistema de diseño.

### 3.2 `clamp()`: tipografía y espacios fluidos

La función más útil del CSS moderno. Sintaxis:

```css
clamp(mínimo, valor-ideal, máximo)
```

Ejemplo real (`styles.css:152`):

```css
.hero-copy h1{font-size:clamp(2.3rem,5.6vw,4rem)}
```

Léelo así: *"nunca menor a 2.3rem, nunca mayor a 4rem, y entre medias que crezca al 5.6%
del ancho de la pantalla"*.

**Esto reemplaza tres media queries.** El titular se adapta de forma continua entre un
iPhone SE y un monitor 4K, sin saltos bruscos en los breakpoints.

Se usa igual para el espaciado (`styles.css:132`):

```css
.section{padding-block:clamp(56px,8vw,104px)}
```

Y para el padding lateral, guardado como token (`styles.css:44`):

```css
--pad:clamp(20px,5vw,40px);
```

Que después consume `.wrap` (`styles.css:67`):

```css
.wrap{max-width:var(--maxw);margin:0 auto;padding-inline:var(--pad)}
```

`.wrap` se usa **16 veces** en el HTML. Es la clase que garantiza que todo el sitio esté
alineado en la misma columna. Si un bloque se ve desalineado, lo primero que revisas es si
le falta `.wrap`.

> `padding-inline` en vez de `padding-left`/`padding-right` son las **propiedades
> lógicas**: se adaptan solas si el idioma se escribe de derecha a izquierda, y son la
> mitad de código.

### 3.3 Reúso de clases: el sistema de botones

`styles.css:77-96`. Este es **el ejemplo a copiar** en tus próximos proyectos.

Hay una clase base con todo lo estructural:

```css
.btn{display:inline-flex;align-items:center;justify-content:center;gap:.55em;
  font-weight:700;font-size:1.02rem;padding:.92em 1.5em;border-radius:999px;
  border:1px solid rgba(255,255,255,.5);
  backdrop-filter:blur(16px) saturate(180%);
  transition:transform .18s ease,box-shadow .18s ease,background .18s ease;
  white-space:nowrap;min-height:52px}
```

Y luego **modificadores** que solo cambian el color:

```css
.btn-primary{background:rgba(240,196,32,.62);color:var(--ink)}
.btn-wa     {background:rgba(37,211,102,.58);color:#04371f}
.btn-outline{background:rgba(255,255,255,.42);color:var(--ink)}
.btn-ghost  {background:rgba(240,196,32,.22);color:var(--gold-deep)}
```

Y **modificadores de tamaño/forma**, independientes del color:

```css
.btn-block{width:100%}
.btn-lg{font-size:1.1rem;padding:1.05em 1.8em;min-height:58px}
```

En el HTML los combinas: `class="btn btn-primary btn-lg"`, `class="btn btn-wa btn-block"`.

Con **8 reglas** cubres todas las combinaciones de la página. La alternativa amateur sería
`.boton-verde-grande`, `.boton-verde-pequeño`, `.boton-dorado-ancho`… y 30 reglas
duplicadas donde cambiar el radio del borde significa editar las 30.

Esta idea (base + modificadores) es la esencia de **BEM** y de los sistemas de diseño en
general. El mismo patrón está en `.section` / `.section.alt` (`styles.css:132-133`) y en
`.compare-col` / `.compare-col.bad` / `.compare-col.good` (`styles.css:264-276`).

**Cómo saber si una clase debe existir:** si el estilo se repite en 2+ lugares, es clase.
Si es único e irrepetible, va en la regla del componente.

### 3.4 Grid: layout con una línea

Casi todas las cuadrículas del sitio son `grid`:

```css
.steps  {display:grid;grid-template-columns:repeat(4,1fr);gap:1rem}   /* :248 */
.bento  {display:grid;grid-template-columns:repeat(3,1fr);gap:1.2rem} /* :279 */
.plans  {display:grid;grid-template-columns:repeat(3,1fr);gap:1.2rem} /* :344 */
.hero-grid{display:grid;grid-template-columns:1.05fr .95fr;gap:clamp(30px,5vw,64px)} /* :150 */
```

`1fr` = "una fracción del espacio disponible". `1.05fr .95fr` en el hero significa que la
columna del texto es apenas más ancha que la del chat.

Y `gap` reemplaza los márgenes entre hijos: no más `margin-right` en todos menos el último.

Truco elegante en la barra de métricas (`styles.css:203-205`):

```css
.metrics-grid{display:grid;grid-template-columns:repeat(4,1fr);gap:1px;
  background:var(--line);border:1px solid var(--line);
  border-radius:var(--radius);overflow:hidden}
.metric{background:#fff;padding:1.3rem 1rem;text-align:center}
```

El `gap:1px` con fondo gris y las celdas en blanco **dibuja las líneas divisorias sin
bordes**. Nunca tienes el problema del borde doble entre celdas.

### 3.5 Contadores CSS: numeración sin tocar el HTML

`styles.css:248-255`:

```css
.steps{...;counter-reset:step}
.step .num{counter-increment:step; ...}
.step .num::before{content:counter(step,decimal-leading-zero)}
```

Y en el HTML (`index.html:241-244`) los divs van **vacíos**:

```html
<div class="step"><div class="num"></div><h3>Diagnóstico y diseño</h3>...
```

El `01`, `02`, `03`, `04` lo genera el CSS. Si reordenas o insertas un paso, **la
numeración se recalcula sola**. `decimal-leading-zero` es lo que produce `01` en vez de `1`.

### 3.6 `@supports` y las media queries de preferencia

Los botones usan `backdrop-filter` (efecto vidrio esmerilado de iOS). No todos los
navegadores lo soportan, y ahí el botón se vería **casi transparente e ilegible**. La
solución (`styles.css:98-101`):

```css
@supports not ((backdrop-filter:blur(1px)) or (-webkit-backdrop-filter:blur(1px))){
  .btn-primary{background:var(--gold)}
  .btn-wa{background:var(--wa)}
}
```

*"Si no soportas el vidrio, usa color sólido."* Eso se llama **mejora progresiva**: la
página funciona en todos lados y se ve mejor donde se puede.

Además hay tres media queries de **preferencia del usuario**, no de tamaño:

```css
@media (prefers-reduced-transparency:reduce){...}   /* :102 — menos transparencia */
@media (prefers-reduced-motion:reduce){...}         /* :509 — menos animación */
```

La de movimiento (`styles.css:509-513`) apaga **todas** las animaciones del sitio:

```css
@media (prefers-reduced-motion:reduce){
  *,*::before,*::after{animation-duration:.001ms!important;
    animation-iteration-count:1!important;transition-duration:.001ms!important}
  .reveal{opacity:1;transform:none}
}
```

Esa última línea es la clave: si apagas la transición del `.reveal` **sin** forzar
`opacity:1`, todo el contenido animado se queda **invisible para siempre**. Es un error
que se ve mucho. Aquí está bien resuelto.

El JS lee la misma preferencia (`index.html:599`) para no animar los contadores ni el chat:

```js
var reduce = window.matchMedia("(prefers-reduced-motion: reduce)").matches;
```

CSS y JS respetando la misma señal del sistema. Eso es coherencia.

### 3.7 Otras funciones que vale la pena robarse de aquí

| Dónde | Qué hace |
|---|---|
| `styles.css:214-216` | `mask-image` con degradado: desvanece los bordes del carrusel de logos. |
| `styles.css:219` | `animation-play-state:paused` en `:hover`: pausa el marquee al pasar el mouse. |
| `styles.css:72-73` | Efecto marcador con `linear-gradient(180deg,transparent 58%,dorado 58%)` — el corte duro en el mismo % simula un resaltador. |
| `styles.css:63` | `text-wrap:balance`: reparte los titulares en líneas parejas, sin una palabra huérfana. |
| `styles.css:207` | `font-variant-numeric:tabular-nums`: dígitos de ancho fijo, para que los contadores no "salten". |
| `styles.css:154` | `max-width:34ch`: la unidad `ch` es el ancho de un carácter. 45–75 caracteres es el óptimo de lectura. |
| `styles.css:65` | `:focus-visible` con contorno dorado: navegación por teclado visible sin ensuciar el mouse. |
| `styles.css:503` | `env(safe-area-inset-bottom)`: evita que la barra fija quede tapada por la barra de gestos del iPhone. |

---

## 4. Móvil: dónde se rompe todo

Esta es la parte que más correcciones recibió. Los patrones a memorizar:

### 4.1 Dos breakpoints, no diez

```css
@media (max-width:960px){ ... }   /* styles.css:460 — tablet: 2 columnas */
@media (max-width:640px){ ... }   /* styles.css:475 — móvil: 1 columna  */
```

Con `clamp()` haciendo el trabajo fino de tipografía y espaciado, los breakpoints solo
tienen que ocuparse de **reordenar el layout**. Por eso son pocos y limpios.

En 960px las cuadrículas bajan a 2 columnas (`:467-469`); en 640px a 1 (`:493`).

### 4.2 Los seis errores clásicos y su antídoto

**1. Desbordamiento horizontal.** Un elemento posicionado fuera del viewport hace que toda
la página se pueda arrastrar de lado. Es *el* error más común y el más feo.

```css
body{overflow-x:hidden}                       /* styles.css:57  — contención */
.lead-badge{right:-14px}  →  @640: {right:8px} /* styles.css:192, 499 — la causa */
.chat{max-width:390px}    →  @640: {max-width:100%} /* styles.css:167, 498 */
```

Fíjate que `overflow-x:hidden` es la **red de seguridad**, no el arreglo. El arreglo real
es meter el `.lead-badge` dentro de la pantalla. Si solo pones `overflow-x:hidden`, el
elemento sigue fuera: simplemente ya no lo ves.

**2. Texto largo que rompe la caja.** Un correo como `contacto@agenciasincemarketing.com`
no tiene espacios, así que no puede cortarse y estira su contenedor. Es exactamente el
problema del `FIX` al final del archivo (`styles.css:555-560`):

```css
.contact-side .c-row > span:last-child{min-width:0;flex:1 1 auto}
.contact-side .c-row b{overflow-wrap:anywhere;word-break:break-word}
```

El `min-width:0` es el que casi nadie sabe: **por defecto un hijo de flex no se encoge por
debajo de su contenido**. Sin esa línea, `overflow-wrap` no alcanza a salvarte.

**3. Áreas tocables muy pequeñas.** El mínimo recomendado es **44×44px** (Apple) / 48×48px
(Google). Aquí se respeta:

```css
.btn{min-height:52px}              /* styles.css:82  */
.nav-toggle{width:46px;height:46px} /* styles.css:124 */
.mobile-cta .btn{min-height:50px}   /* styles.css:506 */
```

**4. Elementos fijos que se pisan.** En móvil hay tres cosas flotando: la barra inferior de
CTAs, el botón de WhatsApp y el botón de volver arriba. Si no los coordinas, se encaraman:

```css
.mobile-cta{bottom:0}                 /* styles.css:502 */
.wa-float  {bottom:76px}  /* @640 */  /* styles.css:500 — sube sobre la barra */
.to-top    {bottom:76px}  /* @640 */  /* styles.css:552 — misma altura, otro lado */
body       {padding-bottom:76px}      /* styles.css:507 — el footer no queda tapado */
```

Ese `padding-bottom` en el `body` es el que se olvida siempre. Sin él, la barra fija te
tapa la última línea del footer y nunca la puedes leer.

**5. La barra de gestos del iPhone.** `env(safe-area-inset-bottom)` (`styles.css:503`):

```css
padding:.7rem var(--pad) calc(.7rem + env(safe-area-inset-bottom));
```

Sin eso, los botones quedan debajo de la rayita negra del iPhone y son casi imposibles de
tocar. En el simulador de Chrome **no se nota**; en un iPhone real, sí.

**6. `scroll-margin-top` con header pegajoso.** El header es `position:sticky`
(`styles.css:109`). Al hacer clic en un ancla `#planes`, el navegador lleva la sección al
tope… **debajo** del header, que la tapa. El arreglo (`styles.css:131`):

```css
section[id]{scroll-margin-top:88px}
```

72px del header + 16px de aire.

### 4.3 Cómo probar de verdad

1. DevTools → modo dispositivo → **iPhone SE (375px)**. Si funciona ahí, funciona casi
   siempre. Es la pantalla más angosta que sigue siendo común.
2. Busca el scroll horizontal: **quítale** `overflow-x:hidden` al body temporalmente y
   arrastra de lado. Si se mueve, tienes un elemento desbordado que debes arreglar de raíz.
3. Pruébalo en un **celular real**. `safe-area-inset`, el teclado tapando el formulario y
   el `100vh` real de Safari no se reproducen en el simulador.
4. Con el teclado abierto, revisa que el campo activo se vea. `type="tel"` y `type="email"`
   ayudan porque abren teclados más bajos.

---

## 5. Deudas técnicas de este repo (aquí se aprende más)

Ser honesto con el propio código es parte del oficio. Lo que este proyecto **no** hace bien:

### 5.1 Parches al final del CSS en vez de editar la sección correspondiente

`styles.css` tiene tres bloques pegados al final:

```
516-525   /* AJUSTES: barra superior sin navegacion */
545-553   /* BOTON VOLVER ARRIBA */
555-560   /* FIX: correo largo en la tarjeta de contacto */
```

El primero agrega `.nav-actions{margin-left:auto}`. Pero `.nav-actions` **ya estaba
definido** en la línea 122, dentro del bloque Header:

```css
122:  .nav-actions{display:flex;align-items:center;gap:.6rem}
...
519:  .nav-actions{margin-left:auto}
```

Un mismo componente, partido en dos puntos del archivo separados por 400 líneas. No
rompe nada hoy, pero cuando abras la línea 122 vas a creer que ahí está todo su estilo, y
no es cierto. Ese `margin-left:auto` pertenece a la línea 122.

**Lo correcto:** el estilo del botón *volver arriba* va en la sección "Flotantes"
(`:439-449`), junto a `.wa-float`, con la que comparte comportamiento. El fix del correo va
en "Contacto" (`:386-416`). El `margin-left:auto` se corrige en la línea 122 y el bloque
`AJUSTES` desaparece.

Esta es probablemente la clase de desorden estructural que hubo que corregir en la entrega.
**Un archivo que crece por el final es un archivo que nadie está manteniendo.**

### 5.2 Los pasos deberían ser una lista ordenada

`index.html:241-244` usa `<div class="step">` para "los 4 pasos". Semánticamente eso es una
secuencia numerada:

```html
<ol class="steps">
  <li class="step"><h3>Diagnóstico y diseño</h3><p>…</p></li>
  …
</ol>
```

Con `<ol>` el lector de pantalla anuncia "lista de 4 elementos, elemento 1 de 4", que es
información real. Y el contador CSS sigue funcionando igual.

Lo mismo aplica a `.ind-grid` (`index.html:324-329`) y a `.bento`.

### 5.3 SVG repetidos

Hay **83 `<svg>` inline** en el HTML, y varios son el mismo dibujo repetido: el ícono de
WhatsApp aparece 4 veces con su `<path>` completo copiado tal cual. La solución estándar es
un sprite con `<symbol>`, definido una vez:

```html
<svg class="u-hidden"><symbol id="i-wa" viewBox="0 0 24 24"><path d="…"/></symbol></svg>
<!-- y en cada botón: -->
<svg><use href="#i-wa"/></svg>
```

Exactamente el mismo principio del reúso de clases, aplicado a gráficos.

### 5.4 Fuentes en base64: el trueque

`styles.css` pesa **191 KB**, de los cuales **145 KB son dos líneas** (`:9-10`) con las
fuentes en base64. El CSS real de la página son los otros **45 KB**.

La ventaja es real: cero peticiones externas y cero dependencia de Google Fonts (bueno para
privacidad y para GDPR).

El costo también: el CSS es **bloqueante de render**. El navegador no pinta **nada** hasta
terminar de descargar los 191 KB completos. Con `@font-face` apuntando a `.woff2`
separados, el CSS bajaría a 45 KB y la página pintaría mucho antes, con las fuentes
llegando después (`font-display:swap` muestra el texto de inmediato con una fuente del
sistema y la cambia al llegar la real). Encima, base64 infla el binario un ~33%: esos
145 KB son ~109 KB de `.woff2` de verdad.

No es un error — es una decisión con un trueque. Pero hay que **saber** que lo tomaste.
Si el cliente se queja de la velocidad en 4G, ese es el primer sitio donde mirar.

### 5.5 `.tst-card blockquote` declarado dos veces

`styles.css:325` y `:333` definen reglas para el mismo selector, separadas por 8 líneas. Es
inofensivo, pero es señal de que se editó a las carreras. Se fusionan en una.

---

## 6. Checklist antes de entregar

**Estructura**
- [ ] Cero `style="..."` en el HTML (`grep -c 'style="' index.html` → 0)
- [ ] Cero `<style>` embebidos
- [ ] Ningún bloque "FIX"/"AJUSTE" pegado al final del CSS
- [ ] Sin colores hardcodeados fuera de `:root`

**HTML / SEO**
- [ ] Un solo `<h1>`, con la palabra clave
- [ ] Jerarquía de encabezados sin saltos
- [ ] `<html lang>` correcto
- [ ] `<title>` ≤ 60 caracteres, `meta description` ≤ 155
- [ ] `canonical` apuntando a la URL definitiva
- [ ] OG image 1200×630, con `og:url` real
- [ ] JSON-LD validado en <https://search.google.com/test/rich-results>
- [ ] JSON-LD coincide con lo que se ve en pantalla
- [ ] Todos los `<img>` con `alt` descriptivo (vacío `alt=""` solo si es decorativa)
- [ ] Todos los `<input>` con su `<label for>`

**CSS / móvil**
- [ ] Sin scroll horizontal en 375px (probado **sin** `overflow-x:hidden`)
- [ ] Áreas tocables ≥ 44px
- [ ] Elementos fijos no se pisan entre sí ni tapan el footer
- [ ] `safe-area-inset` en barras fijas inferiores
- [ ] `scroll-margin-top` si hay header sticky
- [ ] Contraste AA (4.5:1 texto normal, 3:1 texto grande) — <https://webaim.org/resources/contrastchecker/>
- [ ] Con `prefers-reduced-motion` el contenido **sigue siendo visible**
- [ ] Navegable solo con `Tab`, con foco siempre visible

**Cierre**
- [ ] Lighthouse ≥ 90 en Performance, Accesibilidad, SEO y Buenas prácticas
- [ ] Probado en un celular físico, no solo en el simulador

---

## 7. Ejercicios sobre este mismo repo

Ordenados de menor a mayor dificultad. Hazlos en ramas aparte.

1. **Mueve los tres parches.** Reubica los bloques de `styles.css:516-525`, `545-553` y
   `555-560` a sus secciones correspondientes y elimina el `.nav-actions` duplicado.
   Verifica que nada cambie visualmente. *Aprendes: cascada y especificidad.*

2. **Convierte los pasos en `<ol>`.** Cambia `index.html:240-245` a lista ordenada sin
   romper el contador CSS. *Aprendes: semántica y `::before` con `content:counter()`.*

3. **Crea el sprite de SVG.** Empieza por el ícono de WhatsApp (4 copias) y sigue con los
   demás repetidos: `<symbol>` + `<use>`. Mide cuánto pesa menos el HTML. *Aprendes: reúso
   aplicado a gráficos.*

4. **Saca las fuentes a `.woff2`.** Extrae las dos líneas base64 a archivos externos con
   `@font-face` y `font-display:swap`. Compara el "First Contentful Paint" en Lighthouse
   antes y después. *Aprendes: rendimiento y trueques reales.*

5. **Agrega un cuarto tema de color.** Sin tocar nada fuera de `:root`, cambia toda la
   identidad de la página. Si tuviste que editar algo fuera de `:root`, encontraste un
   color hardcodeado — arréglalo. *Aprendes: si tu sistema de tokens es de verdad.*

---

## 8. Referencias

- [MDN — HTML semántico](https://developer.mozilla.org/es/docs/Glossary/Semantics)
- [schema.org](https://schema.org/) · [Test de resultados enriquecidos](https://search.google.com/test/rich-results)
- [MDN — `clamp()`](https://developer.mozilla.org/es/docs/Web/CSS/clamp)
- [MDN — Propiedades lógicas](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_logical_properties_and_values)
- [CSS Tricks — Guía completa de Grid](https://css-tricks.com/snippets/css/complete-guide-grid/)
- [WebAIM — Verificador de contraste](https://webaim.org/resources/contrastchecker/)
- [web.dev — Core Web Vitals](https://web.dev/vitals/)
- [BEM — Convención de nombres](https://getbem.com/naming/)
