# Workflow híbrido: generación barata en Flow (gflow) + reescalado local

Objetivo: gastar el mínimo de créditos de Google Flow priorizando **calidad de generación** (movimiento, coherencia, composición) y no resolución, y subir la resolución después con herramientas propias.

Fuentes revisadas (lectura de código y documentación, nada ejecutado):

- `ffroliva/gflow-cli` v0.85.0 (comandos, modelos, docs de autenticación y contenedor)
- `Tech-Multiverse/4k-ai-video-workflows` (4 workflows de ComfyUI)
- `ssitu/ComfyUI_UltimateSDUpscale` (nodos de ComfyUI)

---

## 1. Aclaraciones antes de diseñar

**Los nombres de modelo no coinciden con lo que hay en gflow.** Lo que `gflow` expone es:

| Tipo | Modelos que acepta `--model` |
|---|---|
| Vídeo | `veo-lite`, `veo-fast`, `veo-quality`, `veo-lite-lp` (son **Veo 3.1**) y `omni-flash` |
| Imagen | Nano Banana 2 (por defecto; en código `NARWHAL`, alias `nano2`) y Nano Banana Pro (`nanopro`) |

No he encontrado "Veo 1.1" ni "Nano Banana 2.1". Asumo que te refieres a **Veo 3.1** y **Nano Banana 2**. Si hablas de otra cosa, dímelo. `gflow models` imprime el catálogo real de tu versión.

**Los dos repos de reescalado hacen cosas distintas, y ninguno es para vídeo de Veo tal cual:**

- `4k-ai-video-workflows` genera con **WAN 2.1 en local** (no usa Veo ni Flow). Lo aprovechable son sus dos workflows de **reescalado**:
  - *MP4 → 4K*: `VHS_LoadVideo` → `UpscaleModelLoader` (`4x_foolhardy_Remacri.pth`) → `ImageUpscaleWithModel` → `VHS_VideoCombine` (h264, CRF 10, 24 fps).
  - *WEBP → 4K*: lo mismo más interpolación de fotogramas con `RIFE VFI` (`rife47`, ×2) y limpieza de VRAM entre pasos.
- `ComfyUI_UltimateSDUpscale` reescala **imágenes** con un paso de difusión por teselas. Sirve para fotogramas clave y miniaturas. **No lo uses fotograma a fotograma en vídeo**: es muy lento y la difusión por fotograma provoca parpadeo entre fotogramas.

**La arquitectura de gflow condiciona dónde puede correr.** Según `docs/AUTHENTICATION.md`, gflow lanza un Chrome real con ventana (no headless) con Playwright, porque reCAPTCHA Enterprise rechaza el modo headless. En un servidor sin pantalla hace falta un display virtual (Xvfb). El `docker/README.md` del proyecto dice que el inicio de sesión de Google dentro de un contenedor **no se ha probado**.

---

## 2. Qué cuesta créditos y qué no

Según el README y la documentación de gflow:

| Operación | ¿Gasta créditos? |
|---|---|
| Imágenes (`gflow image ...`), con cuotas diarias aparte | No (límite diario propio) |
| `gflow image upscale --scale 2k` | No |
| `gflow image upscale --scale 4k` | No, pero **solo con Flow Ultra** (sin Ultra falla con código 22) |
| Composición de escenas / `gflow scene` (concatenado en servidor) | No |
| `gflow credits user` | No (solo lectura) |
| Generación de **vídeo** (Veo) | **Sí**; la cantidad `1-4` por orden multiplica el coste |
| `gflow video upscale --scale 1080p` | **No lo he podido verificar** en la documentación |

Consecuencia: el ahorro está en **mover trabajo a imágenes y a cómputo local** y en generar **pocos clips y bien elegidos**.

---

## 3. Arquitectura propuesta

```
               HOST Ubuntu Server 24.04 (sin pantalla)
 ┌───────────────────────────────────────────────────────────────┐
 │  Xvfb :99  ──►  Chrome real (gflow, perfil persistente)       │
 │  x11vnc (solo localhost) ◄── túnel SSH ── tu PC (solo login)  │
 │                                                               │
 │  gflow (CLI / `gflow mcp run`)  ◄──  OpenClaw (orquestador?)  │
 │         │                                                     │
 │         ▼  imágenes 1K, clips Veo                             │
 │  /data/proyecto/{img,clips,final}                             │
 │         │                                                     │
 │         ▼  fase local (si hay GPU)                            │
 │  ComfyUI (127.0.0.1)  ·  Real-ESRGAN/Remacri  ·  RIFE  ·  ffmpeg│
 └───────────────────────────────────────────────────────────────┘
```

### Autenticación por VNC (solo la primera vez)

`gflow auth login --browser chrome` necesita ver el navegador para resolver 2FA o CAPTCHA.

1. Instalar Google Chrome estable (el `.deb`, instalado en `/opt/google/chrome/chrome`). gflow exige **Chrome real, no Chromium de la distribución**. Instalar también `xvfb` y `xauth` (el proyecto avisa de que sin `xauth` falla).
2. Arrancar Xvfb y un servidor VNC **ligado a 127.0.0.1**, sobre ese display:
   ```bash
   Xvfb :99 -screen 0 1440x900x24 &
   x11vnc -display :99 -localhost -rfbauth ~/.vnc/passwd -forever &
   ```
3. Desde tu PC, túnel SSH (no abras el puerto 5900 a la red) y cliente VNC contra `localhost`:
   ```bash
   ssh -L 5900:127.0.0.1:5900 usuario@host
   ```
4. En el host: `DISPLAY=:99 gflow auth login --browser chrome`, con `GFLOW_CLI_HOME` apuntando a un volumen persistente.
5. Parar VNC. El resto de comandos corren sobre Xvfb sin que mires nada. Comprueba con `gflow auth status`.

Riesgos a vigilar: Google puede pedir verificaciones extra si el inicio de sesión viene de una IP de servidor distinta a la habitual. Haz el login desde el propio host y no muevas el perfil entre máquinas. Lee `docs/ACCOUNT_SAFETY.md` y `DISCLAIMER.md`: la automatización está sujeta a los términos de Google.

### Sobre OpenClaw

No conozco OpenClaw con suficiente detalle para afirmar cómo se integra. Lo que sí puedo decir de gflow:

- gflow lanza **su propio** Chrome con **su propio** perfil (`GFLOW_CLI_HOME/profile_<nombre>`). No puede reutilizar el navegador de otra herramienta, y compartirlo no es una buena idea (el login de Google va ligado al perfil).
- Si OpenClaw habla MCP, gflow trae `gflow mcp run`; y el repo documenta un modo `serve` con `GFLOW_CLI_DAEMON_TOKEN`. Es la vía natural para que OpenClaw ordene tareas sin tocar el navegador.
- Si solo quieres que OpenClaw lance comandos de shell, también vale.

---

## 4. Pipeline por etapas

### Etapa 0: Plan de planos (sin coste)
Usa el Prompt H de `GUIA_VIDEOS_FACELESS_GFLOW.md` para sacar la tabla de planos con duración (4–8 s por clip) y qué planos **merecen vídeo** y cuáles bastan como imagen con movimiento simulado en edición (zoom, paneo).

### Etapa 1: Fotogramas clave con Nano Banana 2 (créditos: ninguno)
- Genera todos los planos como imagen a 1K y elige los buenos. Esta es la etapa de iterar.
- Para personajes recurrentes: `gflow character create` (por defecto Nano Banana 2) y referencia el personaje en cada generación.
- Para los fotogramas que vayan a usarse como **imagen final** (miniatura, planos estáticos): `gflow image upscale <mediaId> --scale 2k` (gratis). 4K solo si tienes Ultra.

### Etapa 2: Vídeo con Veo 3.1 (aquí se gastan créditos)
Reglas de ahorro:

1. **Imagen a vídeo, no texto a vídeo.** `gflow video i2v` parte de un fotograma ya aprobado, así que acierta más veces. gflow usa `veo-lite` por defecto en i2v (el más barato).
2. **Una variante por orden.** La opción de cantidad (1–4) multiplica el coste. Déjala en 1.
3. **Clips cortos.** 4–6 s en vez de 8 s, si el plano lo permite.
4. **Escala de calidad:** `veo-lite` para la mayoría; `veo-fast` si lite falla; `veo-quality` solo para planos héroe. Ten en cuenta que `veo-quality` no acepta R2V (referencias).
5. **Antes de gastar:** `gflow credits user` y, para cadenas largas, `gflow video chain ... --dry-run` (imprime el plan sin enviar nada).
6. **Un test por estilo** antes de lanzar el lote entero.
7. **No regeneres a ciegas:** si un clip falla, cambia el prompt o el fotograma, no repitas.
8. `--resolution 360p|720p` solo existe para `omni-flash`. Si priorizas ahorro y la calidad de movimiento de omni-flash te vale, 360p/720p reduce lo que generas, pero **verifica el coste real** con `credits user` antes y después de un clip.

### Etapa 3: Selección (sin coste)
Descarta clips malos **antes** de reescalar. El reescalado cuesta GPU, y reescalar lo que luego no usas es el desperdicio más común.

### Etapa 4: Reescalado
Decide el destino antes:

| Destino | Qué hacer |
|---|---|
| **1080p** (suficiente para casi todo YouTube) | `gflow video upscale <mediaId> --scale 1080p`, o reescalado local de 720p → 1080p |
| **4K** (solo planos concretos o si quieres el sello 4K) | Reescalado local con modelo ESRGAN + reducción a 2160p |

Reescalado local, basado en `4k-ai-video-workflows`:

1. **Acondicionar con ffmpeg:** normaliza a 24 fps, mismo códec y mismo tamaño en todos los clips.
2. **ComfyUI:** carga el workflow *MP4 → 4K* del repo, que usa `4x_foolhardy_Remacri.pth`. Este modelo es **×4**: un clip 720p queda en 2880p, y para 2160p tienes que **añadir un nodo de redimensionado** después del reescalado (el workflow original no lo trae; es una modificación mía, hay que probarla). La tabla del repo da la regla: para un destino de 3840×2160 con modelo ×4 la entrada ideal es 960×540.
3. **Interpolación opcional:** el workflow *WEBP → 4K* añade `RIFE VFI` (×2). Úsalo solo si el movimiento se ve entrecortado; puede crear artefactos.
4. **VRAM:** el repo recomienda ejecutar cada paso como un workflow separado y reiniciar ComfyUI si salen resultados corruptos (autor con 8 GB de VRAM).
5. **Salida:** h264, CRF 10 es lo que usa el repo; para subir a YouTube puedes bajar a CRF 16–18 y ahorrar espacio.

**UltimateSDUpscale** entra aquí solo para **imágenes**: miniaturas y planos estáticos grandes. Para una miniatura, el reescalado gratis de Flow (2K) suele bastar; usa USDU si necesitas más detalle o 4K sin Ultra, aceptando que añade detalle inventado.

### Etapa 5: Locución y montaje
Igual que en la guía anterior (CapCut u otro). Monta con los clips reescalados.

---

## 5. Hardware y plan confirmados

Host: **NVIDIA RTX con 8 GB de VRAM**, Intel i5 de 12.ª generación, Ubuntu Server 24.04 sin pantalla. Plan de Flow: **Pro**. Agente: **OpenClaw** (agente de IA, similar a hermes-agent).

Consecuencias:

| Dato | Efecto en el diseño |
|---|---|
| RTX 8 GB | Es la misma clase de GPU con la que el autor de `4k-ai-video-workflows` hizo sus workflows (RTX 4060 de 8 GB). El reescalado ESRGAN (Remacri) de clips de Veo es viable en local. La generación con WAN no nos interesa: para eso está Flow. |
| Plan Pro | `gflow image upscale --scale 4k` **no está disponible** (solo Ultra; falla con código 22). Para imágenes: 2K gratis de Flow, y 4K con UltimateSDUpscale en local. |
| i5 12.ª gen | Sobra para ffmpeg, conformado de clips y orquestación. No lo uses para reescalar (cuello de botella). |
| Servidor sin pantalla | Xvfb para Chrome de gflow + VNC solo para el login (sección 3). El driver NVIDIA en Ubuntu 24.04 se instala con `ubuntu-drivers`; compruébalo con `nvidia-smi`. |

Reglas para 8 GB de VRAM (del propio repo y de la lógica del modelo, sin medir):

1. Procesa **un clip cada vez** y reinicia ComfyUI si los resultados salen corruptos.
2. Reescala **clips de 720p**, no de 1080p. Remacri es ×4: un fotograma 720p ya llega a 2880p, así que ni hace falta ni cabe más resolución.
3. Si hay error de memoria, el repo menciona el decodificador VAE por teselas; para el reescalado puro (sin VAE) normalmente no hace falta, pero el nodo `ImageUpscaleWithModel` procesa por lote: baja el lote o trocea el clip en segmentos cortos.
4. No ejecutes a la vez gflow/Chrome y ComfyUI con cargas pesadas: Chrome también consume RAM del sistema.
5. Mide el tiempo con **un clip de prueba** antes de planificar volumen. No tengo cifras de rendimiento para Remacri sobre Veo en esta GPU.

### OpenClaw como orquestador

OpenClaw es un agente de IA. No tengo documentación suya, así que esto es la integración mínima segura, no una descripción de OpenClaw:

- **Opción A (recomendada):** que OpenClaw ejecute comandos de shell con una lista corta y fija: `gflow credits user`, `gflow image t2i`, `gflow video i2v`, `gflow image upscale`, y un script local `upscale_clip.sh`. Así controlas qué puede gastar.
- **Opción B:** si OpenClaw soporta MCP, apuntarlo a `gflow mcp run`. Da más autonomía, pero el agente podría gastar créditos de vídeo sin tu aprobación.
- **Límite de seguridad en ambos casos:** el agente no debe poder lanzar `video` sin un presupuesto. Pon un script envoltorio que lea `gflow credits user`, rechace si el saldo baja de un umbral y limite la cantidad por orden a 1.
- OpenClaw no debe tocar el navegador de gflow ni el perfil `GFLOW_CLI_HOME`. Si OpenClaw tiene su propio navegador, que sea otro perfil y otra sesión.

---

## 6. Resumen de ahorro

| Palanca | Efecto |
|---|---|
| Iterar en imágenes (gratis) y pasar a vídeo solo lo aprobado | Evita regenerar vídeo |
| `i2v` desde fotograma aprobado, `veo-lite`, 1 variante, 4–6 s | Mínimo coste por clip |
| `veo-quality` solo en planos héroe | Gasta donde se nota |
| `image upscale` 2K y `scene` en servidor | Resolución y montaje sin créditos |
| Seleccionar antes de reescalar | Menos horas de GPU |
| Reescalado local o 1080p nativo | Resolución sin coste de generación |

## 7. Qué no está verificado

- Que el login de Google funcione dentro de Xvfb + VNC en tu servidor (el proyecto lo marca como no probado en contenedor).
- El coste en créditos de `video upscale`, de `veo-lite` frente a `veo-fast` y `veo-quality`, y de `omni-flash` a 360p/720p. Se mide con `gflow credits user` antes y después.
- La integración con OpenClaw.
- Que el workflow *MP4 → 4K* acepte clips de Veo sin cambios (el repo lo diseñó para salidas de FramePack/WAN).
- Los nombres exactos de las opciones de cantidad y dry-run: confirma con `gflow video i2v --help` y `gflow video chain --help`.
