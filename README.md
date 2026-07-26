# janium-blog-dist

Bundle de blog para **janium.com**. Es el punto de encuentro entre dos sistemas:

- **Docta** (`Janium/docta`) *renderiza* los artículos (desde el markdown fuente de
  `Janium/docta-vault`), resuelve traducciones e imágenes, y **empuja** aquí el
  resultado como HTML + manifiesto + assets.
- **janium.com** (`Janium/janium-website`) *consume* este repo en build-time: su
  `build.mjs` lee el manifiesto y hornea `/blog`, las páginas de cada entrada, el
  widget de "últimas entradas" del home, `feed.xml` y el sitemap.

Ninguno de los dos lados improvisa el formato: ambos respetan **[`SPEC.md`](SPEC.md)**,
el contrato de este repo. Cambios al contrato se acuerdan por PR aquí antes de
implementarse en cualquiera de los dos lados.

## Por qué git como transporte

- **Versionado**: cada emisión de Docta es un commit. Un build de janium.com puede
  fijarse a un commit del bundle (submódulo) → builds reproducibles y rollback.
- **Diff por publicación**: se ve exactamente qué cambió entre dos estados del blog.
- **Encaja en el flujo**: Jenkins ya hace checkout de git; no hace falta un canal nuevo.

Caveat: las imágenes son binarios. Al vivir en un repo *dedicado* (no el de código)
el bloat queda contenido. Si `assets/` crece mucho, migrar esos tipos a **git-lfs**
(ver `SPEC.md` §Assets); hoy no es necesario.

## Estructura

```
manifest.json                 # índice de todas las entradas publicadas (lo emite Docta)
posts/<slug>.<lang>.html      # cuerpo del artículo renderizado, un archivo por idioma
assets/<archivo>              # portadas e imágenes embebidas (webp/jpg/png)
SPEC.md                       # el contrato (fuente de verdad del formato)
manifest.sample.json          # ejemplo del manifiesto para desarrollar contra él
```

## Quién escribe qué

| | Docta (productor) | janium.com (consumidor) |
|---|---|---|
| `manifest.json` | **escribe** | lee |
| `posts/*.html` | **escribe** (cuerpo del artículo) | lee, envuelve con el chrome del sitio |
| `assets/*` | **escribe** | copia a `/assets/blog/` |
| `SPEC.md` | acuerda por PR | acuerda por PR |

Este repo **no** contiene el markdown fuente (eso vive en `Janium/docta-vault`) ni el
código del sitio (`Janium/janium-website`). Solo el artefacto renderizado y su contrato.
