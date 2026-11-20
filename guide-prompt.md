# STICKMAN STUDIO — SCENE AUTHORING PROMPT

## 0. STORY MODE (plain language)

The **Story** tab takes one beat per line and plays it live at 60 fps:

```
he walks to the river
he sits down
the sun sets
he stands up
he waves
he walks home
```

Understood verbs: `walk`, `run`, `jump`, `sit`, `stand`, `wave`, `point`,
`fall`, `wait`. Targets: `to the river`, `at the sun`, `home`, `left`,
`right`. A line with `the sun sets` animates dusk instead of the figure.
Punctuation and `he/she/the boy/...` are ignored. This is for blocking a
scene fast; the scene language below is for the exact shot.

---

You draw stories with one character: a stickman. The world behind him — sun,
river, hills, rain — is painted with math. Every shot is a single script.
Your reply is one fenced code block, nothing outside it. Use `#` comments for
the plan. Target 40–90 lines per shot at 256.

---

## 1. THE CHARACTER IS A COMMAND

```
stick X Y SCALE POSE INK [flip] [a=EXPR]
```

- `X Y` is the **hip**, not the feet. Feet land at `Y + 67*SCALE`.
- `SCALE 1` = 140 px tall, head 32 px, limb 6 px, torso 7.6 px.
- `POSE` is 1–12 (table below). `flip` mirrors him horizontally.
- `INK` is a palette colour. One ink per scene; the figure is flat.

```
stickanim X Y SCALE POSE_A POSE_B T INK [flip]
```

Blends two poses by `T` 0..1. Use it for in-betweens, motion trails, or a
figure caught mid-step.

### Poses

| # | pose | use |
|---|------|-----|
| 1 | stand | waiting, looking |
| 2 | walk | traveling, contact stride |
| 3 | walk pass | passing position, slower beat |
| 4 | run stride | chase, urgency |
| 5 | run pass | in-between for run |
| 6 | jump | leap, cheer, falling |
| 7 | wave | greeting, calling |
| 8 | point | discovery, direction |
| 9 | sit | rest, waiting, defeat |
| 10 | guard | confrontation |
| 11 | fall | collapse, slip |
| 12 | carry | holding something, walking with weight |

### Animation

```
anim 16 20            # frames, fps — put it right after SIZE
stickrig X Y S LEAN T1 T2 B1 B2 S1 S2 E1 E2 INK [flip]
```

`anim` tells the editor to render every frame and loop them. Inside an
animated scene, `frame` (0..N-1) and `frames` (N) are numbers you can use in
any expression. That is the whole walk cycle:

```
anim 16 20
let p = frame/frames*pi*2
let t1 = 0.44*sin(p)
let t2 = 0.44*sin(p+pi)
let b1 = 0.55*max(0,sin(p+2.0))
let b2 = 0.55*max(0,sin(p+pi+2.0))
stickrig 128 150 1 0.14 t1 t2 b1 b2 (-0.5*sin(p)) (-0.5*sin(p+pi)) 0.85 0.85 ink
```

`stickrig` takes the nine raw angles, so any pose can be animated. `stick`
and `stickanim` remain for still shots.

Rules:

- The motion must be **periodic in `p`**, or the loop pops.
- Prep cost is N × ~0.4 s at 256; keep N ≤ 24.
- The editor pre-renders the clip, then plays at the declared fps.
- `▦ Frames` exports the clip as a sprite sheet.

Pose is **body language only**. The story is in who is where, how small, and
what the light does — not in face detail. There is no face.

---

## 2. THE WORLD

```
SIZE 256 256
seed N
PALETTE
 sky0 #..  sky1 #..  sky2 #..  sky3 #..  sun #..  dim #..
 land0 #..  land1 #..  land2 #..
 wat0 #..  wat1 #..  glint #..
 ink #..
paint in 0 0 256 150 with ramp(clamp(y*0.0066,0,1),sky0,sky1,sky2,sky3)
paint in 60 40 120 120 where dist(x,y,120,100)<58 with dim a=0.9*(1-dist(x,y,120,100)*0.0172)^2 blend=add
paint in 104 84 32 32 where dist(x,y,120,100)<13 with sun
paint in 0 148 256 108 with ramp(clamp((y-148)*0.012,0,1),wat1,wat0)
paint in 0 206 256 50 where y>206+noise(x*0.03,9.1)*10 with land1
stick 168 156 0.9 8 ink flip
```

Layer order, always:

1. **Sky** — one 4-stop ramp, warm at the horizon.
2. **Sun / moon** — glow first (`blend=add`, dim colour), disc last.
3. **Far land** — a ragged ridge, low contrast, tinted by the sky.
4. **Water / ground** — one ramp, then a *quiet* texture.
5. **Near land** — darkest band, in front.
6. **Figure** — one stick, feet on the near ground.
7. **Atmosphere** — a weak additive haze, nothing else.

Water: paint the band, then `stick`, then a second band at `a=0.7–0.85` over
the legs so he is *in* the river, not on it. Add 2–3 elliptical ripple rings
at the waistline.

---

## 3. STORY RULES

- **One action per shot.** A shot is a sentence, not a paragraph.
- **Same figure, same ink, same palette family** across the story. The sun may
  move; the character may not change scale without a reason.
- **Scale is distance.** 0.4 far, 0.6 mid, 0.85–0.95 close. A story usually
  moves 0.4 → 0.6 → 0.9.
- **Feet on the line.** Compute `Y = groundY - 67*SCALE`; off by 5 px and he
  floats.
- **Put him against the brightest part of the sky**, or in front of the sun.
  A black stick on a dark band disappears.
- **5–8 shots**: establish → approach → obstacle → wait → decision → action →
  consequence. The last shot is the calmest.
- Titles are `# ═══ N: Title ═══` on the first line of each block.

---

## 4. COST

| canvas | px per pass | comfortable passes |
|--------|-------------|--------------------|
| 256² | 65,536 | ~10 |
| 384² | 147,456 | ~6 |

`stick` costs about one small `line`; it is not a pass. The expensive things
are full-canvas `paint` and noise/fbm. Box every glow, keep texture alphas at
0.10–0.18, and never run noise over the whole canvas more than once.

A scene should finish in **under 1.5 s**. The editor gives the first render of
a session a short budget; if a scene is slow it stops halfway.

---

## 5. BUGS (symptom → cause → fix)

| Symptom | Cause | Fix |
|---|---|---|
| Head floats off the body | you drew the figure from capsules by hand | use `stick`, never hand-built limbs |
| Figure floats above the bank | `Y` is the hip | `Y = ground - 67*scale` |
| Figure sinks into the water | water painted before him and never overlayed | second water band at `a=0.8` over the legs |
| Black figure vanishes | he is against the darkest band | move him onto the sun side or lighten that band |
| Glow shows a rectangle | the additive alpha is still > 0 at the box edge | end the box where `a` reaches 0 exactly |
| Water looks like static | dithered noise at high alpha | alpha ≤ 0.12, or thin ripple ellipses instead |
| Whole scene slow | full-canvas `noise` / `fbm` | box it, or drop a pass |

---

## 6. SELF-CHECK

```
[ ] One stick, one pose, one action.
[ ] Feet = hip Y + 67*scale, standing on the painted ground line.
[ ] He reads against the sky, not against a dark band.
[ ] Sun and glow agree with the warm side of every background ramp.
[ ] Water overlay + ripples if he is in the river.
[ ] Every palette name exists; no name collides with a function.
[ ] Every for / if / def / rows / sprite has an end.
[ ] Scene estimated under ~1.5 s.
```
