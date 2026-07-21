# Landing Page — Agente de IA de Ventas · Since Marketing Digital

Landing page persuasiva de un solo servicio (Agente de IA de Ventas 24/7) para
**Since Marketing Digital**. Optimizada para **SEO, persuasión y legibilidad**, con
fondo blanco, prueba social, multimedia funcional, CTAs abundantes y diseño 100% responsive.

Basada en el contenido de <https://agenciasincemarketing.com/servicios/agente-ia-ventas/>.

## Características

- **Un solo archivo autocontenido** (`index.html`): CSS, JS, gráficos SVG y **fuentes embebidas
  en base64** (Bricolage Grotesque + Hanken Grotesk). Cero peticiones externas → carga rápida,
  privacidad y máxima compatibilidad, incluido **celular**.
- **CTAs constantes**: header, hero (WhatsApp + formulario), tras cada sección, planes, contacto
  final, botón flotante de WhatsApp y barra fija inferior en móvil.
- **Prueba social**: muro de logos de clientes + testimonios (marcados como *ejemplo*) + métricas.
- **Multimedia funcional**: chat animado del agente, gráfico de conversión, diagrama de 4 pasos,
  comparativa "sin/con IA", contadores animados y gráfico de crecimiento de leads.
- **SEO**: HTML semántico, metas, Open Graph/Twitter, datos estructurados JSON-LD
  (Organization, Service, FAQPage, BreadcrumbList), `sitemap.xml`, `robots.txt`.

## Estructura

```
index.html              Landing page (todo incluido)
assets/img/favicon.svg   Favicon
assets/img/og-image.png  Imagen para compartir en redes (1200×630)
robots.txt · sitemap.xml · site.webmanifest
netlify.toml             Configuración de despliegue en Netlify
```

## Cómo desplegarla

Es un sitio **estático**. Cualquiera de estas opciones funciona:

- **Netlify** (recomendado, incluye captura del formulario):
  arrastra la carpeta a <https://app.netlify.com/drop> o conecta este repo.
  El formulario ya está listo con **Netlify Forms** (`data-netlify="true"`); los envíos
  aparecen en el panel de Netlify → *Forms*.
- **Vercel / GitHub Pages / Cloudflare Pages**: publica la raíz del repositorio.
- **Local**: `python3 -m http.server` y abre <http://localhost:8000>.

## ✅ Antes de publicar: reemplaza estos placeholders

1. **Testimonios** — Sustituye los 3 testimonios de *ejemplo* (marcados con la etiqueta
   "Ejemplo") por reales: frase, nombre, empresa y foto. Ideal: agregar un video-testimonio.
2. **Logos de clientes** — Los chips del muro usan el nombre del cliente. Reemplázalos por los
   logos reales (SVG/PNG) si los tienes.
3. **Formulario** — Si no usas Netlify, conecta el `<form>` a tu herramienta (Formspree, tu CRM
   Go High Level, etc.). Sin backend, el formulario muestra confirmación y ofrece WhatsApp.
4. **Canonical / URLs / OG** — En `index.html` (`<link rel="canonical">`, `og:url`, `og:image`)
   y en `sitemap.xml` / `robots.txt` está la URL del servicio de Since. Cámbiala si la publicas
   en otro dominio (ej. Netlify).
5. **Precios y planes** — Confirma que Starter/Growth/Pro, la implementación ($500.000 COP) y la
   promoción (14 días gratis, 50% dto.) siguen vigentes.
6. **Imagen OG** — Puedes reemplazar `assets/img/og-image.png` por una con foto real del equipo/producto.

## Contacto configurado

- WhatsApp: **+57 316 052 0942** (`wa.me/573160520942`, con mensajes prellenados por sección)
- Correo: **contacto@agenciasincemarketing.com**
- Redes: Facebook, Instagram, TikTok, LinkedIn (enlaces en el footer)
