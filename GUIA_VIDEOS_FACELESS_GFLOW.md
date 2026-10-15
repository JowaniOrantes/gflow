# Guía de producción de vídeos faceless con IA + gflow

Adaptación del contenido útil del manual "Lanza tu canal de YouTube sin cara en 7 días con IA" (Viralyt), centrada en producir **vídeo e imagen con gflow-cli** (Google Flow: Veo para vídeo, Imagen para imagen).

**Qué se ha descartado del original:** la academia y su venta, la búsqueda y validación de micronichos, la tabla de RPM, las cifras de éxito personales y el "calentamiento" de cuenta. Se parte de que ya tienes tema o nicho elegido.

**Aviso:** los comandos `gflow` de esta guía salen del README de gflow-cli (v0.85.0). Antes de automatizar nada, comprueba cada comando con `gflow --help` o `gflow <comando> --help`, porque el proyecto está en alfa y las opciones pueden cambiar.

---

## 0. Cómo hablarle a la IA

Un buen prompt tiene cuatro partes:

1. **Rol**: quién tiene que ser la IA.
2. **Contexto**: tu tema, idioma, nivel y presupuesto reales.
3. **Tarea concreta**: UNA cosa específica y medible.
4. **Formato de salida**: tabla, lista, nº de palabras, variante del idioma.

Regla de uso: pega el prompt, sustituye todo lo que está entre `[CORCHETES]` y envía. Empieza cada sesión con el prompt maestro.

### Prompt maestro

```
Vas a ser mi consultor experto en canales de YouTube faceless (sin mostrar la cara) para el mercado [MERCADO/IDIOMA]. Tienes experiencia real construyendo y escalando este tipo de canales.

Mi situación:
- Tema del canal: [TEMA]
- Idioma del canal: [español de España / español LATAM / inglés]
- Nivel de experiencia: [ninguno / he probado antes / tengo un canal]
- Herramienta de generación visual: gflow-cli (Google Flow: Veo para vídeo, Imagen para imagen)
- Tiempo disponible: [X] horas al día
- Presupuesto para herramientas: [0€ / X€ al mes]

Reglas para trabajar conmigo:
1. Sé directo y práctico. Nada de teoría genérica.
2. Decide con lógica de negocio y datos, no con entusiasmo.
3. Si te falta información, pregúntame antes de inventar.
4. Cuando escribas textos para el canal, usa español natural de mi mercado, sin que suene a traducción.
5. Si algo que te pido es mala idea para mi situación, dímelo antes de hacerlo.

Confirma que lo has entendido y espera mi primera tarea.
```

> La IA no tiene datos en tiempo real de YouTube. Úsala para estructurar, redactar y analizar lo que tú le pegues, no para afirmar qué está funcionando hoy.

---

## 1. Análisis de referencia (estructura, no contenido)

Se copia la **estructura** (temas, patrones de título, huecos), nunca el contenido.

Para 3 canales de referencia de tu tema: abre la pestaña **Vídeos**, ordena por **Más populares** y ejecuta el prompt en un chat separado por canal.

### Prompt A: análisis de canal

```
Analiza este canal de YouTube que tengo en pantalla y hazme un informe completo:

1. Identifica los 10 vídeos con más visualizaciones.
2. Extrae la fórmula de título de cada uno (la estructura, no el título).
3. Categoriza todos los vídeos del canal por tema.
4. Dime qué categorías y patrones de título consiguen sistemáticamente más visitas.
5. Encuentra los huecos: temas con bajo rendimiento o temas obvios que no ha tocado.
6. Analiza la duración: ¿los que mejor funcionan son largos o cortos?
7. Dame las 10 mejores ideas de vídeo que yo debería crear.

Sé específico. No quiero categorías genéricas, quiero el patrón exacto.
```

### Prompt B: banco de 30 ideas

```
Voy a lanzar un canal de YouTube faceless sobre [TEMA] en [IDIOMA].

Informes de análisis de 3 canales de referencia:
[PEGA LOS 3 INFORMES DEL PROMPT A]

Genera 30 ideas de vídeo. Para cada una:
- Tema
- Por qué debería funcionar (según los patrones de los informes)
- Duración recomendada
- Dificultad de producción con IA generativa (baja / media / alta)
- Potencial de visitas (alto / medio / bajo)

Reparte: 20 que sigan patrones ya probados y 10 que ataquen huecos detectados.
Ordénalas por prioridad (potencial + facilidad de producción) y dime cuál debe ser mi PRIMER vídeo. Criterio: alto potencial pero fácil de terminar.
```

**Adaptación a gflow:** añade al criterio de dificultad cuántos planos de vídeo necesita la idea. Cada clip de Veo cuesta créditos de Flow; las imágenes y la composición, según el README, no.

---

## 2. Identidad del canal (máximo 1 hora)

El branding se cambia en cualquier momento. La única parte con impacto real es la descripción del canal.

### Prompt C: pack de identidad

```
Voy a crear un canal de YouTube faceless sobre [TEMA] en [IDIOMA].
Tipo de contenido: [UNA FRASE].

Entrégame:
1. NOMBRES: 15 opciones, fáciles de escribir y recordar, que no me encierren si amplío temas. Marca 3 favoritos y explica por qué.
2. DESCRIPCIÓN DEL CANAL: 3-5 líneas con de qué trata, qué vídeos habrá y cada cuánto se publica. Keywords naturales, sin lista.
3. TEXTO DEL BANNER: máximo 6 palabras.
4. BRIEF VISUAL: paleta de 2-3 colores con su porqué, estilo recomendado y qué debe verse en la foto de perfil para leerse en móvil.

Español natural de [MERCADO].
```

**Con gflow:** genera la foto de perfil y el banner con `gflow image t2i "<prompt visual>"`, usando el brief visual del paso 4. Para el banner, pide el aspecto adecuado con la opción de aspecto del comando, y comprueba en `gflow image t2i --help` cuál admite.

---

## 3. Título y miniatura antes del guion

El título y la miniatura deciden el clic. Se diseñan **antes** de escribir el guion, y el guion cumple la promesa.

### Fórmulas de título

| Fórmula | Ejemplo | Por qué funciona |
|---|---|---|
| Curiosidad abierta | "Nadie habla de esto..." | Deja un hueco que el cerebro quiere cerrar |
| Información oculta | "Lo que no te contaron sobre..." | Promete acceso a algo reservado |
| Contradicción | "Todo lo que creías saber es falso" | Choca con una creencia |
| Autoridad implícita | "Después de analizar 100 casos..." | El número hace de credencial |
| Número específico | "7 errores que te cuestan dinero" | Concreto, con expectativa clara |
| Comparación | "X frente a Y: cuál gana" | Curiosidad y debate |

El título **tiene que cumplirse**. Un clickbait que no entrega perjudica más que un título flojo.

### Prompt D: generador de títulos

```
Eres especialista en títulos de YouTube para canales faceless en [IDIOMA].
Tema: [TEMA]. Vídeo: [IDEA ELEGIDA].

Patrones de título que funcionan en mis canales de referencia:
[PEGA LOS PATRONES DEL PROMPT A]

Genera 20 títulos de menos de 60 caracteres, del más al menos potente.
Reglas:
- Varía los enfoques (curiosidad, información oculta, contradicción, autoridad, número, comparación).
- Al menos 8 siguen patrones ya probados en mi tema.
- Indica el ángulo psicológico entre paréntesis.
- Sin mayúsculas gritadas ni exclamaciones acumuladas.
- Ninguno promete algo que el vídeo no pueda cumplir.

Al final, elige el que publicarías tú y explica por qué.
```

### Reglas de miniatura

- Máximo **3 elementos visuales**.
- **3 a 5 palabras** de texto, que completan el título y no lo repiten.
- **Alto contraste**, pensando en pantalla de móvil.
- Que cuente una historia en una sola imagen. Estilo cinematográfico y realista.
- Faceless no significa sin emoción: se transmite con contraste, flechas, círculos y composición.

### Prompt E: diseño de miniatura

```
Eres experto en miniaturas de YouTube para canales faceless.

Título: [TÍTULO ELEGIDO]
Tema: [TEMA]
Emoción a provocar: [CURIOSIDAD / SORPRESA / AMBICIÓN / TENSIÓN]
Características de las miniaturas de mis referencias: [COLORES, TEXTO, COMPOSICIÓN OBSERVADOS]

Entrégame:
1. CONCEPTO PRINCIPAL: qué se ve, dónde está cada elemento, colores y por qué funciona.
2. 4 VARIANTES del mismo concepto.
3. 5 OPCIONES DE TEXTO de 3 a 5 palabras que complementen el título.
4. PROMPT DE GENERACIÓN DE IMAGEN en inglés, con estilo, iluminación, composición y encuadre. Sin texto dentro de la imagen: el texto se añade después.

Requisito: máximo 3 elementos, alto contraste y legible en móvil.
```

**Con gflow:**

```bash
# Una variante por prompt; lanza las 4 y compara
gflow image t2i "<prompt en inglés del punto 4>" --aspect 16:9

# Si quieres que gflow reescriba un prompt corto en uno más rico antes de generar
gflow image t2i "<idea corta>" --tool creative-director
```

El texto de la miniatura se añade después en un editor (CapCut, Canva, etc.).

### Comprobación del packaging

Coloca tu miniatura y título junto a los de los 5 vídeos más vistos de tu tema, al mismo tamaño. Si no harías clic en el tuyo antes que en los otros cinco, corrige el packaging antes de seguir.

### Prompt F: auditoría de packaging

```
Actúa como el espectador de mi tema, no como mi asistente. Sé duro.

Mi vídeo:
- Título: [TÍTULO]
- Miniatura: [DESCRÍBELA O ADJUNTA LA IMAGEN]

Vídeos con los que competiría en la portada:
[PEGA 5 TÍTULOS]

Responde:
1. Del 1 al 10, ¿cuánta curiosidad genera mi título frente a esos 5?
2. ¿Se entiende de qué va en menos de 2 segundos?
3. ¿Miniatura y título se complementan o repiten?
4. ¿Promete más de lo que el vídeo puede cumplir?
5. Viendo los 6 juntos, ¿en cuál harías clic y por qué?

Si mi combinación no gana, reescribe el título y describe la miniatura que sí ganaría.
```

---

## 4. Guion

Con el título cerrado, el guion cumple la promesa y retiene.

### Estructura

| Bloque | Duración | Qué pasa |
|---|---|---|
| Hook | 0-15 s | Enganchas o pierdes |
| Promesa | 15-30 s | Qué se lleva quien se quede |
| Desarrollo | Cuerpo | Bloques con mini-ganchos cada 60-90 s |
| Clímax | Antes del cierre | Se entrega lo que prometió el título |
| Cierre | Final | Remate memorable + qué ver después |

Mini-ganchos: al cerrar una idea, abre la siguiente ("Y aquí es donde esto se complica").

### Prompt G: guion con el ADN de tu tema

Antes, saca la transcripción de 3 vídeos top de tu tema (menú de los tres puntos → *Mostrar transcripción*). Se usan para extraer cómo enganchan y cierran, **no para reproducir su texto**.

```
Vas a ser mi guionista para un canal de YouTube faceless en [IDIOMA].

FASE 1 — ANÁLISIS
Transcripciones de 3 vídeos que funcionan en mi tema de [TEMA]:
[PEGA LAS 3 TRANSCRIPCIONES]

Extrae: cómo construyen el hook de 15 s, qué estructura narrativa siguen, cada cuánto colocan mini-ganchos y de qué tipo, qué registro y ritmo usan, y cómo cierran.
Enséñame el análisis antes de continuar.

FASE 2 — GUION
Escribe el guion completo:
- Título: [TÍTULO]
- Duración objetivo: [X] minutos
- Estructura: Hook → Promesa → Desarrollo con mini-ganchos cada 60-90 s → Clímax → Cierre

Reglas:
- Cumple literalmente lo que promete el título.
- Frases cortas y habladas; se leerá en voz alta.
- Cero relleno.
- Que no suene a IA: nada de "en el mundo actual" ni "es importante destacar".
- Marca los mini-ganchos entre corchetes.

Escribe el guion entero, no un esquema.
```

### Prompt H: del guion al plan de planos para gflow

Este paso sustituye a "buscar stock" y conecta el guion con la generación.

```
Tengo este guion de [X] minutos:
[PEGA EL GUION]

Conviértelo en una tabla de planos para generar con IA (Veo e Imagen). Para cada plano:
- Nº y marca de tiempo aproximada
- Frase de la locución que cubre
- Tipo: imagen estática / vídeo de texto a vídeo / vídeo desde imagen
- Prompt visual en inglés: sujeto, acción, cámara, luz, estilo. Solo la acción de ese plano; los personajes se referencian por nombre
- Duración (máximo 8 s por clip)
- Personajes que aparecen

Reglas:
- La imagen debe cambiar cada 3-5 segundos.
- Mismo estilo visual en todos los planos (defínelo una vez al principio y repítelo).
- Prioriza imágenes estáticas con movimiento suave donde un clip de vídeo no aporte, para ahorrar créditos.
- Lista aparte los personajes recurrentes con descripción de cara y cuerpo.
```

---

## 5. Producción visual con gflow

### Preparación

```bash
uv tool install gflow-cli
uv tool run --from gflow-cli playwright install chromium   # una vez, ~150 MB
gflow auth login --browser chrome                         # una vez, abre Chrome
gflow credits user                                         # saldo de créditos
```

Lee antes `DISCLAIMER.md` y `docs/ACCOUNT_SAFETY.md` del repo de gflow-cli: usa tu propia cuenta de Google y asumes el riesgo respecto a sus términos de servicio. Según el README, los créditos solo se gastan en vídeo; las imágenes y la composición no.

### Flujo recomendado

1. **Personajes recurrentes** (si los hay), para que sean el mismo en todos los planos:
   ```bash
   gflow character create --project <id> --name "Aria" \
     --face-prompt "..." --body-prompt "..."
   ```
2. **Imágenes**, una por plano estático o como imagen inicial:
   ```bash
   gflow image t2i "<prompt visual del plano>" --aspect 16:9
   ```
3. **Vídeo** de los planos que lo necesiten:
   ```bash
   gflow video t2v "<prompt>" --aspect 16:9     # texto a vídeo
   gflow video i2v ...                          # imagen a vídeo (mira --help)
   gflow video r2v ...                          # con imágenes de referencia
   ```
4. **Planos más largos de 8 s:** `gflow video extend` continúa un clip existente.
5. **Lotes:** el README muestra un bucle de shell sobre un archivo de prompts, y existe `gflow run --project ID` para generar un lote dentro de un proyecto de Flow existente. Revisa su formato con `gflow run --help`.

### Buenas prácticas de ahorro

- Prueba primero con imágenes (gratis según el README) y pasa a vídeo solo lo que lo justifique.
- Genera primero un plano de prueba por estilo antes de lanzar el lote entero.
- Mantén un archivo con los prompts finales y sus resultados para repetirlos o corregirlos.
- Si el CLI falla con un código de salida de "selector drift" (el README dice que es el 23), la interfaz de Flow ha cambiado: no repitas en bucle, revisa `KNOWN_ISSUES.md`.

---

## 6. Locución y montaje

### Locución

- Opción rápida: voz de IA (por ejemplo ElevenLabs, con plan gratuito para empezar).
- Opción con más matiz: locutor humano (Upwork, Workana), con más coste y plazo.
- **Escucha el audio entero** antes de montar. Corrige las frases que escritas funcionaban y habladas no.

### Orden de montaje (CapCut u otro editor)

1. Locución primero: es la columna vertebral.
2. Corta silencios y errores del audio.
3. Monta los clips e imágenes de gflow encima, siguiendo la tabla de planos.
4. Música de fondo muy baja (alrededor del 10-15 % del volumen de la voz).
5. Subtítulos automáticos (mucha gente ve sin sonido).
6. Revisa el primer minuto tres veces.

Regla de oro: **la imagen cambia cada 3-5 segundos**. Un plano fijo de 20 s con voz encima invita a cerrar el vídeo.

Si el vídeo se produce con muchas escenas, FlowKit (`crisng95/flowkit`) cubre el concatenado, la narración y la revisión por IA. Es una alternativa a montarlo todo a mano, con sus propias limitaciones (ver la comparación anterior).

---

## 7. Publicación

### Revisión final

- ¿El primer minuto engancha o hay una introducción recortable?
- ¿El vídeo cumple lo que prometía el título?
- ¿La locución se entiende con la música debajo?
- ¿Algún tramo de más de 5 s con la misma imagen?
- ¿La miniatura se lee a tamaño pequeño?

Déjalo reposar una noche y revísalo con ojos frescos antes de subirlo.

### Prompt de palabras clave (opcional, para el SEO)

```
Voy a publicar un vídeo sobre [TEMA] en [IDIOMA].
Dame una lista de: 10 términos principales, 20 long-tail y 15 preguntas reales de la audiencia, con volumen estimado (alto/medio/bajo) y si dan para un vídeo entero o una sección.
Formato: tabla.
```

(Son estimaciones de la IA, no datos reales de búsqueda: úsalas como punto de partida.)

### Prompt I: descripción, etiquetas y ficha

```
Eres especialista en SEO de YouTube para canales faceless en [IDIOMA].

Vídeo: [TÍTULO]
Tema: [TEMA]
Canal: [NUEVO / CON HISTORIAL]
Guion: [PEGA EL GUION O UN RESUMEN DETALLADO]
Palabras clave: [PEGA LA LISTA]

Entrégame:
1. DESCRIPCIÓN: primer párrafo de 2-3 líneas que enganche, con las keywords principales de forma natural; después un resumen del contenido, sin relleno.
2. CAPÍTULOS: marcas de tiempo con nombres que inviten a quedarse, según la estructura del guion.
3. ETIQUETAS: 15 principales, 15 long-tail y 10 de contexto, separadas por comas.
4. TEXTO FIJADO: un comentario para fijar que invite a la conversación.

Nada de amontonar keywords: todo debe leerse natural para una persona.
```

El orden de prioridad es **contenido > packaging > SEO**: dedica al SEO unos 15 minutos y pasa al siguiente vídeo.

### Divulgación de contenido sintético

YouTube pide declarar el contenido alterado o generado de forma que parezca realista. Revisa la opción de divulgación al subir el vídeo y compruébalo en la ayuda oficial. Los canales con contenido masivo, repetitivo o sin valor añadido pueden perder la monetización.

### Después de publicar

Con el banco de 30 ideas del paso 1 y los prompts guardados, el vídeo siguiente reutiliza todo el sistema: nueva idea → título → miniatura → guion → tabla de planos → generación con gflow → montaje.

---

## Resumen del flujo

| Paso | Entrada | Salida | Herramienta |
|---|---|---|---|
| 1. Análisis | 3 canales de referencia | Patrones + 30 ideas | IA (prompts A, B) |
| 2. Identidad | Tema | Nombre, descripción, brief visual | IA (C) + `gflow image` |
| 3. Packaging | Idea elegida | Título + miniatura | IA (D, E, F) + `gflow image` |
| 4. Guion | Título + transcripciones | Guion + tabla de planos | IA (G, H) |
| 5. Producción visual | Tabla de planos | Imágenes y clips | `gflow image`, `video`, `character` |
| 6. Locución y montaje | Guion + clips | Vídeo final | TTS + editor |
| 7. Publicación | Vídeo + guion | Ficha optimizada | IA (I) + YouTube |
