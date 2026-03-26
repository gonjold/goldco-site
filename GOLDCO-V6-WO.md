# WO: Gold Co Landing Page v6 — Sharp Logo, Visible Form

## Pre-flight
```
cd ~/goldco-site
cat index.html
```
Read the entire file. You are making surgical changes to three specific things. Do NOT rewrite the file.

## Problem 1: Logo Geometry Looks Like Rounded Tubes
The TorusGeometry creates round cylindrical cross-sections that look like pipes/jewelry. The brand mark is FLAT, SHARP strokes — like the 2D SVG version. We need to replace the 3D geometry approach entirely.

### Fix: Replace all logo geometry with flat extruded arc shapes

Delete ALL existing logo geometry code (the TorusGeometry for C and O arcs, and the ExtrudeGeometry for the G element). Replace with this approach:

**For the C arc and O arc:** Use RingGeometry to create flat band shapes, then modify them to only show partial arcs.

Actually, the cleanest approach: build the arcs as custom ExtrudeGeometry from Shape paths that trace the arc as a flat band.

Here's the exact code to replace the logo construction with:

```javascript
// --- LOGO (flat, sharp strokes — NOT round tubes) ---
var logo = new THREE.Group();
var R = 2.0;

// Helper: create a flat arc band (like SVG stroke but as 3D geometry)
function makeArc(radius, strokeWidth, startAngle, endAngle, depth) {
  var shape = new THREE.Shape();
  var inner = radius - strokeWidth / 2;
  var outer = radius + strokeWidth / 2;
  var steps = 120;
  var angleSpan = endAngle - startAngle;
  
  // Outer edge (forward)
  for (var i = 0; i <= steps; i++) {
    var angle = startAngle + (i / steps) * angleSpan;
    var x = Math.cos(angle) * outer;
    var y = Math.sin(angle) * outer;
    if (i === 0) shape.moveTo(x, y);
    else shape.lineTo(x, y);
  }
  // Inner edge (backward)
  for (var i = steps; i >= 0; i--) {
    var angle = startAngle + (i / steps) * angleSpan;
    shape.lineTo(Math.cos(angle) * inner, Math.sin(angle) * inner);
  }
  shape.closePath();
  
  return new THREE.ExtrudeGeometry(shape, {
    depth: depth,
    bevelEnabled: false
  });
}

// C arc: bold, ~220 degrees
// Gap is on the right side. Arc goes from 50° to 310° (260° but we want 220°)
// 70° gap means endpoints at +70° and -70° from horizontal right
// So arc goes from -70° to 290° = from -1.222rad to 5.061rad
// In THREE terms: startAngle = 70° in radians, sweeping CCW 220°
var cStartAngle = (-70) * Math.PI / 180;  // bottom endpoint
var cEndAngle = (250) * Math.PI / 180;    // top endpoint (going CCW)
var cGeo = makeArc(R, 0.28, cStartAngle, cEndAngle, 0.06);
var cMesh = new THREE.Mesh(cGeo, matC);
cMesh.position.z = -0.03;
logo.add(cMesh);

// O arc: thin, completes the remaining ~140°
var oGeo = makeArc(R, 0.06, (250) * Math.PI / 180, (290) * Math.PI / 180, 0.03);
var oMesh = new THREE.Mesh(oGeo, matO);
oMesh.position.z = -0.015;
logo.add(oMesh);

// G element: ┐ shape — flat extruded box shapes
var gDepth = 0.06;
var gThick = 0.11;  // stroke thickness
var hLen = R * 0.35;
var vLen = R * 0.32;
var endX = R * Math.cos(70 * Math.PI / 180);

// Horizontal bar — sits on center diameter (y=0)
var hBar = new THREE.Mesh(
  new THREE.BoxGeometry(hLen, gThick, gDepth),
  matG
);
hBar.position.set(endX - hLen / 2, 0, 0);
logo.add(hBar);

// Vertical bar — drops down from right end of horizontal
var vBar = new THREE.Mesh(
  new THREE.BoxGeometry(gThick, vLen, gDepth),
  matG
);
vBar.position.set(endX, -vLen / 2, 0);
logo.add(vBar);

scene.add(logo);
```

**Key difference:** The arcs are now FLAT BANDS (like SVG strokes rendered as thin 3D shapes), not round tubes. The G element uses BoxGeometry — perfectly sharp edges. The whole mark looks like the 2D SVG version but with just enough depth (0.06 units) to catch light and cast reflections as it rotates.

**IMPORTANT:** Make sure the materials (matC, matO, matG) are still the same MeshStandardMaterial with gold color, high metalness, low roughness. Don't change those.

## Problem 2: Text Says "building something" — Should Say "still loading"

Find the HTML element with text "building something" and change it to "still loading".

Also find any JavaScript string that sets or references "building something" and change those too.

## Problem 3: Form Is Too Dark / Invisible

Find the CSS for form input borders and placeholder colors. Change:
- Input/textarea border-bottom from `rgba(255,255,255,0.08)` to `rgba(255,255,255,0.15)` 
- Placeholder color from `rgba(255,255,255,0.15)` to `rgba(255,255,255,0.25)`
- The email link at bottom from `rgba(255,255,255,0.1)` to `rgba(255,255,255,0.18)`

## Do NOT:
- Rewrite the entire file
- Remove mouse parallax, particle systems, camera transitions, or any other v5 features
- Change camera position, particle counts, or lighting
- Add any new dependencies
- Touch anything not mentioned in this WO

## After changes:
```
git add -A && git commit -m "v6: flat sharp logo geometry, still loading text, visible form" && git push origin main
```
