# WO: Gold Co Landing Page v3 — Interactive Spatial Experience

## Pre-flight
```
cd ~/goldco-site
cat index.html
```
Read the current file. You're replacing it entirely with a dramatically better version.

## What's Wrong With v2
- The animation is passive — user just watches a spinning logo, nothing interactive
- "building something" is invisible (color too dark)
- "Gold Co" typography and placement are off
- The form is hard to read
- The particle field has no depth or atmosphere
- The logo just spins — boring, no life to it
- There's no spatial feel — it should feel like you're IN a universe, not watching a screensaver
- No mouse interaction at all — this is 2026, the page should react to the user

## Core Philosophy
This page should feel like entering a living, breathing gold universe. The user is NOT a passive viewer — they're inside the space, and it reacts to them. Every mouse movement shifts perspective. Particles flow around the cursor. The logo breathes and responds. It should feel like a luxury brand crossed with a demoscene.

## Technical Stack
- Three.js r128 from cdnjs.cloudflare.com
- Plus Jakarta Sans from Google Fonts
- Single index.html, no build tools, no frameworks
- Must work on desktop and mobile (touch events for mobile)

## The 3D Scene — DETAILED SPECS

### Camera
- Default position: (0, 0, 5)
- **MOUSE PARALLAX:** Camera position responds to mouse movement. As user moves mouse across screen, camera shifts slightly (max ±0.8 on X, ±0.5 on Y). Use lerping for smooth follow (lerp factor 0.03). This creates a living parallax effect — the logo and particles shift as you move the mouse. The entire scene feels reactive and alive.
- Gentle idle breathing on Z axis: 5 + sin(t * 0.25) * 0.15
- Always lookAt(0, 0, 0)
- Mobile: use device orientation (gyroscope) if available for the same parallax effect, fallback to touch drag

### Logo Mark — Three 3D Pieces
All on same circle radius R = 2.0.

**C arc (bold):**
- TorusGeometry(R, 0.14, 48, 150, PI * 1.222) — ~220° sweep
- This is the hero element — should look like a thick, polished gold ring
- rotation.z = PI * 0.5 + PI * 0.611 (positions gap on the right)

**O arc (thin):**
- TorusGeometry(R, 0.04, 24, 80, PI * 0.778) — ~140° completing circle
- rotation.z = PI * 0.5 + PI * 0.611 + PI * 1.222

**G element (┐ shape):**
- Build as ExtrudeGeometry from a Shape path
- Horizontal bar length = R * 0.35, vertical bar length = R * 0.32
- Bar thickness = 0.05 (full width), half-height = 0.04
- Depth: 0.08, bevel: enabled, bevelThickness 0.018, bevelSize 0.018, bevelSegments 4
- Position: horizontal bar at y=0 (center diameter), vertical right edge at x = R * cos(70°)

**All materials — MeshStandardMaterial:**
- color: 0xC9A84C
- metalness: 0.94
- roughness: 0.13
- emissive: 0x4A3510
- emissiveIntensity: 0.3

**Logo animation — NOT just spinning:**
- Very slow Y rotation: t * 0.08 (slower than before — majestic, not frantic)
- Gentle X tilt that RESPONDS TO MOUSE: base sin(t*0.06)*0.1 + mouseY * 0.15
- Z tilt responds to mouse: mouseX * 0.08
- Gentle vertical bob: sin(t * 0.3) * 0.05
- The logo should feel like it's floating and lazily turning, while also reacting to where the user is looking

### Lighting — Cinematic, 7 lights
```
AmbientLight(0x060606, 0.4)                         // barely there — scene is DARK
PointLight(0xC9A84C, 5, 30)   pos(3, 2.5, 5)       // key — front right
PointLight(0xFFD700, 2.5, 22) pos(-3, 3, 4)         // warm fill — front left
PointLight(0xC9A84C, 3, 16)   pos(-4, -1, 1)        // rim — back left edge highlight
PointLight(0x8B6914, 2, 22)   pos(4, -3, 2)         // rim — back right
PointLight(0xFFD700, 1.2, 28) pos(0, 6, 3)          // top — crown highlight
PointLight(0xC9A84C, 2, 18)   pos(0, -1, -4)        // back — creates halo/glow behind logo
PointLight(0xC9A84C, 1, 15)   pos(0, 0, 8)          // front fill — near camera
```
Key and fill lights should subtly shift position based on mouse (lerped, ±0.5 range) to create dynamic reflections on the gold surface as user moves mouse.

### Particle System — Two Layers

**Layer 1: Close atmospheric particles (300 particles)**
- Spread in a sphere of radius 6-12 around origin
- Size: 1.5 - 4.0
- Alpha: up to 0.6 for nearby ones
- These are the visible, warm, glowing dots the user sees around the logo
- Gentle drift on all axes using sine/cosine (speed 0.003-0.008)

**Layer 2: Deep space dust (800 particles)**
- Spread in a huge volume: 80x80x120 units
- Size: 0.3 - 1.5
- Alpha: 0.02 - 0.2 (very faint, atmospheric)
- These create depth — tiny points far in the background, like stars

**MOUSE REPULSION on Layer 1:**
- Track mouse position and project into 3D space using raycasting onto an invisible plane at z=0
- Particles within radius 2.0 of the mouse intersection point get gently pushed away
- Push force = (2.0 - distance) * 0.02 — gentle, not violent
- Particles slowly return to their original drift after mouse moves away
- This creates the effect of the user "pushing" through the gold dust

**Both layers use ShaderMaterial:**
```glsl
// Vertex
attribute float size;
varying float vAlpha;
void main() {
  vec4 mv = modelViewMatrix * vec4(position, 1.0);
  float d = -mv.z;
  vAlpha = clamp(1.0 - d / 40.0, 0.03, 0.6);
  gl_PointSize = size * (6.0 / max(d, 0.5));
  gl_Position = projectionMatrix * mv;
}

// Fragment
varying float vAlpha;
void main() {
  float d = length(gl_PointCoord - vec2(0.5));
  if (d > 0.5) discard;
  float a = smoothstep(0.5, 0.05, d) * vAlpha;
  gl_FragColor = vec4(0.79, 0.66, 0.30, a);
}
```
- transparent: true, depthWrite: false, blending: AdditiveBlending

### Renderer
- setClearColor(0x020202)
- toneMapping: ACESFilmicToneMapping
- toneMappingExposure: 1.5
- antialias: true
- pixelRatio: min(devicePixelRatio, 2)

### Mouse Tracking (critical — makes the whole thing feel alive)
```javascript
const mouse = { x: 0, y: 0 };
const smoothMouse = { x: 0, y: 0 };
document.addEventListener('mousemove', (e) => {
  mouse.x = (e.clientX / window.innerWidth) * 2 - 1;
  mouse.y = -(e.clientY / window.innerHeight) * 2 + 1;
});
// In render loop:
smoothMouse.x += (mouse.x - smoothMouse.x) * 0.03;
smoothMouse.y += (mouse.y - smoothMouse.y) * 0.03;
```
Use smoothMouse for all reactive elements (camera, logo tilt, lights).

For mouse-particle repulsion, use THREE.Raycaster:
```javascript
const raycaster = new THREE.Raycaster();
const mousePlane = new THREE.Plane(new THREE.Vector3(0, 0, 1), 0);
const mouseWorld = new THREE.Vector3();
// In render loop:
raycaster.setFromCamera(new THREE.Vector2(mouse.x, mouse.y), camera);
raycaster.ray.intersectPlane(mousePlane, mouseWorld);
```

## UI Overlay — HTML/CSS

### Typography and Layout
All text uses Plus Jakarta Sans. The entire overlay is a flex column, centered.

**"Gold Co" wordmark:**
- font-size: 30px
- font-weight: 600
- letter-spacing: 6px
- color: rgba(255,255,255,0.9)
- Position: margin-top: 260px below center (pushes it well below the logo)
- Appears at 2.0s with fade up (opacity 0→0.9, translateY 15px→0, duration 1.8s)

**"building something" text:**
- font-size: 10px
- font-weight: 400
- letter-spacing: 5px
- color: rgba(255,255,255,0.12) — THIS IS THE KEY CHANGE. It was #1a1a1a before which is invisible on dark bg. Use white at 12% opacity instead — just barely visible, mysterious, but READABLE.
- text-transform: lowercase
- position: absolute, bottom: 115px, centered
- Appears at 3.5s with slow fade (opacity transition 3s)

**"start a project" CTA:**
- font-size: 11px
- font-weight: 400
- letter-spacing: 8px
- color: rgba(255,255,255,0.15) — same approach, barely visible but findable
- text-transform: lowercase
- position: absolute, bottom: 48px, centered
- Appears at 4.0s
- HOVER: color transitions to #C9A84C (gold), letter-spacing expands to 14px over 0.9s with cubic-bezier(0.23,1,0.32,1)
- Gold gradient line beneath: width 0 → 100%, height 0.5px, background linear-gradient(90deg, transparent, #C9A84C, transparent)
- pointer-events: auto on the CTA zone, everything else pointer-events: none

### Camera Transition (on CTA click)
Duration: 3500ms (slightly longer for more travel feel).

**Path:** Cubic bezier from (0,0,5) to (0.3,0,-50).
- Control points: cp1(2, 1, -8), cp2(-1.5, -0.8, -32)
- Add sine wobble: x += sin(progress * PI * 3) * 0.4 * (1 - progress), y += sin(progress * PI * 2.2) * 0.25 * (1 - progress)
- Easing: easeInOutQuart

**During transition:**
- Hero UI fades out in 0.5s
- Camera flies THROUGH the particle field — particles streak past naturally
- The logo shrinks into the distance behind
- Mouse parallax continues during flight (adds to the organic feel)
- Camera lookAt follows a point slightly ahead on the bezier curve

**Arrival:**
- Camera settles at z=-50 with very gentle drift
- sin(t*0.03)*0.2 on X, cos(t*0.025)*0.12 on Y
- lookAt(0, 0, -55)

### Contact Form — In The Void
Appears 2.2s after camera arrives (CSS transition-delay).

**Container:** max-width 400px, centered, no background, no border, no card. Just floating fields.

**Inputs (3 fields: name, email, textarea):**
- width: 100%
- padding: 18px 0
- background: none
- border: none EXCEPT border-bottom: 1px solid rgba(255,255,255,0.06)
- color: #fff
- font-size: 15px, weight 300, letter-spacing 0.5px
- placeholder color: rgba(255,255,255,0.12)
- On focus: border-bottom-color #C9A84C with 0.5s transition
- IMPORTANT: these must be LEGIBLE. The current version is too dark.

**"send" button:**
- color: #C9A84C
- font-size: 11px, weight 500, letter-spacing 5px, lowercase
- No background, no border
- Animated arrow: 24px line with chevron, extends to 36px on hover
- On hover: gap increases, letter-spacing grows

**"jon@jgoldco.com" link:**
- Below the form, margin-top 48px
- font-size: 10px, color rgba(255,255,255,0.08), letter-spacing 3px
- On hover: color #C9A84C

**"back" link:**
- position fixed, top 28px, left 28px
- font-size: 10px, color rgba(255,255,255,0.1), letter-spacing 3px, lowercase
- On hover: color #C9A84C
- Appears with 2.5s delay after form appears

**Form submit:** Opens mailto:jon@jgoldco.com with form contents. Shows "sent" / "we'll be in touch" success state.

### Return Transition (clicking "back")
- Same 3500ms duration, reversed bezier path
- Camera flies back from z=-50 to z=5
- On arrival: hero UI fades back in

### Responsive
```css
@media (max-width: 600px) {
  .wordmark { font-size: 22px; margin-top: 180px; letter-spacing: 4px; }
  .form-void { max-width: 320px; }
  .status, .cta-text { font-size: 9px; }
}
```
Mobile: disable mouse parallax, use simple touch to rotate logo slightly.

### Meta Tags
```html
<title>Gold Co</title>
<meta name="description" content="Gold Co — websites, marketing, and digital strategy for local businesses.">
<meta property="og:title" content="Gold Co">
<meta property="og:description" content="Websites, marketing, and digital strategy for local businesses.">
<meta property="og:type" content="website">
<meta property="og:url" content="https://jgoldco.com">
```

Keep the existing SVG favicon (GCO mark data URI).

## Summary of What Makes v3 Different From v2
1. **Mouse parallax** — camera, logo tilt, and lights all respond to mouse position
2. **Particle repulsion** — particles push away from cursor, user "touches" the space
3. **Two particle layers** — close atmosphere + deep space dust for real depth
4. **Slower, more majestic logo rotation** that responds to mouse
5. **Legible text** — "building something" and "start a project" use white at low opacity instead of dark gray
6. **Better typography** — larger wordmark, more letter-spacing, better vertical positioning
7. **Longer transition** (3.5s) with more dramatic bezier curve and wobble
8. **Legible form** — input borders and placeholders visible (white at low opacity, not invisible)
9. **Dynamic lighting** — lights shift with mouse for living reflections on gold
10. **Overall feel** — you're INSIDE the universe, not watching it through glass

## After changes:
```
git add -A && git commit -m "v3: interactive spatial experience - mouse parallax, particle repulsion, legible UI" && git push origin main
```

Auto-deploys to jgoldco.com via Cloudflare Pages.

## Do NOT:
- Add any build tools, package.json, or frameworks
- Create multiple files — everything in one index.html
- Remove any existing functionality
- Touch any other repos or projects
- Use any external assets or images
