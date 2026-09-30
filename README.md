[![English](https://img.shields.io/badge/English-f2e3e3?style=for-the-badge&labelColor=f2e3e3&color=b07070)](README.en.md)
[![Español](https://img.shields.io/badge/Espa%C3%B1ol-8b1a1a?style=for-the-badge)](README.es.md)

# shuohao-skills

**Colección de skills para producción de microdramas con IA** — de una novela a material listo para rodar: bíblias de personajes, esquemas de adaptación, bíblias de arte (escenarios y utilería), guiones y storyboards. Diseñados para agentes de código con IA; **funcionan tanto en Claude Code como en codex**.

Así se ve la línea completa — **el esquema converge la estructura; guion, escenarios y personajes iteran juntos; el storyboard solo exporta, no toma decisiones nuevas**:

<img src="assets/pipeline-en.webp" alt="Flujo de producción de microdramas con IA" width="680">

| Skill | Qué hace |
| --- | --- |
| [**novel-outline**](skills/novel-outline) | Adapta una novela al paquete de 5 piezas del esquema de un microdrama: notas de adaptación, elenco, ganchos, sinopsis por episodio y lista de activos (incluida la tabla de utilería narrativa). Las 14 puertas de calidad se verifican por script; incluye modo de revisión para esquemas ya existentes |
| [**novel-characters**](skills/novel-characters) | Convierte el elenco definido en el esquema en una bíblia de personajes: perfiles, prompts de diseño, prompts de voz y láminas de personaje. Se alimenta de `outline.json`; idioma del informe configurable |
| [**character-refs**](skills/character-refs) | **Genera de verdad** imágenes de referencia para personajes de cualquier historia (no solo novelas): describes al personaje en un mensaje, se desglosan los campos, se confirman, primero se genera el ancla frontal de cuerpo entero y las demás vistas (rostro, perfil 90°, espalda, detalles) referencian solo esa ancla. Cada imagen queda identificada y es regenerable por separado, con control de obsolescencia. Compatible con Qwen Image (ComfyUI), generación local de codex, GPT Image 2 API o comando propio |
| [**novel-art**](skills/novel-art) | Bíblias de arte para producción con IA (escenarios + utilería narrativa): anclas de consistencia, variantes de luz y estado, referencias de escala, fondos blancos sin personas ni manos. Se alimenta de `outline.json`; las 10 puertas de calidad se verifican por script |
| [**novel-script**](skills/novel-script) | Guion para microdramas con IA: escenas + flujo de beats (acción alternando con diálogo), duración por episodio estimada de forma determinista según la velocidad de lectura, gancho de arranque en frío dentro de los primeros 3 beats verificado como puerta, libro de líneas por personaje con prompts de voz listo para TTS. Las 10 puertas de calidad se verifican por script |
| [**novel-storyboard**](skills/novel-storyboard) | Storyboard para microdramas con IA: segmentos (una sola generación, ≤15 s) → cortes (puerta dura de 2–5 s) → fotogramas clave (principal fijado en 0,00 s, subfotogramas en sus marcas de corte), con auditoría literal de la alineación y tiempos del prompt MiniMax H3; los fotogramas se generan de verdad usando las láminas de diseño como referencia, y `export` produce en un comando los paquetes de producción **H3 / Seedance / Omni (Google Flow · gemini-omni-1.1-flash, con cuerpos de request API listos para POST)**. Las 19 puertas de calidad se verifican por script |

## Novedades de esta rama (optimización para Google Flow)

Este fork añade soporte de **tercer protocolo** en `novel-storyboard`, orientado a **Gemini Omni Flash 1.1 (`gemini-omni-1.1-flash`)**, el modelo que utiliza Google Flow:

- **Nuevo generador de prompts `omniPrompt()`**, siguiendo la documentación oficial:
  - Declaración de fuentes/referencias con sintaxis oficial: `[# Sources <FIRST_FRAME>@ImageN]` (el primer fotograma del storyboard actúa como fotograma inicial literal) y `[# References <IMAGE_REF_k>@ImageN …]` (las demás imágenes son referencias de consistencia, no fotogramas literales).
  - Capa temporal con timecodes oficiales `[0-3s] … [3-6s] …` derivados de forma determinista de los cortes del storyboard.
  - Por corte: cámara traducida a frases naturales en inglés, plano, lente, posición, composición, iluminación, línea de mirada y foco.
  - Diálogos textuales entrecomillados por hablante, más líneas explícitas `Sound design:` y `Music:` (la doc oficial insiste en describir el audio).
  - Bloque `Constraints:` numerado con negativos escritos inline, porque Omni **no soporta parámetros de negative prompt**.
  - Los prompts se generan **en inglés** (la documentación oficial solo evalúa prompts en ese idioma), aunque el guion interno pueda estar en chino.
- **Tercer protocolo de exportación `--protocol omni`**:
  - Por segmento genera `omni.md` (prompt listo para pegar en Google Flow + orden exacto de subida de imágenes) y **`omni-request.json`** — cuerpo de request listo para POST al endpoint `/v1beta/interactions` con `model: gemini-omni-1.1-flash`, imágenes base64 (placeholders) y `response_format: {type:"video", aspect_ratio:"9:16", resolution:"720p"}` (vertical 9:16 = formato microdrama; opciones oficiales 360p/720p/1080p/4k).
  - En la raíz del paquete: `omni-manifest.json` con roles por anexo (`<FIRST_FRAME>` / `<IMAGE_REF_k>`), segundos, cortes e imágenes faltantes marcadas.
- **Puerta de calidad #19 `omni-shot`**: valida que el texto de toma Omni (`cut.shotOmni`) esté en inglés y sin símbolos prohibidos (timecodes escritos a mano, segundos, comillas de diálogo, `[Shot k]`). Retrocompatible: si no hay `shotOmni` y el shot está en chino, se delega la composición al fotograma clave; los protocolos H3/Seedance quedan intactos.
- **Interfaz en español (`--lang es`)**: informes HTML/markdown completamente en español (título, paneles "Puertas de calidad 19 / 19", etiquetas de cada puerta, textos de CLI). La interfaz de datos del repo sigue en chino por defecto; `--lang en` sigue disponible en inglés.
- **Guía documentada**: nueva referencia [`references/omni-flash-prompt.md`](skills/novel-storyboard/references/omni-flash-prompt.md) con las normas de escritura estilo `seedance-prompt.md`, campo `shotOmni` documentado en `references/schema.md` y `SKILL.md` actualizado con el tercer protocolo.

Uso rápido:

```bash
node scripts/novel-storyboard.mjs render <demo>/storyboard --lang es   # informe en español
node scripts/novel-storyboard.mjs export <demo>/storyboard --protocol omni --out <demo>/storyboard
# genera por segmento: omni.md + omni-request.json, y omni-manifest.json en la raíz
```

## Cómo se ve

Cinco conjuntos salen de una novela:

**novel-outline · Esquema de adaptación**

![Informe de esquema](skills/novel-outline/assets/report.webp)

**novel-characters · Bíblia de personajes**

![Informe de personajes](skills/novel-characters/assets/report.webp)

**novel-art · Bíblia de arte (escenarios + utilería; las láminas las genera el skill)**

![Informe de arte](skills/novel-art/assets/report.webp)

**novel-script · Guion (medidor de duración + guion por episodio + libro de líneas)**

![Informe de guion](skills/novel-script/assets/report.webp)

**novel-storyboard · Storyboard (banda de ritmo + fotogramas principales/secundarios generados por el skill + prompts H3/Omni)**

![Informe de storyboard](skills/novel-storyboard/assets/report.webp)

## Instalación

```bash
git clone https://github.com/Cosio33/shuohao-skills-CosioFork.git
cd shuohao-skills-CosioFork
./scripts/install.sh
```

Detecta automáticamente si tienes Claude Code o codex y **enlaza simbólicamente** todos los skills — tras `git pull` surte efecto de inmediato, sin reinstalar.

## Requisitos

| | ¿Obligatorio? | Notas |
| --- | --- | --- |
| **Node** | Sí | ≥ 18. Solo usa la librería estándar; **sin dependencias npm, sin instalar nada** |
| **Cuota de modelo** | Sí | Usa la cuota de tu sesión actual; **no requiere ninguna API key** |
| **codex CLI** | Opcional | Equivalente a Claude Code como entorno de ejecución. Los cinco skills de la línea **no generan imágenes**; solo character-refs necesita un modelo de imagen |

## Convenciones del repositorio

Un directorio por skill, **autocontenido y copiable por separado**:

```
skills/<nombre-skill>/
├── SKILL.md          flujo de trabajo para el agente (obligatorio)
├── README.md         guía para humanos
├── scripts/
│   ├── <name>.mjs    herramienta determinista, cero dependencias
│   └── selftest.mjs  auto-prueba sin llamar al modelo (obligatoria)
├── references/       instrucciones detalladas bajo demanda
├── examples/         ejemplos incluidos, usados también como fixtures de test
└── assets/           capturas
```

Antes de añadir un skill nuevo, ejecuta todas las auto-pruebas:

```bash
for f in skills/*/scripts/selftest.mjs; do node "$f"; done
```

La suite de `novel-storyboard` incluye ahora casos para `omniPrompt` y la puerta `omni-shot`: **344 pruebas en verde**.

## Licencia

[Apache 2.0](LICENSE)
