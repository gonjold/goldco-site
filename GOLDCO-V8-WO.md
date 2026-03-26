# WO: Fix G Corner Joint + Gold Co Text — v8

## Pre-flight
```
cd ~/goldco-site
cat index.html
```

## Three Fixes — Surgical Only

### Fix 1: G Element Corner Joint
The horizontal bar and vertical bar don't connect properly at the corner. The problem is both bars are centered on their own axes so there's a gap at the joint.

Find the hBar and vBar position code. The current code positions them independently. Fix by making the vertical bar START at the horizontal bar's center line (y=0) instead of below it:

Find this (or similar):
```javascript
vBar.position.set(endX, -vLen / 2, 0);
```

Replace with:
```javascript
vBar.position.set(endX, -(vLen / 2) + (gThick / 2), 0);
```

This shifts the vertical bar UP by half the stroke thickness so it overlaps with the horizontal bar at the corner — creating a clean connected ┐ joint.

Also make the vertical bar slightly longer to compensate:
Find the vBar BoxGeometry creation line and change the Y dimension from `vLen` to `vLen + gThick`:
```javascript
var vBar = new THREE.Mesh(
  new THREE.BoxGeometry(gThick, vLen + gThick, gDepth),
  matG
);
vBar.position.set(endX, -(vLen / 2), 0);
```

Actually the simplest fix — replace BOTH the hBar and vBar blocks entirely with:

```javascript
// Horizontal bar of ┐
var hBar = new THREE.Mesh(
  new THREE.BoxGeometry(hLen, gThick, gDepth),
  matG
);
hBar.position.set(endX - hLen / 2, 0, 0);
logo.add(hBar);

// Vertical bar of ┐ — top edge flush with top edge of horizontal bar
var vBar = new THREE.Mesh(
  new THREE.BoxGeometry(gThick, vLen, gDepth),
  matG
);
vBar.position.set(endX, -(vLen / 2) + (gThick / 2), 0);
logo.add(vBar);
```

This makes the vertical bar's top edge align with the horizontal bar's top edge, creating a perfect ┐ corner.

### Fix 2: "Gold Co" → "GOLD CO" 
Find the HTML text "Gold Co" in the wordmark element and change it to "GOLD CO".

### Fix 3: Move Wordmark Below the Circle
The wordmark overlaps with the bottom of the circle. Find the CSS for the wordmark class (likely `.wm`) and change `margin-top` from whatever it currently is to `340px`. This pushes it well below the logo.

## Do NOT:
- Touch anything else — no particles, camera, lights, transitions, form, materials
- Rewrite the entire file
- Change the arc geometry

## After changes:
```
git add -A && git commit -m "v8: fix G corner joint, GOLD CO uppercase, text below circle" && git push origin main
```
