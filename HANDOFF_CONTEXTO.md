# Traspaso de contexto entre sesiones

Para retomar este trabajo desde otra sesión (por ejemplo, la local de OpenClaw), lee los documentos de esta rama en este orden. Una sesión local no puede ver el historial de esta sesión en la nube: lo que importa está en el repo.

**Repo:** `JowaniOrantes/gflow`, rama `claude/flowkit-gflow-cli-comparison-k30u0t`

```bash
git clone https://github.com/JowaniOrantes/gflow
cd gflow && git checkout claude/flowkit-gflow-cli-comparison-k30u0t
```

## Orden de lectura

1. `IDEAS_FABRICA_REELS.md`: el estado actual (lluvia de ideas, no hay plan), decisiones abiertas, qué está verificado y qué no, y el experimento mínimo.
2. `WORKFLOW_HIBRIDO_GFLOW_UPSCALE.md`: Flow + reescalado local, hardware (RTX 8 GB), plan Pro de Flow y consejos de ahorro de créditos.
3. `GUIA_VIDEOS_FACELESS_GFLOW.md`: prompts de guion, título, miniatura y SEO adaptados a gflow.

## Resumen en diez líneas

- Proyecto: fábrica de reels y shorts (9:16) para YouTube, Facebook, Instagram (prioridad) y TikTok, publicación diaria. Todavía es lluvia de ideas.
- Herramienta de generación: `gflow-cli` v0.85.0 sobre Google Flow (plan Pro). Es alfa y no oficial.
- Veo siempre genera audio; se quita al montar. Los clips duran 4, 6 u 8 s.
- `i2v` no conserva la identidad de un personaje. Por eso el narrador empieza **sin rostro**, con voz en off añadida después.
- Vertex AI (OpenClaw): voz con Chirp 3 HD, música con Lyria 2, guion con `gemini-4-argon` (dato del usuario, sin verificar). Vertex no lista modelos: se prueban con llamadas.
- Host: Ubuntu Server 24.04, RTX 8 GB, i5 de 12.ª gen. Chrome bajo Xvfb; login por VNC solo la primera vez.
- Reescalado local (ESRGAN/Remacri en ComfyUI) solo si hace falta; nunca difusión fotograma a fotograma.
- Riesgos: cuenta de Google, etiquetado de contenido con IA, monetización, dependencia de una herramienta alfa.
- Siguiente paso: un reel de prueba de ~24 s y medir créditos, clips repetidos, tiempo humano y calidad de la voz en español.
- OpenClaw no debe poder lanzar vídeo sin presupuesto ni límite de variantes.

## Qué no está verificado

Ver la sección 9 de `IDEAS_FABRICA_REELS.md`.
