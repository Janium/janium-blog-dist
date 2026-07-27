# janium-blog-dist

Contenido del blog de **[Janium](https://www.janium.com)**, renderizado y listo para
publicar. Es el puente entre **Docta** —el sistema que redacta y renderiza los
artículos— y el sitio **janium.com**, que lo hornea en tiempo de build.

- **Blog en vivo:** https://www.janium.com/blog/
- Cada entrada: `posts/<slug>.<lang>.html` (el cuerpo del artículo) + su portada en
  `assets/`, indexadas en `manifest.json`. Idiomas: `es`, `en`, `pt`.
- Contrato del formato: **[`SPEC.md`](SPEC.md)** (`contract 1`).

## Sobre Janium Collect

[Janium Collect](https://www.janium.com/janiumcollect/) cataloga acervos con IA: lee
documentos, imágenes, audio y video, y produce registros en formatos estándar —Dublin
Core, MARC21/UNIMARC, ISAD-G, CDWA (Getty)—. **Identifica y enriquece con procedencia**:
lo verificable se contrasta contra autoridades y se atribuye; cada registro se evalúa y
se marca para revisión. Procesa en la infraestructura de la institución cuando el
material no puede salir. Bibliotecas, archivos y museos en Latinoamérica, España y
Portugal.

## Cómo se produce

Docta renderiza los artículos **publicados** desde su vault (nunca borradores) y empuja
aquí `manifest.json` + `posts/` + `assets/`. janium.com consume este repositorio en
build-time: valida el contrato, genera las páginas del blog y copia las imágenes. El
detalle del formato y las responsabilidades de cada lado están en `SPEC.md`.
