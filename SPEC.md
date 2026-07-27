# Contrato del bundle de blog (`janium-blog-dist`)

Versión del contrato: **1**

Este documento define el formato que **Docta** produce y que **janium.com** consume.
Es la fuente de verdad. Ninguno de los dos lados asume nada que no esté aquí; si algo
falta, se resuelve por PR a este archivo antes de codificarlo.

---

## 1. Layout del repo

```
manifest.json                 # obligatorio; índice de entradas publicadas
posts/<slug>.<lang>.html      # un archivo por (entrada, idioma) presente
assets/<archivo>              # imágenes referenciadas por manifiesto y posts
```

- `<slug>` es **estable, único e independiente del idioma**. Es el segmento de URL de
  la entrada. Vive en el mismo `slug` para todos los idiomas; el idioma se distingue
  por el prefijo de ruta del sitio (`/blog/…`, `/en/blog/…`, `/pt/blog/…`), no por el slug.
- `<lang>` ∈ `es | en | pt` (códigos ISO 639-1). El contrato es extensible a más idiomas
  añadiéndolos al enum en una nueva versión.

---

## 2. `manifest.json`

Índice completo de las entradas **publicadas**. Docta lo regenera entero en cada emisión.

```json
{
  "contract": 1,
  "generated": "2026-07-26T00:00:00Z",
  "posts": [
    {
      "slug": "que-es-janium-collect",
      "date": "2026-07-24",
      "datetime": "2026-07-24T06:00:00Z",
      "tags": ["fundamentos", "producto"],
      "translations": {
        "es": {
          "title": "Qué es Janium Collect",
          "description": "Extracción de metadatos con IA, sin plantillas...",
          "cover": "assets/que-es-collect.es.webp",
          "html": "posts/que-es-janium-collect.es.html"
        },
        "en": {
          "title": "What Janium Collect is",
          "description": "AI metadata extraction, no templates...",
          "cover": "assets/que-es-collect.en.webp",
          "html": "posts/que-es-janium-collect.en.html"
        }
      }
    }
  ]
}
```

### Campos

| Campo | Tipo | Regla |
|---|---|---|
| `contract` | int | Debe ser `1`. Si janium.com lee un `contract` que no soporta, **falla el build** (no adivina). |
| `generated` | string ISO-8601 UTC | Sello de la emisión. Informativo. Lo estampa Docta. |
| `posts` | array | Puede estar vacío (`[]`). Orden **no** significativo: el consumidor ordena por `datetime` (con fallback a `date`). |
| `posts[].slug` | string | `[a-z0-9-]+`. Único en el manifiesto. Estable en el tiempo (cambiarlo rompe URLs y canonical). |
| `posts[].date` | string `YYYY-MM-DD` | Fecha **a mostrar** (la del autor). Etiqueta visible del post y del RSS. |
| `posts[].datetime` | string ISO-8601 UTC (sufijo `Z`) | **Opcional (aditivo en contrato 1).** Instante de publicación en GMT/UTC. **Ordena** índice, widget del home, RSS `pubDate` y `datePublished` del JSON-LD (más reciente primero). Permite conservar el orden al publicar desde husos distintos. Si falta (bundle previo), el consumidor cae a `date` a medianoche UTC. |
| `posts[].tags` | array de string | Puede estar vacío. Taxonomía libre; el consumidor puede mostrarlos o ignorarlos. |
| `posts[].translations` | objeto | **Al menos un idioma.** Clave = código de idioma; valor = objeto de traducción. |

### Objeto de traducción (`translations.<lang>`)

| Campo | Tipo | Regla |
|---|---|---|
| `title` | string | Título de la entrada. janium.com lo usa como `<h1>` y `<title>` — **no** debe repetirse dentro del HTML del cuerpo (ver §3). |
| `description` | string | 1–2 frases. Va en `<meta name="description">`, Open Graph y la tarjeta del índice. Texto plano, sin HTML. |
| `cover` | string \| null | Ruta repo-relativa bajo `assets/`, o `null` si la entrada no tiene portada. |
| `html` | string | Ruta repo-relativa al fragmento del cuerpo. Debe existir en `posts/`. |

### Reglas de presencia

- **Solo entradas publicadas** aparecen en el manifiesto. El filtro `estado:publicado`
  (del frontmatter en `docta-vault`) es responsabilidad de **Docta**; janium.com confía
  en que lo que está en el manifiesto es publicable.
- Una entrada aparece en el blog de un idioma **solo si ese idioma está en `translations`**.
  Una entrada solo en `es` → solo en `/blog/…`; no genera ruta `/en/blog/…`.
- Toda ruta en `cover` y `html` debe **existir** en el repo. Un puntero roto es un error
  de emisión: janium.com **falla el build** en vez de publicar una entrada rota.

---

## 3. Fragmentos HTML (`posts/<slug>.<lang>.html`)

Cada archivo es el **cuerpo del artículo, y nada más**:

- **Sin** `<!doctype>`, `<html>`, `<head>`, `<body>`, ni chrome del sitio (header, footer,
  nav). janium.com envuelve el fragmento con su propia plantilla — el mismo mecanismo que
  ya usa para las páginas legales (`prosePage`): mete `<title>`, `<meta>`, `hreflang`,
  header/footer y CSP.
- **Sin** el `<h1>` del título. El título lo pone janium.com desde `manifest.title` para
  no duplicar el H1. **El cuerpo empieza en `<h2>`.**
- Elementos permitidos: `h2`–`h4`, `p`, `ul`/`ol`/`li`, `blockquote`, `pre`/`code`,
  `table`/`thead`/`tbody`/`tr`/`th`/`td`, `a`, `strong`/`em`, `figure`/`figcaption`, `img`.
- **Sin recursos externos.** La CSP de janium.com es estricta: nada de `<script>`,
  `<style>`, fuentes, iframes ni imágenes remotas. Todo recurso es local (ver §4).
- **Enlaces**: internos como rutas absolutas del sitio (`/blog/otro-slug/`); externos con
  `https://`. janium.com puede añadir `rel="noopener"` a los externos.
- Debe ser HTML **bien formado** (elementos cerrados, atributos entrecomillados). Es un
  fragmento, no un documento: se inyecta tal cual dentro de un `<article>`.

---

## 4. Assets (`assets/`)

- Imágenes referenciadas por `cover` y por los `<img>` de los fragmentos.
- Formatos: `webp` (preferido), `jpg`, `png`. Optimizadas por Docta (tamaño y peso web).
- **Referencia desde el HTML**: el `src` de cada `<img>` es una ruta **repo-relativa**
  que empieza por `assets/` (p. ej. `<img src="assets/f1-diagrama.webp" alt="...">`).
  janium.com copia `assets/` → `dist` en `/assets/blog/` y reescribe `assets/` → `/assets/blog/`
  al hornear. Docta **no** debe emitir rutas absolutas del sitio ni URLs remotas.
- Todo `<img>` debe traer `alt`. La accesibilidad del texto alternativo es de Docta
  (conoce el idioma y el contexto).
- **git-lfs** (opcional, futuro): si `assets/` crece, migrar `*.webp`/`*.jpg`/`*.png` a LFS
  con un `.gitattributes`. Hoy no está activado; activarlo es un cambio coordinado por PR
  (ambos lados deben tener `git lfs` en su checkout, incluido el runner de Jenkins).

---

## 5. Responsabilidades

**Docta (productor)**
1. Filtra entradas `estado:publicado`.
2. Renderiza el markdown a fragmento HTML por idioma (§3).
3. Resuelve traducciones (es/en/pt) e imágenes; optimiza y coloca assets (§4).
4. Regenera `manifest.json` completo (§2).
5. Empuja un commit a este repo (rama `main`).

**janium.com (consumidor), en `build.mjs`**
1. Hace checkout/submódulo de este repo en build-time.
2. Valida `contract` y la integridad de punteros (falla el build si algo no cuadra).
3. Por cada entrada y cada idioma presente genera la página envolviendo el fragmento con
   el chrome del sitio, `<title>`/`<meta>`/Open Graph desde el manifiesto, y `hreflang`
   entre los idiomas disponibles de esa entrada.
4. Genera el índice `/blog` (y `/en/blog`, `/pt/blog`) ordenado por `date` desc.
5. Genera el widget de "últimas 3–5 entradas" del home.
6. Genera `/blog/feed.xml` (RSS) y añade las entradas al sitemap.
7. Copia `assets/` → `/assets/blog/` y reescribe los `src`.
8. Añade el enlace "Blog" a nav y footer.

---

## 6. SEO y sindicación (nota de diseño, no normativa)

El HTML se hornea en janium.com, así que la **autoridad de dominio se acumula en
janium.com** (no en el transporte ni en Medium). Si una entrada se sindica a Medium, se
publica allí con `rel="canonical"` apuntando a la URL de janium.com, para conservar el
SEO en el origen sin renunciar a la exposición de Medium. Esto no impone nada al bundle:
el `canonical` lo construye janium.com desde el `slug` y el idioma.

---

## 7. Versionado del contrato

- Cambios compatibles (añadir un campo opcional, un idioma): se documentan aquí sin subir
  `contract`, y el consumidor los ignora si no los conoce.
- Cambios incompatibles (renombrar/quitar un campo, cambiar semántica): **suben `contract`**
  y se coordinan por PR. El consumidor que no soporte el nuevo número falla el build en vez
  de producir salida silenciosamente incorrecta.
