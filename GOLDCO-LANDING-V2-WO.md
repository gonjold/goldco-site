# WO: Rebuild Gold Co Landing Page — v2

## Pre-flight
```
cd ~/goldco-site
cat index.html
```
Read the current file. You're rebuilding it in place — same file, same repo, dramatically better.

## Context
The current page is live at goldco-site.pages.dev via Cloudflare Pages. It auto-deploys from this repo on push. The page has a 3D GCO logo mark built with Three.js but it looks underwhelming. We need to make it cinematic and impactful.

## What's Wrong With v1
- Torus arcs are too thin — look like wire, not a premium gold ring
- The G element (┐ shape) is tiny and hard to see
- No visible particle field — the space feels empty and dead
- Lighting is flat — needs dramatic rim lighting and glow
- Camera is too far back — need to be closer and more intimate
- "Gold Co" text overlaps with the mark — needs to be clearly below
- Overall feels like a tech demo, not a luxury brand landing page

## What to Build (single index.html, no frameworks, no build tools)

### Dependencies
- Three.js r128 from cdnjs.cloudflare.com
- Plus Jakarta Sans from Google Fonts

### The 3D Scene

**Logo Mark — three separate 3D pieces, all gold metallic:**
1. **C arc:** TorusGeometry, radius 2.0, tube radius 0.14 (thick, substantial). Sweeps ~220 degrees. This should look like a chunky gold ring, not a wire.
2. **O arc:** TorusGeometry, same radius 2.0, tube radius 0.04 (visibly thinner but not invisible). Completes the remaining ~140 degrees.
3. **G element:** Extruded ┐ shape floating inside the circle. The horizontal bar sits on the center diameter (y=0). The vertical bar's x-coordinate aligns with the C arc endpoints. Make it substantial — bar thickness ~0.05 units, horizontal length = 35% of radius, vertical length = 32% of radius. Use ExtrudeGeometry with bevel for rounded edges.

**Materials — all pieces:**
- MeshStandardMaterial
- color: 0xC9A84C
- metalness: 0.92-0.95
- roughness: 0.12-0.20
- emissive: 0x4A3510, emissiveIntensity: 0.25-0.35
- These should look like polished gold jewelry, not matte plastic

**Logo rotation:**
- Slow continuous Y rotation: t * 0.12
- Gentle X oscillation: sin(t * 0.07) * 0.15
- Very subtle Z: sin(t * 0.1) * 0.04
- Gentle vertical bob: sin(t * 0.35) * 0.06

**Lighting — dramatic, cinematic, 6+ lights:**
- Ambient: very dim (0x080808, intensity 0.5) — the scene should be DARK
- Key light: warm gold PointLight (0xC9A84C, intensity 4) positioned front-right-above
- Second key: slightly different gold (0xFFD700, intensity 2) positioned front-left-above
- Rim light 1: gold (0xC9A84C, intensity 2.5) positioned back-left — creates edge highlight
- Rim light 2: dark gold (0x8B6914, intensity 1.5) positioned back-right
- Top light: bright gold (0xFFD700, intensity 1) directly above
- Back light: gold (0xC9A84C, intensity 1.5) behind the logo — creates halo effect
- Lights should subtly follow camera position during transitions

**Particle field — 1000+ particles:**
- Spread across a large volume (60x60x80 units)
- Use ShaderMaterial with:
  - Vertex shader: size attenuation based on distance, alpha fades with distance
  - Fragment shader: soft circle (discard outside 0.5 radius), smooth alpha falloff
  - Color: warm gold (0.79, 0.66, 0.30)
  - Max alpha: 0.5 for nearby particles
  - Additive blending, no depth write
- Particles should gently drift (sine/cosine oscillation on all axes)
- They should be clearly visible — not invisible dust

**Camera:**
- Default position: (0, 0, 5.5) — closer than v1
- Gentle idle orbit: small sine/cosine movement on X and Y, slight Z breathing
- Always lookAt(0, 0, 0) in idle state

**Renderer settings:**
- setClearColor(0x020202)
- toneMapping: ACESFilmicToneMapping
- toneMappingExposure: 1.4
- antialias: true

**Post-processing glow (optional but ideal):**
If you can add UnrealBloomPass from Three.js examples, do it:
- Import EffectComposer, RenderPass, UnrealBloomPass from cdnjs
- Bloom strength: 0.4, radius: 0.6, threshold: 0.8
- This will give the gold edges a cinematic glow
- If the imports are too complex, skip it — the lighting alone should be sufficient

### UI Overlay (HTML/CSS on top of canvas)

**Reveal sequence (timed with CSS transitions):**
1. 1.8s: "Gold Co" fades in — font-size 28px, weight 600, letter-spacing 5px, positioned 240px below center (margin-top on flex column). Opacity to 0.9.
2. 3.0s: "building something" appears at bottom — font-size 9px, letter-spacing 6px, color #1a1a1a (barely visible). Position: absolute bottom 110px.
3. 3.5s: "start a project" appears at very bottom — font-size 10px, letter-spacing 8px, color #1f1f1f.

**"start a project" hover effect:**
- On hover: color transitions to #C9A84C, letter-spacing expands to 13px
- A thin gold gradient line (height 0.5px) grows from 0 to full width beneath it
- Use cubic-bezier(0.23, 1, 0.32, 1) for smooth spring-like easing

**Click "start a project" → camera transition:**
- Duration: 3200ms
- Camera flies from (0,0,5.5) to (0,0,-40) along a cubic bezier path
- Control points create a curve, NOT a straight line: cp1(1.5, 0.8, -5), cp2(-1, -0.5, -25)
- Add sine wave wobble during flight: x += sin(progress * PI * 2.5) * 0.3 * (1-progress), y += sin(progress * PI * 1.8) * 0.2 * (1-progress)
- Use easeInOutQuart easing
- Camera looks ahead on the path (sample bezier at progress + 0.05)
- During transition: hero UI elements fade out quickly (0.4-0.6s)
- Particles stream past during the flight — this happens naturally since camera moves through the particle field

**Contact form — appears after camera arrives:**
- Delay appearance until transition completes (use CSS transition-delay: 2.2s)
- No card, no container, no background — just form fields floating in black void
- Inputs: border-bottom only (1px solid #111), no other borders. On focus: border-bottom-color #C9A84C
- Placeholder color: #1a1a1a (nearly invisible until you focus)
- Fields: name, email, textarea for message
- Submit: "send" text + animated arrow line. On hover: gap increases, arrow extends, letter-spacing grows
- Below form: "jon@jgoldco.com" as a mailto link, very dim (#191919), lights up gold on hover
- "back" link: top-left corner, very dim, appears after form loads. Clicking it reverses the camera transition back to the logo.

**Form submit action:**
Opens mailto:jon@jgoldco.com with the form contents as subject/body. Then shows "sent" / "we'll be in touch" in place of the form.

**Contact state camera:**
- Subtle drift at z=-40: x = sin(t*0.04)*0.15, y = cos(t*0.03)*0.1
- lookAt(0, 0, -45)

**Return transition (clicking "back"):**
- Same duration (3200ms), reversed bezier path
- After arriving back, restore hero UI elements

### Responsive
- On mobile (max-width 600px): wordmark font-size 20px, form max-width 320px

### Meta
- Title: "Gold Co"
- Description meta tag: "Gold Co — websites, marketing, and digital strategy for local businesses."
- OG tags for title and description
- Favicon: inline SVG data URI of the GCO mark (already in current file, keep it)

## After changes:
```
git add -A && git commit -m "v2: dramatic 3D rebuild - thicker geometry, cinematic lighting, particles, glow" && git push origin main
```

Cloudflare will auto-deploy. Verify at goldco-site.pages.dev.

## Do NOT:
- Add any build tools, package.json, or frameworks
- Create multiple files — everything in one index.html
- Change the fundamental structure (Three.js + HTML overlay + contact form)
- Remove any functionality that exists in v1
- Touch any other repos or projects
