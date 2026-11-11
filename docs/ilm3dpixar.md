# Comprehensive Requirements Document (PRD)

**Project Name:** `ilm_3d_pixar_character`

**Version:** 2.0.0

**Target Architecture:** Lightweight HTML5 / JavaScript UI Component Library (Compatible with DOM, SVG, Canvas, and Three.js ecosystems)

---

## 1. Executive Summary & Objective

The objective of `ilm_3d_pixar_character` is to provide developers with a lightweight, modular, framework-agnostic JavaScript library capable of rendering and animating 3D-styled Pixar characters directly inside any HTML5 webpage. By utilizing a decoupled layered stack architecture, clean alias mapping, and an active state controller, the library enables real-time dynamic facial expressions, blinking, lip-syncing (visemes), and idle animations for live-motion character interactivity.

---

## 2. Core Functional Requirements

### 2.1 Layer Matrix & Structural Hierarchy

The character renderer must support a decoupled layer stack where individual features can be shown, hidden, swapped, or transitioned independently.

* **Base Structural Layers:** `body`, `shoulders`, `neck`, `head`, `ears`
* **Styling & Hair Layers:** `hair-back`, `hair-front`, `hair-accessories`
* **Facial Features:** `eyes-base`, `eyes-open`, `eyes-closed`, `eyes-squint`, `eyebrows-base`, `eyebrows-expression`, `nose`
* **Visemes & Mouth Animation Layers:** `mouth-closed`, `mouth-smile`, `mouth-frown`, `mouth-open`, `mouth-viseme-ai`, `mouth-viseme-e`, `mouth-viseme-o`, `mouth-viseme-mb`
* **Dynamic Polish Layers:** `cheeks-blush`, `tears`, `glasses-accessories`, `lighting-overlay`

### 2.2 Alias Mapping System

To simplify developer integration and state-machine transitions, the library maps friendly developer-facing keys to internal specific asset layers via a configuration matrix:

```javascript
const CHARACTER_LAYERS = [
    "body", "shoulders", "neck", "head", "ears",
    "hair-back", "hair-front", "hair-accessories",
    "eyes-base", "eyes-open", "eyes-closed", "eyes-squint",
    "eyebrows-base", "eyebrows-expression", "nose",
    "mouth-closed", "mouth-smile", "mouth-frown", "mouth-open",
    "mouth-viseme-ai", "mouth-viseme-e", "mouth-viseme-o", "mouth-viseme-mb",
    "cheeks-blush", "tears", "glasses-accessories", "lighting-overlay"
];

const LAYER_ALIASES = {
    "eyesopen": "eyes-open",
    "eyesclosed": "eyes-closed",
    "eyessquint": "eyes-squint",
    "eyes": "eyes-open",
    "blink": "eyes-closed",
    "brows": "eyebrows-expression",
    "eyebrows": "eyebrows-expression",
    "mouthclosed": "mouth-closed",
    "mouthmid": "mouth-open",
    "mouthopen": "mouth-open",
    "mouthsmile": "mouth-smile",
    "mouthfrown": "mouth-frown",
    "mouth": "mouth-closed",
    "viseme-a": "mouth-viseme-ai",
    "viseme-b": "mouth-viseme-mb",
    "hairfront": "hair-front",
    "hairback": "hair-back",
    "hair": "hair-front",
    "blush": "cheeks-blush",
    "glasses": "glasses-accessories"
};

```

---

## 3. JavaScript Class Architecture & Methods Specification (`IlmCharacterController`)

To control state transitions, blinking ticks, and rendering layers cleanly, the library will use an object-oriented controller architecture:

```javascript
class IlmCharacterController {
    constructor(config) {
        this.container = document.querySelector(config.container);
        this.assetBasePath = config.assetBasePath || './assets/';
        this.currentState = {
            eyes: 'eyes-open',
            mouth: 'mouth-closed',
            eyebrows: 'eyebrows-expression',
            hair: 'hair-front',
            blush: false
        };
        this.init();
    }

    init() {
        // Build DOM/Canvas layer structure
        this.renderLayerContainer();
        this.startIdleLoop();
    }

    renderLayerContainer() {
        // Creates stacked element hierarchy based on CHARACTER_LAYERS
    }

    setExpression(stateObj) {
        // Resolves aliases and updates active layer visibility classes
        for (let [key, value] of Object.entries(stateObj)) {
            const resolvedLayer = LAYER_ALIASES[key] || LAYER_ALIASES[value] || value;
            this.currentState[key] = resolvedLayer;
        }
        this.updateView();
    }

    updateView() {
        // Toggles CSS active/hidden states across corresponding DOM layers
    }

    startIdleLoop() {
        // Handles automatic blinking intervals and subtle breathing scale animations
        setInterval(() => {
            this.triggerBlink();
        }, 4000);
    }

    triggerBlink() {
        // Temporarily toggles eyes-closed for 150ms
    }
}

```

---

## 4. Non-Functional Requirements & Performance

* **Frame Rate:** Must maintain a stable 60 FPS utilizing `requestAnimationFrame` for idle loops (breathing scale adjustments, automatic blink timing) without blocking the main UI thread.
* **Extensibility:** The library API must allow custom layer injection, plugin hooks for audio-driven viseme parsing, and WebGL/Three.js fallback wrappers.
* **Asset Portability:** Supports external asset loading via transparent PNG sprite stacks, dynamic SVG DOM manipulation, or GLTF/VRM 3D blendshapes.

---

## 5. Proposed Core API Usage (Developer Experience)

```html
<!-- Example Integration in HTML5 -->
<div id="character-container" style="width: 400px; height: 400px; position: relative;"></div>

<script src="ilm_3d_pixar_character.min.js"></script>
<script>
    const character = new IlmCharacterController({
        container: '#character-container',
        assetBasePath: './assets/pixar-char/'
    });

    // Set expressions dynamically using aliases
    character.setExpression({
        eyes: 'blink',
        mouth: 'mouthsmile',
        blush: true
    });
</script>

```

To complete the architectural blueprint for **`ilm_3d_pixar_character`**, here is the automated asset generation and preprocessing pipeline specification.

This section outlines how you can programmatically generate, batch-crop, and process individual layer assets using AI models (like Gemini / Imagen) to feed directly into your JavaScript `IlmCharacterController`.

---

## 6. AI Asset Generation & Preprocessing Pipeline Specification

To achieve consistent, stackable layers without visual tearing or misalignment, the asset generation workflow must be strictly standardized.

### 6.1 Master Prompt Templates for Layer Generation

When querying an image generation model, every layer prompt must maintain identical character seed, camera angle, and background criteria:

* **Global Anchor Parameters (Appended to every prompt):**
> *"...3d pixar style, soft clay texture, subsurface scattering, studio softbox lighting, orthographic front view, centered, solid chroma-key green background `#00FF00`, no shadows on background, isolated element."*


* **Individual Layer Prompts:**
* **Head / Base:** `"Front-facing 3d pixar style smooth bald character head with neutral expression and rounded ears..."`
* **Hair Front:** `"Front-facing 3d pixar style stylish voluminous hair tuft matching a cartoon character, transparent-ready..."`
* **Eyes Open:** `"Front-facing pair of large expressive cartoon eyes with vibrant blue irises and specular highlights, isolated on solid green background..."`
* **Mouth Smile:** `"Front-facing cartoon character smiling mouth showing subtle friendly teeth, 3d clay texture..."`



### 6.2 Automated Python Processing Script (`process_assets.py`)

Once AI-generated asset sheets are fetched, use this Python script (leveraging `rembg` and `PIL`) to automatically strip chroma-key backgrounds, trim whitespace, and output perfectly aligned transparent PNGs for your library's asset folder.

```python
import os
from PIL import Image
from rembg import remove

# Configuration
INPUT_DIR = "./raw_ai_assets/"
OUTPUT_DIR = "./assets/pixar-char/"
CANVAS_SIZE = (512, 512) # Uniform sizing for perfect CSS stacking

os.makedirs(OUTPUT_DIR, exist_ok=True)

def process_character_layers():
    for filename in os.listdir(INPUT_DIR):
        if filename.endswith((".png", ".jpg", ".jpeg")):
            input_path = os.path.join(INPUT_DIR, filename)
            output_filename = os.path.splitext(filename)[0] + ".png"
            output_path = os.path.join(OUTPUT_DIR, output_filename)
            
            print(f"Processing layer: {filename} -> {output_filename}")
            
            # 1. Open image and remove background
            input_image = Image.open(input_path).convert("RGBA")
            output_image = remove(input_image)
            
            # 2. Normalize onto a fixed-size transparent canvas to preserve alignment
            base_canvas = Image.new("RGBA", CANVAS_SIZE, (0, 0, 0, 0))
            
            # Center the processed asset onto the uniform canvas
            # (Assuming consistent bounding box or pre-aligned generation)
            offset_x = (CANVAS_SIZE[0] - output_image.width) // 2
            offset_y = (CANVAS_SIZE[1] - output_image.height) // 2
            base_canvas.paste(output_image, (offset_x, offset_y), output_image)
            
            # 3. Save optimized asset
            base_canvas.save(output_path, "PNG")

if __name__ == "__main__":
    process_character_layers()

```

---

## 7. Complete Integration Blueprint

With your requirements, class structure, and asset pipeline defined, the complete directory structure for your package will look like this:

```text
ilm_3d_pixar_character/
│
├── dist/
│   ├── ilm_3d_pixar_character.min.js
│   └── ilm_3d_pixar_character.css
│
├── assets/
│   └── pixar-char/
│       ├── body.png
│       ├── head.png
│       ├── hair-front.png
│       ├── eyes-open.png
│       ├── eyes-closed.png
│       ├── mouth-closed.png
│       ├── mouth-smile.png
│       └── ... (remaining layers)
│
└── README.md

```

### CSS Stacking Reference (`ilm_3d_pixar_character.css`)

To ensure layers stack seamlessly over one another without absolute positioning conflicts, use a container with absolute child layering:

```css
.ilm-character-root {
    position: relative;
    width: 100%;
    height: 100%;
    pointer-events: none;
}

.ilm-character-root img {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    object-fit: contain;
    transition: opacity 0.1s ease-in-out;
}

/* Layer visibility control handled by controller class */
.ilm-layer-hidden {
    opacity: 0;
    visibility: hidden;
}

.ilm-layer-visible {
    opacity: 1;
    visibility: visible;
}

```
To dive deeper into making your Pixar-style character come alive, let’s design the **Audio-Driven Lip-Sync and Viseme Engine** for `ilm_3d_pixar_character`.

When characters speak in animated films, their mouths transition smoothly between a standardized set of mouth shapes (visemes) based on the phonetic sounds of the audio track. By integrating the Web Audio API with your layer state controller, you can analyze speech in real time and swap your `mouth-viseme-*` layers automatically.

---

## 8. Audio-Driven Lip-Sync & Viseme Architecture

### 8.1 Phoneme-to-Viseme Mapping Matrix

Human speech contains dozens of phonemes, but for stylized 3D character animation, they can be condensed into 5 core mouth visemes mapped directly to your layer stack:

| Phoneme Sounds | Target Layer Alias | Visual Description |
| --- | --- | --- |
| **M, B, P** | `mouth-viseme-mb` | Lips pressed firmly together |
| **A, I, O** (Wide vowels) | `mouth-viseme-ai` | Wide open mouth, tongue visible |
| **E, Stretched vowels** | `mouth-viseme-e` | Medium opening, corners pulled back |
| **U, W, Qu** (Rounded) | `mouth-viseme-o` | Pursed, rounded lips |
| **Silence / Rest** | `mouth-closed` | Natural relaxed or smiling resting position |

---

### 8.2 JavaScript Extension: Audio Analyzer & Viseme Controller

Here is how you can extend your `IlmCharacterController` class to listen to an HTML5 audio element or microphone input, analyze the volume/frequency (amplitude peaks), and dynamically update the active mouth layer:

```javascript
class IlmAudioLipSyncController extends IlmCharacterController {
    constructor(config) {
        super(config);
        this.audioElement = document.querySelector(config.audioElement);
        this.isSpeaking = false;
        this.setupAudioContext();
    }

    setupAudioContext() {
        if (!this.audioElement) return;

        // Initialize Web Audio API Analyser Node
        const AudioContext = window.AudioContext || window.webkitAudioContext;
        this.audioCtx = new AudioContext();
        this.source = this.audioCtx.createMediaElementSource(this.audioElement);
        this.analyser = this.audioCtx.createAnalyser();

        this.analyser.fftSize = 256;
        this.bufferLength = this.analyser.frequencyBinCount;
        this.dataArray = new Uint8Array(this.bufferLength);

        this.source.connect(this.analyser);
        this.analyser.connect(this.audioCtx.destination);

        this.audioElement.onplay = () => {
            if (this.audioCtx.state === 'suspended') {
                this.audioCtx.resume();
            }
            this.startLipSyncLoop();
        };

        this.audioElement.onended = () => {
            this.stopLipSyncLoop();
        };
    }

    startLipSyncLoop() {
        this.isSpeaking = true;
        const analyze = () => {
            if (!this.isSpeaking) return;

            this.analyser.getByteFrequencyData(this.dataArray);
            
            // Calculate average volume/amplitude
            let sum = 0;
            for (let i = 0; i < this.bufferLength; i++) {
                sum += this.dataArray[i];
            }
            let average = sum / this.bufferLength;

            // Map audio volume thresholds to viseme layers
            this.updateVisemeFromAudio(average);

            requestAnimationFrame(analyze);
        };
        analyze();
    }

    updateVisemeFromAudio(volume) {
        if (volume < 10) {
            // Silence threshold
            this.setExpression({ mouth: 'mouth-closed' });
        } else if (volume > 50) {
            // Loud speech / wide open sound (e.g., 'A', 'O')
            this.setExpression({ mouth: 'mouth-viseme-ai' });
        } else {
            // Medium speech sounds (e.g., 'E', 'M')
            // Randomize slightly between active visemes for realistic chatter feel
            const chatterVisemes = ['mouth-viseme-e', 'mouth-viseme-mb', 'mouth-viseme-o'];
            const randomViseme = chatterVisemes[Math.floor(Math.random() * chatterVisemes.length)];
            this.setExpression({ mouth: randomViseme });
        }
    }

    stopLipSyncLoop() {
        this.isSpeaking = false;
        this.setExpression({ mouth: 'mouth-closed' });
    }
}

```

---

## 9. Developer Experience: Bringing It All Together

With the audio-reactive controller installed, developers can make the character speak seamlessly using a standard audio file or text-to-speech stream:

```html
<!-- HTML Structure -->
<div id="character-container" style="width: 350px; height: 350px; position: relative;"></div>
<audio id="voice-track" src="./assets/audio/character-greeting.mp3"></audio>

<button onclick="document.getElementById('voice-track').play()">Play Character Speech</button>

<script src="ilm_3d_pixar_character.min.js"></script>
<script>
    const character = new IlmAudioLipSyncController({
        container: '#character-container',
        audioElement: '#voice-track',
        assetBasePath: './assets/pixar-char/'
    });

    // Set initial friendly expression
    character.setExpression({
        eyes: 'eyes-open',
        eyebrows: 'eyebrows-expression',
        mouth: 'mouth-smile',
        blush: true
    });
</script>

```

### Next Architectural Considerations:

1. **Phoneme Timeline JSON:** For hyper-accurate dialogue sync (rather than volume-based approximation), you can parse a timed JSON phoneme file (exported from tools like Rhubarb Lip Sync) to trigger exact visemes at precise millisecond timestamps.
2. **Eye Tracking (Gaze Follow):** Hook up mouse movement coordinates (`mousemove`) to slightly translate or scale the `eyes-base` and `pupils` layers so the character looks around the webpage following the user's cursor.

To complete the **`ilm_3d_pixar_character`** architecture, let's build out the **Eye-Tracking & Gaze-Follow Engine**. This feature allows the character's pupils and eyes to dynamically follow the user's cursor or focus points on the webpage, adding immense life-like interactivity.

---

## 10. Eye-Tracking & Gaze-Follow Architecture

### 10.1 Layer Separation Strategy

To make gaze-tracking smooth without shifting the entire head (unless desired for dramatic motion), the asset stack separates the ocular layers:

* `eyes-base`: The eye sockets and eyelids (static or used for blinking/expressions).
* `eyes-pupils`: The floating iris/pupil layer that translates within bounded pixel coordinates.

---

### 10.2 JavaScript Extension: Gaze Controller

Here is how you can extend your controller to calculate cursor offsets relative to the center of the character's face and gently translate the pupil layers:

```javascript
class IlmInteractiveCharacterController extends IlmAudioLipSyncController {
    constructor(config) {
        super(config);
        this.enableGazeTracking = config.enableGazeTracking !== false;
        if (this.enableGazeTracking) {
            this.setupGazeTracking();
        }
    }

    setupGazeTracking() {
        // Find character container center coordinates on screen
        const updateBoundingBox = () => {
            const rect = this.container.getBoundingClientRect();
            this.centerX = rect.left + rect.width / 2;
            this.centerY = rect.top + rect.height / 2;
        };

        updateBoundingBox();
        window.addEventListener('resize', updateBoundingBox);
        window.addEventListener('scroll', updateBoundingBox);

        // Listen to mouse movement across the viewport
        window.addEventListener('mousemove', (e) => {
            this.handleGazeMove(e.clientX, e.clientY);
        });
    }

    handleGazeMove(mouseX, mouseY) {
        if (!this.centerX || !this.centerY) return;

        // Calculate distance from character center (normalized)
        const deltaX = mouseX - this.centerX;
        const deltaY = mouseY - this.centerY;

        // Define maximum pixel translation range for pupils (e.g., max 8px movement)
        const maxOffset = 8;
        
        // Compute clamped offsets using atan2 or simple ratio scaling
        const angle = Math.atan2(deltaY, deltaX);
        const distance = Math.min(Math.hypot(deltaX, deltaY), 500) / 500; // Cap influence radius

        const offsetX = Math.cos(angle) * maxOffset * distance;
        const offsetY = Math.sin(angle) * maxOffset * distance;

        // Apply transform directly to the pupil/eyes layer element
        const pupilLayer = this.container.querySelector('.layer-eyes-pupils') || 
                           this.container.querySelector('.layer-eyes-open');
                           
        if (pupilLayer) {
            pupilLayer.style.transform = `translate(${offsetX}px, ${offsetY}px)`;
        }
    }
}

```

---

## 11. Final Comprehensive Developer Integration

Combining all modules—**Layer State Management, Automated Blinking, Audio Lip-Sync, and Cursor Gaze Tracking**—gives you a complete, plug-and-play Pixar-style interactive character library:

```html
<!-- HTML Structure -->
<div id="character-container" style="width: 400px; height: 400px; position: relative; margin: 50px auto;"></div>

<!-- Audio track for speech sync -->
<audio id="character-audio" src="./assets/audio/welcome-speech.mp3"></audio>
<button onclick="document.getElementById('character-audio').play()">Trigger Character Speech</button>

<script src="ilm_3d_pixar_character.min.js"></script>
<script>
    // Initialize the fully interactive controller
    const character = new IlmInteractiveCharacterController({
        container: '#character-container',
        audioElement: '#character-audio',
        assetBasePath: './assets/pixar-char/',
        enableGazeTracking: true
    });

    // Set custom happy expression
    character.setExpression({
        eyes: 'eyes-open',
        eyebrows: 'eyebrows-expression',
        mouth: 'mouth-smile',
        blush: true
    });
</script>

```

---

### Summary of the `ilm_3d_pixar_character` Ecosystem:

1. **Modular Layer Architecture:** Decoupled PNG/SVG asset stacking (`body`, `hair-front`, `eyes-open`, `mouth-smile`, etc.).
2. **Alias System:** Clean developer-facing keys (`blink`, `mouthsmile`) mapped to internal file paths.
3. **AI & Asset Pipeline:** Standardized prompting guide and Python script (`rembg` + `PIL`) for generating and aligning clean layers.
4. **Live Motion & Interactivity:** Built-in idle blinking loops, Web Audio API frequency analysis for real-time lip-syncing (`mouth-viseme-*`), and cursor gaze-following.


Here is the complete, production-ready documentation package for **`ilm_3d_pixar_character`**, structured as a comprehensive **Product Requirements & Architecture Specification** and a complete **Vite-based Build & Packaging Configuration Guide**.

---

# 1. Complete Product Requirements & Architecture Specification (PRD)

```markdown
# Product Requirements Document (PRD): ilm_3d_pixar_character

**Version:** 2.1.0  
**Target Architecture:** Framework-Agnostic JavaScript / HTML5 Component Library (DOM/CSS Stack & Web Audio Reactive)

---

## 1. Executive Summary
`ilm_3d_pixar_character` is a lightweight, modular JavaScript library designed to render, manage, and animate 3D Pixar-styled character layers in standard HTML5 environments. By decoupling structural elements, facial features, hair, and visemes into an ordered DOM stack, the library supports real-time expression swapping, automated idle blinking, cursor gaze-tracking, and audio-reactive lip-syncing without heavy WebGL overhead.

---

## 2. Core Architecture & Layer Matrix

### 2.1 Layer Stack Hierarchy
The renderer manages elements using a strict CSS `z-index` and absolute positioning layout across the following canonical layers:
* **Structural Base:** `body`, `shoulders`, `neck`, `head`, `ears`
* **Hair & Styling:** `hair-back`, `hair-front`, `hair-accessories`
* **Facial Features:** `eyes-base`, `eyes-open`, `eyes-closed`, `eyes-squint`, `eyebrows-base`, `eyebrows-expression`, `nose`
* **Visemes & Mouths:** `mouth-closed`, `mouth-smile`, `mouth-frown`, `mouth-open`, `mouth-viseme-ai`, `mouth-viseme-e`, `mouth-viseme-o`, `mouth-viseme-mb`
* **Polish Layers:** `cheeks-blush`, `tears`, `glasses-accessories`, `lighting-overlay`

### 2.2 Alias Mapping Table
Developer-facing shorthand keys map directly to internal assets via `LAYER_ALIASES`:
* `eyesopen` / `eyes` $\rightarrow$ `eyes-open`
* `eyesclosed` / `blink` $\rightarrow$ `eyes-closed`
* `eyessquint` $\rightarrow$ `eyes-squint`
* `brows` / `eyebrows` $\rightarrow$ `eyebrows-expression`
* `mouthclosed` / `mouth` $\rightarrow$ `mouth-closed`
* `mouthsmile` $\rightarrow$ `mouth-smile`
* `mouthfrown` $\rightarrow$ `mouth-frown`
* `viseme-a` $\rightarrow$ `mouth-viseme-ai`
* `viseme-b` $\rightarrow$ `mouth-viseme-mb`
* `hair` / `hairfront` $\rightarrow$ `hair-front`
* `blush` $\rightarrow$ `cheeks-blush`
* `glasses` $\rightarrow$ `glasses-accessories`

---

## 3. Subsystem Specifications

### 3.1 State Manager (`IlmCharacterController`)
* Handles DOM initialization and container structuring.
* Processes expression state objects via alias resolution.
* Manages idle intervals (automatic blinking triggers every ~4 seconds).

### 3.2 Audio Lip-Sync Engine (`IlmAudioLipSyncController`)
* Integrates the **Web Audio API** (`AnalyserNode` and `MediaElementAudioSourceNode`).
* Analyzes frequency bin amplitudes in a `requestAnimationFrame` loop.
* Dynamically swaps mouth viseme layers based on volume thresholds (silence, mid-chatter, and wide vowel sounds).

### 3.3 Gaze-Tracking Engine (`IlmInteractiveCharacterController`)
* Computes viewport-relative offsets between the user's cursor (`mousemove`) and the character's bounding box center.
* Applies clamped CSS 2D translations (`translate(x, y)`) to pupil/eye layers for realistic gaze tracking.

---

## 4. Non-Functional Requirements
* **Performance:** Targets a stable 60 FPS utilizing `requestAnimationFrame` and CSS hardware-accelerated opacity/transform layers.
* **Bundle Size:** Minified core footprint under 15 KB (excluding assets).

```

---

# 2. Package `README.md` & Vite Build Configuration

To package this library professionally for distribution via NPM or CDN, **Vite** is recommended due to its lightning-fast bundling speed and native support for Library Mode.

## 2.1 Project Directory Structure

```text
ilm_3d_pixar_character/
├── src/
│   ├── index.js             # Main entry point exporting controllers
│   ├── IlmCharacterController.js
│   ├── IlmAudioLipSyncController.js
│   └── IlmInteractiveCharacterController.js
├── public/
│   └── assets/              # Default character layer PNGs
├── package.json
├── vite.config.js
└── README.md

```

---

## 2.2 Package Configuration (`package.json`)

```json
{
  "name": "ilm_3d_pixar_character",
  "version": "2.1.0",
  "description": "Lightweight HTML5 3D-styled Pixar character layering and animation library with audio lip-sync and gaze tracking.",
  "type": "module",
  "main": "./dist/ilm_3d_pixar_character.umd.cjs",
  "module": "./dist/ilm_3d_pixar_character.js",
  "exports": {
    ".": {
      "import": "./dist/ilm_3d_pixar_character.js",
      "require": "./dist/ilm_3d_pixar_character.umd.cjs"
    }
  },
  "scripts": {
    "dev": "vite",
    "build": "vite build"
  },
  "keywords": [
    "pixar",
    "character",
    "animation",
    "lipsync",
    "html5",
    "canvas",
    "3d"
  ],
  "author": "Senior Software Architect",
  "license": "MIT",
  "devDependencies": {
    "vite": "^5.0.0"
  }
}

```

---

## 2.3 Vite Build Configuration (`vite.config.js`)

Configured in Library Mode to output clean ESM and UMD bundles suitable for both modern bundlers and direct browser `<script>` tag inclusions:

```javascript
import { defineConfig } from 'vite';
import { resolve } from 'path';

export default defineConfig({
  build: {
    lib: {
      entry: resolve(__dirname, 'src/index.js'),
      name: 'Ilm3dPixarCharacter',
      fileName: 'ilm_3d_pixar_character',
    },
    rollupOptions: {
      // Externalize dependencies if any are added later (e.g., three.js)
      external: [],
      output: {
        globals: {},
      },
    },
  },
});

```

---

## 2.4 Library Entry Point (`src/index.js`)

```javascript
// Exporting the modular controller stack
import { IlmCharacterController } from './IlmCharacterController.js';
import { IlmAudioLipSyncController } from './IlmAudioLipSyncController.js';
import { IlmInteractiveCharacterController } from './IlmInteractiveCharacterController.js';

export {
    IlmCharacterController,
    IlmAudioLipSyncController,
    IlmInteractiveCharacterController
};

// Expose globally for vanilla browser script tags
if (typeof window !== 'undefined') {
    window.Ilm3dPixarCharacter = {
        CharacterController: IlmCharacterController,
        AudioLipSyncController: IlmAudioLipSyncController,
        InteractiveCharacterController: IlmInteractiveCharacterController
    };
}

```

---

## 2.5 Usage Documentation (`README.md`)

```markdown
# ilm_3d_pixar_character 🚀

A lightweight, framework-agnostic JavaScript library for rendering and animating modular 3D-styled Pixar characters in HTML5 with real-time blinking, audio lip-syncing, and cursor gaze-tracking.

## Installation

### Via Script Tag (CDN)
```html
<script src="[https://unpkg.com/ilm_3d_pixar_character/dist/ilm_3d_pixar_character.js](https://unpkg.com/ilm_3d_pixar_character/dist/ilm_3d_pixar_character.js)"></script>

```

### Via NPM

```bash
npm install ilm_3d_pixar_character

```

---

## Quick Start Guide

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Pixar Character Demo</title>
    <style>
        #character-viewport {
            width: 400px;
            height: 400px;
            position: relative;
            margin: 50px auto;
        }
    </style>
</head>
<body>

    <div id="character-viewport"></div>

    <!-- Optional audio track for automatic lip-sync -->
    <audio id="voice-track" src="./assets/audio/greeting.mp3"></audio>
    <button onclick="document.getElementById('voice-track').play()">Make Character Speak</button>

    <!-- Import Library -->
    <script type="module">
        import { IlmInteractiveCharacterController } from './dist/ilm_3d_pixar_character.js';

        const character = new IlmInteractiveCharacterController({
            container: '#character-viewport',
            audioElement: '#voice-track',
            assetBasePath: './public/assets/pixar-char/',
            enableGazeTracking: true
        });

        // Set initial expression state
        character.setExpression({
            eyes: 'eyes-open',
            eyebrows: 'eyebrows-expression',
            mouth: 'mouth-smile',
            blush: true
        });
    </script>
</body>
</html>

```

---

## License

MIT License. Built for high-performance interactive web experiences.

```

```