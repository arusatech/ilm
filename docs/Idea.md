# ILM (Interactive Live Motion / News-Reader Tutorial) Application

## 1. Executive Summary

The **ILM App** is a lightweight, single-page web-based tutorial studio optimized for low-bandwidth environments. It combines a continuous vertical-flowing presentation whiteboard with a floating, draggable "News Anchor" video feed. Instead of broadcasting high-bandwidth raw video streams from unflattering desk or tabletop camera angles, ILM utilizes client-side face landmark tracking to render an expressive, low-bandwidth avatar/sprite or a cropped studio-framed portrait that replicates the presenter's lip-sync and expressions in real time.

---

## 2. Core Architecture & System Goals

* **Low-Bandwidth Optimization:** Avoids heavy multi-megabit video streaming by transmitting lightweight coordinate matrices or rendering locally via client-side web technologies.
* **Single-Page Continuous Flow:** Replaces multi-slide decks with a continuous vertical document layout that expands infinitely as topics are conveyed.
* **Professional Presentation Layer:** Separates raw camera environments from the viewer experience using a polished "News Anchor" widget overlay.

---

## 3. Functional Requirements

### 3.1. The Vertical Infinite Whiteboard

* **Continuous Scroll Layout:** Presentation content is structured into stacked topic blocks that flow downward seamlessly.
* **Auto-Expanding Canvas:** The canvas height dynamically scales as new cards or diagrams are appended.
* **Navigation Controls:**
* Step-by-step navigation buttons (**Next Block** / **Previous Block**) that execute smooth programmatic scrolling (`scrollIntoView`).
* Keyboard shortcut support (`ArrowDown` / `ArrowUp`).


* **Whiteboard Drawing Layer:**
* Transparent HTML5 Canvas overlay across blocks.
* Tools: Pen, Highlighter, Eraser, and Color/Stroke width pickers.



### 3.2. The "News Reader" Video Feed (Avatar / Retargeted View)

* **Camera Capture & Processing:**
* Accesses the mobile front camera using `navigator.mediaDevices.getUserMedia`.
* Operates locally in the browser to bypass poor table/desk framing angles.


* **Expression & Lip-Sync Tracking:**
* Uses a lightweight browser-based model (e.g., MediaPipe Face Landmarker / WebGL face mesh) to extract key points tracking mouth opening, jaw movement, eye blinking, and micro-expressions.
* Drives a stylized 2D avatar or a cropped, studio-filtered portrait box that mimics the presenter's movements.


* **Draggable & Dockable Floating UI:**
* Rendered inside a passport-sized frame (`3:4` portrait ratio).
* Equipped with touch and mouse pointer event listeners enabling the presenter to drag and drop the video box anywhere across the whiteboard canvas.



---

## 4. Non-Functional Requirements

* **Performance:** Target a stable 20–30 FPS on mobile processors using hardware-accelerated WebGL / Canvas rendering.
* **Bandwidth Footprint:** Minimal network usage (< 50 kbps requirement during presentation mode since tracking calculations run locally on the edge/client).
* **Cross-Platform Compatibility:** Fully responsive layout optimized for mobile browsers (Safari, Chrome, Firefox) and desktop environments.
* **Privacy:** Face mesh analysis is computed completely on-device with zero raw video recording or cloud streaming required by default.

---

## 5. Proposed Technology Stack

| Layer | Technology | Function |
| --- | --- | --- |
| **Frontend Layout** | HTML5, Tailwind CSS | Responsive styling and smooth vertical flex layout. |
| **Animation & Interactivity** | Pointer Events API, HTML5 Canvas | Smooth touch dragging, whiteboarding, and freehand annotation. |
| **Face & Expression Tracking** | MediaPipe Face Landmarker (WASM/WebGL) | Real-time extraction of mouth, lip-sync, and eye coordinates. |
| **Rendering Engine** | 2D Canvas / SVG Sprite Engine | Renders the responsive news-anchor avatar mirroring live facial parameters. |