# WO: Gold Co Landing Page v4 — Vastness Fix

## Pre-flight
```
cd ~/goldco-site
cat index.html
```

## What's Wrong
The logo fills almost the entire screen. It needs to feel like a small, precious object floating in an infinite dark universe — like a single planet in space. The user should feel the VASTNESS around it. The logo should occupy maybe 30% of the viewport, surrounded by darkness and gold particle dust stretching out in every direction.

## Changes — Surgical, Do NOT Rewrite The Entire File

### 1. Camera — Pull Way Back
Change camera starting position from z=5 to **z=12**. This is the single biggest change — it makes the logo much smaller on screen, surrounded by space.

Find the camera position initialization and change z to 12.

Also update the idle Z breathing to orbit around 12:
```
camera.position.z = 12 + Math.sin(t * 0.25) * 0.2;
```

Update mouse parallax range — camera X max ±1.2 (wider range since we're further back), Y max ±0.8.

### 2. Particle Layer 1 (close particles) — Spread Wider
Change the spawn radius from 6-12 to **4-20**. More particles should be between camera and logo, and around the edges of the viewport. The user should see gold dust scattered across the entire field of view, not just clustered near the logo.

### 3. Particle Layer 2 (deep dust) — Make More Visible
Increase the deep dust volume from 80x80x120 to **120x120x160**. Increase max alpha from 0.2 to **0.3** — they should be faintly but clearly visible, like distant stars.

### 4. Add a Third Particle Layer — Foreground Floaters (50 particles)
Create 50 large, very close particles that float between the camera and the logo. These should be:
- Spread in a volume from z=3 to z=11 (between camera at z=12 and logo at z=0)
- X/Y spread: ±15
- Size: 3-8 (large, soft, out of focus feeling)
- Alpha: 0.05-0.15 (very subtle, almost like lens dust or bokeh)
- These create depth by existing in the foreground — some will drift across the viewport slowly
- Same gold color, same shader, same additive blending

### 5. Camera Transition — Adjust for New Distance
Update the transition bezier start point from z=5 to z=12:
- camStart z: **12**
- cp1: (2.5, 1.2, -2)
- cp2: (-2, -1, -32)
- camEnd: (0.3, 0, -55)

Update return path similarly — reversed from z=-55 back to z=12.

### 6. Contact State Camera — Adjust
- Camera z position in contact state: **-55** (was -50)
- lookAt target: (0, 0, -60)

### 7. "Gold Co" Wordmark — Move Below Logo
The wordmark margin-top should be **300px** (was 260px) to account for the smaller logo. It needs clear separation below the mark.

### 8. Fog — Reduce or Remove
If there's scene fog, reduce it significantly or remove it. The vast dark space should be visible, not fogged out. The deep particles should be visible far into the distance.

If using FogExp2, change density from whatever it is to **0.008** (very light).

### 9. Ambient Light — Slightly Brighter
Bump ambient from 0x060606 to **0x0a0a0a** — just enough that the space doesn't feel like a pure void, more like deep space with the faintest ambient glow.

## Do NOT:
- Rewrite the entire file
- Remove mouse parallax, particle repulsion, or any v3 features  
- Change the logo geometry, materials, or construction
- Change the form, CTA, or text content
- Touch fonts, colors, or brand elements
- Add any new dependencies or files

## After changes:
```
git add -A && git commit -m "v4: vastness - camera pulled back, wider particle field, foreground bokeh layer" && git push origin main
```

Auto-deploys to jgoldco.com.
