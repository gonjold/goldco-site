# WO: Fix GCO Logo — Flat Geometry, Correct Orientation

## Pre-flight
```
cd ~/goldco-site
cat index.html
```
Read the entire file first. Find ALL code related to logo geometry creation.

## Problem
The logo uses TorusGeometry which creates round tube cross-sections. The brand mark has FLAT, SHARP strokes — like extruded rectangles, not pipes. Also the orientation may be wrong — the gap in the circle must be on the RIGHT side (between roughly 1:30 and 4:30 on a clock face), not the bottom or top.

Reference the uploaded brand mark image: bold C arc sweeps from upper-right around through left and down to lower-right. Thin O arc fills the gap on the right. The ┐ G element sits center-right with its horizontal bar on the exact center line and vertical bar dropping down.

## Task
Delete ALL existing logo geometry code — every line that creates cMesh, oMesh, gMesh, cGeo, oGeo, gGeo, gs (the shape), hBar, vBar, or any TorusGeometry, ExtrudeGeometry, or BoxGeometry used for the logo. Delete from the first logo-related geometry line down to `scene.add(logo)`.

Replace with this EXACT code (paste verbatim, do not modify):

```javascript
// --- LOGO (flat sharp strokes, correct orientation) ---
var logo = new THREE.Group();
var R = 2.0;

// Helper: flat arc band (like SVG stroke as 3D shape)
function makeArc(radius, strokeW, startDeg, endDeg, depth) {
  var sh = new THREE.Shape();
  var inner = radius - strokeW / 2;
  var outer = radius + strokeW / 2;
  var steps = 128;
  var s = startDeg * Math.PI / 180;
  var e = endDeg * Math.PI / 180;
  var span = e - s;
  for (var i = 0; i <= steps; i++) {
    var a = s + (i / steps) * span;
    var x = Math.cos(a) * outer, y = Math.sin(a) * outer;
    if (i === 0) sh.moveTo(x, y); else sh.lineTo(x, y);
  }
  for (var i = steps; i >= 0; i--) {
    var a = s + (i / steps) * span;
    sh.lineTo(Math.cos(a) * inner, Math.sin(a) * inner);
  }
  sh.closePath();
  return new THREE.ExtrudeGeometry(sh, { depth: depth, bevelEnabled: false });
}

// C arc: bold, from -70° to 250° (sweeps 320° = leaves 40° gap... no)
// We want ~220° of bold arc. Gap is ~140° on the right side.
// Gap from -70° to +70° (centered on 0° which is the right side)
// C arc goes from 70° to 290° (= 220° sweep, CCW through top-left-bottom)
var cGeo = makeArc(R, 0.26, 70, 290, 0.07);
var cMesh = new THREE.Mesh(cGeo, matC);
cMesh.position.z = -0.035;
logo.add(cMesh);

// O arc: thin, from 290° to 430° (= 70° + 360°) — fills the remaining gap on right side
var oGeo = makeArc(R, 0.06, 290, 430, 0.04);
var oMesh = new THREE.Mesh(oGeo, matO);
oMesh.position.z = -0.02;
logo.add(oMesh);

// G element: ┐ shape using BoxGeometry (perfectly sharp edges)
var hLen = R * 0.35;   // horizontal bar length
var vLen = R * 0.32;   // vertical bar drop
var gThick = 0.12;     // stroke thickness (visual weight)
var gDepth = 0.07;     // extrusion depth (same as C arc)

// The G's right edge aligns with C arc endpoints
// C endpoints are at 70° and 290° on circle of radius R
// At 70°: x = R * cos(70°) = 2.0 * 0.342 = 0.684
// Horizontal bar: centered vertically at y = 0 (center diameter)
var endX = R * Math.cos(70 * Math.PI / 180); // ≈ 0.684

var hBar = new THREE.Mesh(
  new THREE.BoxGeometry(hLen, gThick, gDepth),
  matG
);
hBar.position.set(endX - hLen / 2, 0, 0);
logo.add(hBar);

// Vertical bar: drops down from right end of horizontal bar
var vBar = new THREE.Mesh(
  new THREE.BoxGeometry(gThick, vLen, gDepth),
  matG
);
vBar.position.set(endX, -vLen / 2, 0);
logo.add(vBar);

scene.add(logo);
```

## IMPORTANT — Verify These Things After Replacing:
1. The variable `matC`, `matO`, `matG` must still exist before this code runs. Do NOT delete the material definitions.
2. The variable `logo` must be declared as `new THREE.Group()` in this code block (it is).
3. `scene.add(logo)` is at the end of this block (it is).
4. All later references to `logo` (rotation, position in the render loop) should still work since `logo` is the same Group variable name.
5. If there is a `makeArc` function already defined elsewhere, remove the old one — this block defines its own.

## Do NOT:
- Modify ANYTHING outside the logo geometry section
- Touch particles, camera, lights, UI, CSS, HTML, mouse tracking, or transitions
- Change material definitions
- Change the render loop's logo.rotation or logo.position references
- Add any new dependencies
- Rewrite the entire file

## After changes:
```
git add -A && git commit -m "v7: flat sharp logo geometry, correct orientation with gap on right" && git push origin main
```
