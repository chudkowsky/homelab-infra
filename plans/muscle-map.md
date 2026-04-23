# Muscle Map — Interactive 3D Muscle Explorer

A minimalistic, aesthetic web app with a 3D human body where you click a muscle group and get a description + exercises. Built to run standalone now, embeddable in a larger app later.

---

## Motivation

A clean reference tool for workout planning — no clutter, no ads, no bloated fitness apps. Click a muscle, learn what it does, see what exercises target it. Designed to eventually embed inside a larger fitness or personal dashboard.

---

## UX Design

```
┌──────────────────────────────────────────────────┐
│  [front / back toggle]                           │
│                                                  │
│         3D body — rotatable, dark bg             │
│         muscle groups glow on hover              │
│         click → locks selection                  │
│                                                  │
└──────────────────────────────────────────────────┘
                        │
              muscle clicked
                        │
                        ▼
┌──────────────────────────────────────────────────┐
│  Chest (Pectoralis Major)                        │
│  ─────────────────────────────                   │
│  Primary function: horizontal push,              │
│  shoulder flexion, adduction.                    │
│                                                  │
│  Exercises:                                      │
│  • Bench Press                                   │
│  • Push-up                                       │
│  • Cable Fly                                     │
│  • Dips                                          │
└──────────────────────────────────────────────────┘
```

- Dark background, muted neutral body color, accent color on hover/select (single color, e.g. warm amber or electric blue)
- Side/bottom panel slides in on click — no page navigation
- Minimal typography, no icons clutter
- Keyboard: `Escape` deselects, `F` flips front/back

---

## Muscle Groups (MVP Scope — ~15)

| Group | Muscles covered |
|---|---|
| Chest | Pectoralis major, minor |
| Front Shoulders | Anterior deltoid |
| Side Shoulders | Lateral deltoid |
| Rear Shoulders | Posterior deltoid |
| Biceps | Biceps brachii, brachialis |
| Triceps | Triceps brachii |
| Forearms | Flexors + extensors (merged) |
| Upper Back | Trapezius, rhomboids |
| Lats | Latissimus dorsi |
| Abs | Rectus abdominis, obliques |
| Lower Back | Erector spinae |
| Glutes | Gluteus maximus, medius |
| Quads | Rectus femoris, vastus group |
| Hamstrings | Biceps femoris, semitendinosus |
| Calves | Gastrocnemius, soleus |

---

## Architecture

```
muscle-map/
├── public/
│   └── model/
│       └── body.glb          # 3D model, ~15 named mesh objects
├── src/
│   ├── data/
│   │   └── muscles.ts        # muscle id → { name, description, exercises[] }
│   ├── components/
│   │   ├── Scene.tsx          # R3F Canvas, lighting, camera controls
│   │   ├── BodyModel.tsx      # loads GLB, maps meshes → muscle ids, hover/click
│   │   └── InfoPanel.tsx      # slides in with muscle info
│   ├── App.tsx
│   └── main.tsx
├── index.html
└── vite.config.ts
```

---

## Stack

| Layer | Choice | Why |
|---|---|---|
| Framework | React + TypeScript | clean component model, embeddable later as a package |
| 3D | React Three Fiber + @react-three/drei | idiomatic React wrapper for Three.js |
| Build | Vite | fast, minimal config |
| Styling | CSS Modules or plain CSS | no framework bloat, full control |
| Data | Static TypeScript file | no DB needed for MVP, easy to extend |
| Model | Z-Anatomy (CC BY-SA 4.0) → exported GLB | open source, separate mesh per muscle group |

---

## 3D Model Pipeline

1. Download **Z-Anatomy** Blender file from https://simtk.org/projects/z-anatomy
2. Open in Blender — muscles are organized in named collections
3. Merge individual muscles into ~15 **group meshes** (e.g. all quad muscles → one object named `quads`)
4. Export as **GLB** (binary GLTF) — single file, web-ready
5. Name each mesh object to match keys in `muscles.ts`
6. Optimize: Draco compression, ~2–5 MB target

Blender is the only tool needed. Alternatively, if Blender work is too involved, **Sketchfab** has free CC-licensed low-poly anatomy models where groups are already merged.

---

## Data Format

```typescript
// src/data/muscles.ts
export type MuscleData = {
  name: string
  latin: string
  function: string
  exercises: { name: string; type: 'compound' | 'isolation' }[]
}

export const muscles: Record<string, MuscleData> = {
  chest: {
    name: 'Chest',
    latin: 'Pectoralis Major',
    function: 'Horizontal pushing, shoulder flexion and adduction.',
    exercises: [
      { name: 'Bench Press', type: 'compound' },
      { name: 'Push-up', type: 'compound' },
      { name: 'Cable Fly', type: 'isolation' },
      { name: 'Dips', type: 'compound' },
    ],
  },
  // ...
}
```

---

## Embeddability Plan

To embed in a larger app later:
- Export as a **Web Component** (`<muscle-map />`) or a plain React component with a stable prop API
- All state internal; expose optional `onMuscleSelect(id: string)` callback prop
- No global CSS — scoped styles only
- GLB model served from `/public` or configurable via a `modelUrl` prop
- Zero backend dependency — fully static

---

## Deployment (Homelab)

- Docker Compose with nginx serving static build output
- Domain: `muscles.chudkowsky.com` (or subdomain TBD)
- Port: `8003` (next available per CLAUDE.md)
- Certbot TLS after initial HTTP-only test

---

## Phases

### Phase 1 — Core (MVP)
- [ ] Scaffold Vite + React + R3F project
- [ ] Source and prepare GLB model (Z-Anatomy or Sketchfab)
- [ ] Load model, wire hover highlight + click selection
- [ ] Write `muscles.ts` data for all 15 groups
- [ ] Build `InfoPanel` component
- [ ] Basic styling — dark theme, accent color, panel animation

### Phase 2 — Polish
- [ ] Camera auto-rotate to face selected muscle
- [ ] Front/back toggle or free orbit
- [ ] Smooth hover + selection animations (emissive glow)
- [ ] Mobile touch support
- [ ] Keyboard navigation (Escape, arrow keys)

### Phase 3 — Embed
- [ ] Wrap as Web Component or extract as npm package
- [ ] Add `onMuscleSelect` prop
- [ ] Make model URL and accent color configurable via props
- [ ] Write embedding docs

---

## Open Questions

- **Model source**: Z-Anatomy (best quality, needs Blender work) vs a ready-made low-poly Sketchfab model (faster start, less control)
- **Accent color**: single warm amber vs cool blue — decide once the dark theme is visible
- **Exercise data depth**: for MVP just name + type; later could link to video demos
