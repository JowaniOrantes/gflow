# Ideas: fábrica de reels y shorts

**Estado: lluvia de ideas, todavía no es un plan.** Este documento reúne lo que se ha hablado, lo que está verificado y lo que falta decidir. No define una arquitectura.

## 1. Idea

Una fábrica de vídeos cortos verticales (9:16) para publicar a diario en YouTube Shorts, Facebook, Instagram (prioridad), y TikTok. Un único master limpio por pieza, distribuido a todas las redes.

Flujo imaginado:

1. Guion en hoja de tomas.
2. Imágenes: miniaturas y fotogramas inicial y final de cada escena.
3. Escenas cortas con Flow, dentro de su límite de duración y de reescalado.
4. Voz en off y música.
5. Unión de las escenas en un solo vídeo y publicación.

## 2. Recursos disponibles

| Recurso | Qué se sabe |
|---|---|
| gflow-cli (v0.85.0) sobre Google Flow, plan **Pro** | Veo 3.1 (`veo-lite`, `veo-fast`, `veo-quality`) y `omni-flash`; Nano Banana 2 y Pro. Alfa, no oficial |
| Host Ubuntu Server 24.04, RTX 8 GB, i5 de 12.ª gen | Chrome bajo Xvfb; login por VNC solo la primera vez |
| OpenClaw (agente de IA) con Vertex AI | Dice el usuario que tiene `gemini-4-argon`. Vertex no lista modelos: se prueba con llamadas |
| Vertex AI: voz y música | Por probar (ver sección 5) |

## 3. Qué está verificado en gflow-cli (lectura de código y docs, nada ejecutado)

- Clips de 4, 6 u 8 s (10 s en omni-flash). 9:16 y 16:9.
- Las imágenes, el montaje de escenas en servidor y el escalado a 2K no gastan créditos. El vídeo de Veo sí. Pedir 1–4 variantes multiplica el coste.
- Veo **siempre genera audio**: no hay modo silencioso. Se quita al montar con ffmpeg (`-an`).
- Con `i2v` solo cuenta lo que aparece en el fotograma inicial: **no lleva la identidad de un personaje**.
- Solo se admite una referencia con cara por generación (con dos, error 400).
- `extend` mantiene movimiento y audio, pero cada tramo trae unos 7 s reales aunque se cobra como 8, y deja un segundo congelado antes de cada corte. Se arregla recortando.
- `chain` no mantiene el audio, y sus clips caen en proyectos distintos: se unen con ffmpeg.
- `scene create -o` une en servidor y sin créditos, **solo dentro de un mismo proyecto**.
- Con ffmpeg: usar el filtro concat, no el demuxer; mezclar fps provoca desfase del audio.
- No hay documentado ningún límite de duración total de una escena, ni qué aspecto o resolución devuelve la unión.

## 4. Decisiones abiertas

| Decisión | Recomendación inicial | Por qué |
|---|---|---|
| **Tono** | Uno solo y repetible (por ejemplo, curiosidades narradas) | Una plantilla fija reduce el coste por pieza y facilita medir |
| **Guion** | Hoja de tomas: toma, acción, frase, duración. Escrita por un LLM | Permite comprobar que cada frase cabe en un clip de 8 s |
| **Narrador** | **Sin rostro**: voz en off añadida después, clips sin hablante visible | Elimina la sincronización de labios y el problema de la voz entre clips |
| **Personaje que habla** | Experimento aparte, en una fase 2 | La voz de un personaje de Flow no está comprobada al generar vídeo; el español figura como "no evaluado" |
| **Formato** | 9:16, 20–40 s, 3–5 clips | Mínimo que cuenta una historia sin multiplicar fallos |
| **Plataformas** | Todas, con un master limpio y sin marca de agua de otras redes | Es el mismo archivo; el coste de distribuir es bajo |
| **Cadencia** | A diario, **pero primero medir** cuántos créditos y cuántos clips repetidos cuesta cada pieza | Hoy no hay ese dato |
| **Reescalado** | 1080p nativo o local; 4K solo en casos puntuales | Con plan Pro no hay 4K de imagen |

## 5. Modelos de Vertex AI a probar

Los datos de precio vienen de documentación de Google (lo oficial) y de webs de terceros (sin confirmar). **Verifica contra la página de precios de Google Cloud antes de decidir.** Vertex no lista modelos en este entorno, así que cada uno se comprueba con una llamada.

| Necesidad | Candidato | Qué se sabe | Cómo probarlo |
|---|---|---|---|
| Voz en off | **Chirp 3 HD** | 30 voces con nombre; la documentación lista es-ES y es-US; GA | Una frase de 20 s en español; escuchar naturalidad y ritmo |
| Voz en off con control de estilo | Gemini TTS (`gemini-2.5-flash-tts`, `gemini-3.1-flash-tts-preview`) | El 3.1 está en preview; **no confirmado** que soporte español | Misma frase, con instrucciones de tono |
| Música de fondo | **Lyria 2** (`lyria-002`) | Oficial: 0,06 $ por 30 s, WAV 48 kHz, solo instrumental | Una pista de 30 s; probar que no compite con la voz |
| Música (más nuevo) | Lyria 3 Clip/Pro | Precio de terceros: 0,04–0,08 $ por canción, sin confirmar; no verifiqué si incluye voz | Una llamada de prueba |
| Guion | `gemini-4-argon` u otro | El usuario dice que OpenClaw lo tiene; **no lo he podido verificar** | Pedir una hoja de tomas con el formato de la sección 4 |
| Vídeo con Veo en Vertex | Veo 3.1 (API oficial) | Cobra por segundo; precios de terceros se contradicen (Fast ≈ 0,10–0,30 $/s) | No por ahora; ver abajo |

**Reparto propuesto de recursos:**

- **Créditos de Flow Pro:** imágenes y vídeo (donde el modelo y la herramienta ya se conocen).
- **Vertex AI:** voz y música, que son baratos por llamada y no dependen de automatizar una web.
- **Veo en Vertex:** reservarlo como plan B si gflow se rompe o la cuenta de Flow se bloquea. Es una vía oficial, pero se paga por segundo.
- **Gemini:** guion y hoja de tomas.

Con esto se aprovechan los créditos de Flow solo en lo que Flow hace mejor, y las llamadas a Vertex para lo que cuesta céntimos.

## 6. Pros, contras y oportunidades

**Pros**
- Imágenes, montaje en servidor y 2K no gastan créditos: se puede iterar mucho antes de gastar en vídeo.
- 9:16 es nativo; no hay que recortar desde horizontal.
- Los fallos conocidos de gflow ya están documentados con su arreglo.
- Un narrador sin rostro evita los tres puntos más inciertos: voz, español y labios.

**Contras**
- Un reel de 30 s son unos 4–5 clips; cada variante extra multiplica el coste.
- El audio de Veo se descarta (se paga por algo que se quita).
- Todo depende de una herramienta alfa que automatiza una web, con riesgo de rotura y de cuenta.
- Publicar a diario en 4 redes multiplica la producción.

**Oportunidades**
- Una biblioteca de fondos reutilizables como "set" de marca.
- Subtítulos tomados del guion.
- Mezcla de audio y reescalado en el host, para depender menos de Flow.
- Una sola pieza adaptada a varias redes con títulos y descripciones distintos.

## 7. Riesgos

- **Cuenta de Google:** gflow no es oficial; el uso masivo automatizado puede chocar con los términos. Usar una cuenta aparte (ver `docs/ACCOUNT_SAFETY.md` de gflow-cli).
- **Plataformas:** YouTube, Meta y TikTok piden declarar el contenido sintético realista. Revisar la política vigente de cada una.
- **Publicación por API:** Instagram permite unos 100 posts por API cada 24 h (la documentación de Meta también cita 50: leer el límite real del endpoint). No he verificado las condiciones de TikTok ni de YouTube para publicar por API; comprobarlas antes de automatizar.
- **Monetización:** el contenido repetitivo o masivo puede no ser monetizable. Hace falta un ángulo editorial real.
- **Derechos:** no imitar voces ni personajes reales.
- **Dependencia:** guardar prompts y recursos fuera de Flow.
- **Agente:** OpenClaw no debe poder lanzar generación de vídeo sin un presupuesto ni sin límite de variantes.

## 8. Siguiente paso mínimo

**Un reel de prueba**, sin personaje que hable:

1. Hoja de tomas de 3 clips de 8 s (≈24 s), en 9:16, con un tono definido.
2. Fotogramas con Nano Banana 2; clips con `i2v` y `veo-lite`, una variante por clip.
3. Voz en off con Chirp 3 HD y música con Lyria 2.
4. Montaje con ffmpeg: audio de Veo fuera, voz sobre música más baja, subtítulos.
5. Publicar el mismo master en las redes elegidas.

Qué medir:

| Pregunta | Cómo se responde |
|---|---|
| ¿Cuántos créditos de Flow gasta un reel? | `gflow credits user` antes y después |
| ¿Cuántos clips hubo que repetir? | Contarlos |
| ¿La voz de Chirp 3 HD suena natural en español? | Escucharla entera |
| ¿Encaja la música de Lyria? | Revisar que no tape la voz |
| ¿Funciona el flujo de principio a fin sin intervención? | Anotar cada punto donde hubo que intervenir |
| ¿Cuánto tiempo humano costó? | Cronometrar |
| ¿Responden los modelos de Vertex? | Una llamada a cada uno; registrar errores y regiones |

Si el español de Chirp falla, se prueba Gemini TTS; si falla el flujo de gflow, se evalúa Veo en Vertex.

## 9. No verificado

- Que `gemini-4-argon` exista y sirva para guion (lo afirma el usuario).
- Soporte de español en Gemini TTS.
- Precios de Veo y de Lyria 3 en Vertex.
- Si Lyria 3 genera voz.
- Condiciones de publicación por API de TikTok y YouTube.
- El coste en créditos de cada modelo de Flow, y el de `video upscale`.
- Si `video upscale` funciona en vertical.
- La integración con OpenClaw.
- Que el login de Google funcione dentro de Xvfb con VNC.
