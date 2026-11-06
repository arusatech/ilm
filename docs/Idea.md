# ILM (Interactive Live Motion / News-Reader Tutorial) Application

## 1. Executive Summary

The **ILM App** is a lightweight, single-page tutorial studio. It combines one whiteboard that grows vertically and horizontally with a floating news-reader portrait. The portrait is built entirely in the browser: the presenter uploads a passport photo, the page styles it as a Ghibli-like tutor, and live lip-sync plus expression come from the microphone and the webcam. No image or video is sent to a server.

By shifting image processing to the client, the experience has no server dependency and runs inside the user's browser.

---

## 2. Core Architecture & System Goals

* **Zero server:** Upload, stylization, face tracking, lip-sync, drawing, and recording all run in the browser. There is no image API and no video stream to host.
* **Low-bandwidth presentation:** The learner sees a small animated portrait, not a raw desk-camera feed.
* **Single-page growing board:** One whiteboard surface that starts at the window and extends downward and to the right as drawings, shapes, text, and images pass the current edges. The presenter scrolls that surface with the pointer or by touch.
* **News-reader frame:** The tutor stays in a draggable passport-sized (`3:4`) picture-in-picture over the board.

The previous still-photo preset (guides for head, eyes, ears, nose, mouth, teeth, and neck, then a hand-drawn mouth and neck wobble) is retired. It does not read like a person talking. The replacement is a client-side Ghibli avatar with live lip-sync.

---

## 3. Functional Requirements

### 3.1. The Growing Whiteboard

* **Open board:** One continuous surface. Content is drawings, shapes, text, and images on that surface, with no topic blocks and no Next, Previous, or arrow-key paging.
* **Growth in both directions:** The board starts at the window size. When a mark or image extends past the right or bottom edge, the board grows that way so the new content stays on the surface.
* **Scrolling:** The presenter moves around the enlarged board with the pointer or by touch.
* **Whiteboard drawing layer:**
  * A transparent HTML5 Canvas that stays aligned with the growing board.
  * Tools: Pen, Highlighter, Eraser, and Color/Stroke width pickers.
* **Image library:** An uploaded board image can have its background removed in the browser, then be placed and resized from a corner.

### 3.2. Client-Side Ghibli Avatar and Live Lip-Sync

Heavy models such as Stable Diffusion are not the path. They do not run well on a normal laptop or phone browser. The avatar uses one of these two client-side approaches.

* **Approach A — Pre-generated sprite sheets (the one to build):**
  When the presenter uploads a passport photo, draw it on an HTML5 canvas, apply a watercolor look (soft edges, posterization, muted color), and composite it into local Ghibli-style hair and clothing layers. Mouth, eyes, and a small head tilt switch or blend sprite frames. No network model is required for the look.
* **Approach B — In-browser style transfer (later, optional):**
  Transformers.js or ONNX Runtime Web can run a small image-to-image model on the user's GPU through WebGPU. This is not required for the first version.

What the portrait is allowed to do, so it still reads as a tutor and not a cartoon body:

* Eyes, nose, ears, lips, and a little neck or head tilt.
* Lips and jaw follow speech.
* Eyes blink and follow the live face.
* Hair, clothes, and the news-reader frame stay still.

#### Libraries

| Job | Library |
| --- | --- |
| Image upload | Native `FileReader` and HTML5 Canvas |
| Face and expression | `@mediapipe/tasks-vision` (WebAssembly / WebGPU). Tracks lips, eyes, and head tilt from the webcam, on device. |
| Audio lip-sync | Web Audio API `AnalyserNode`. Reads the microphone or speech. No extra package. |
| Sprite rendering | `pixi.js`, so closed-mouth and open-mouth frames can swap without repainting the whole page. |

```bash
npm install @mediapipe/tasks-vision pixi.js
```

#### Step 1 — Passport upload

The presenter drops or chooses a front-facing photo. The file never leaves the browser.

```javascript
const uploadInput = document.getElementById("passportUpload");
uploadInput.addEventListener("change", (event) => {
  const file = event.target.files[0];
  const reader = new FileReader();
  reader.onload = (loadEvent) => {
    const img = new Image();
    img.src = loadEvent.target.result;
    img.onload = () => processUserAvatar(img);
  };
  reader.readAsDataURL(file);
});
```

#### Step 2 — Stylize and layer the face

Draw a local tutor body, clip the uploaded face into the head, then grade it toward the painted look.

```javascript
function processUserAvatar(userImg) {
  const canvas = document.getElementById("avatarCanvas");
  const ctx = canvas.getContext("2d");
  const ghibliBody = new Image();
  ghibliBody.src = "assets/ghibli_tutor_body.png";
  ghibliBody.onload = () => {
    ctx.drawImage(ghibliBody, 0, 0, canvas.width, canvas.height);
    ctx.save();
    ctx.beginPath();
    ctx.arc(200, 150, 60, 0, Math.PI * 2);
    ctx.clip();
    ctx.drawImage(userImg, 140, 90, 120, 120);
    ctx.restore();
    applyGhibliFilters(ctx, canvas);
  };
}
```

`applyGhibliFilters` is a canvas or WebGL pass: soft blur, slight posterization, and a warm watercolor grade. Hair and clothing stay as static local assets. Only the face window is the uploaded photo.

#### Step 3 — Webcam expression and audio lip-sync

MediaPipe Face Landmarker reads the webcam locally and produces blendshapes (mouth open, jaw, blink, head tilt). `AnalyserNode` supplies the speech envelope so the mouth still moves with the voice when the face mesh is quiet. Pixi swaps or blends the mouth and eye sprites from those two signals. The webcam picture itself is not shown in the news-reader frame.

```javascript
import { FaceLandmarker, FilesetResolver } from "@mediapipe/tasks-vision";

async function setupWebcamTracking() {
  const vision = await FilesetResolver.forVisionTasks(
    "https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@latest/wasm"
  );
  const faceLandmarker = await FaceLandmarker.createFromOptions(vision, {
    baseOptions: {
      modelAssetPath: "https://storage.googleapis.com/mediapipe-models/face_landmarker/face_landmarker/float16/1/face_landmarker.task",
      delegate: "GPU"
    },
    outputFaceBlendshapes: true,
    runningMode: "VIDEO"
  });
  // Read webcam frames locally and drive mouth, eye, and neck sprites.
}
```

The microphone is analysed in the page and is not played back through the speakers. Recording the whiteboard composites the Pixi portrait into the passport frame and, when the presenter allows it, includes that same microphone track.

#### Floating frame

* Passport ratio `3:4`.
* Drag with mouse or touch anywhere over the board.
* The frame shows the styled avatar, never the raw webcam.

---

## 4. Non-Functional Requirements

* **Performance:** Target a stable 20–30 FPS on mobile processors using hardware-accelerated WebGL / Canvas rendering.
* **Bandwidth:** Tracking, stylization, and playback stay on the device. After the page and the local avatar assets load, presentation mode should stay under about 50 kbps.
* **Cross-platform:** Usable in mobile and desktop Chrome, Safari, and Firefox. WebGPU style transfer (Approach B) is optional and must not be required to open the board.
* **Privacy:** The webcam, microphone, and photo are processed on the device. Raw video is not uploaded. Recording, when the presenter starts it, is a local file.

---

## 5. Proposed Technology Stack

| Layer | Technology | Function |
| --- | --- | --- |
| **Frontend layout** | HTML5, CSS | Single page. The board grows vertically and horizontally. |
| **Board** | Pointer Events, HTML5 Canvas | Drawing, scrolling, images, and recording. |
| **Avatar look** | Canvas filters plus local sprite layers | Passport photo clipped into a Ghibli-style tutor. Approach B (Transformers.js or ONNX Runtime Web) is optional later. |
| **Face and expression** | `@mediapipe/tasks-vision` | On-device lips, eyes, and head tilt from the webcam. |
| **Lip-sync** | Web Audio `AnalyserNode` | Speech level drives the mouth with the face mesh. |
| **Avatar rendering** | `pixi.js` | Swaps closed-mouth, open-mouth, and blink frames inside the passport frame. |
