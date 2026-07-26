# assets/

Portadas e imágenes embebidas de las entradas. Las coloca **Docta** al emitir el bundle.

- Formatos: `webp` (preferido), `jpg`, `png`; optimizadas para web.
- Se referencian desde `manifest.json` (`cover`) y desde los `<img>` de `posts/*.html`
  con ruta **repo-relativa** que empieza por `assets/` (ver `SPEC.md` §4).
- janium.com las copia a `/assets/blog/` en build-time y reescribe los `src`.

Si esta carpeta crece, considerar **git-lfs** para los binarios (coordinar por PR).
