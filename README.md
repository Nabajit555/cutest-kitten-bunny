# cutest-kitten-bunny

Entry for [make.fun · cutest-picture-01](https://make.fun/bounty/cutest-picture-01):
a kawaii cream kitten hugging a lavender plush bunny, drawn entirely in code.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out python3 main.py
---

**Content 1 — main.py** (for Link 1; copy the whole thing):

```python
#!/usr/bin/env python3
"""cutest-picture-01 entry: draws a kawaii cream kitten hugging a plush bunny.

Pure Python stdlib. Writes /output/entry.png (1024x1024, <1 MiB).
Deterministic: no randomness, no network, same bytes every run.
"""
import os, struct, zlib
from math import sin, cos, sqrt, atan2

W = H = 1024
AA = 1.4

def lerp(a, b, t): return a + (b - a) * t
def mix(c1, c2, t): return tuple(lerp(c1[i], c2[i], t) for i in range(len(c1)))

def ellipse_field(px, py, cx, cy, rx, ry, rot=0.0):
    if rot:
        dx, dy = px - cx, py - cy
        c, s = cos(rot), sin(rot)
        px = cx + dx * c + dy * s
        py = cy - dx * s + dy * c
    f = sqrt(((px - cx) / rx) ** 2 + ((py - cy) / ry) ** 2) - 1.0
    return f * min(rx, ry)  # approx pixel distance

def circle_field(px, py, cx, cy, r):
    return sqrt((px - cx) ** 2 + (py - cy) ** 2) - r  # already pixels

def capsule_field(px, py, x1, y1, x2, y2, r):
    dx, dy = x2 - x1, y2 - y1
    L2 = dx * dx + dy * dy or 1.0
    t = max(0.0, min(1.0, ((px - x1) * dx + (py - y1) * dy) / L2))
    qx, qy = x1 + t * dx, y1 + t * dy
    return sqrt((px - qx) ** 2 + (py - qy) ** 2) - r

def heart_field(px, py, cx, cy, s):
    x = (px - cx) / s
    y = -(py - cy) / s
    a = x * x + y * y - 1
    return ((a ** 3) - x * x * y * y * y) * s * 0.35  # approx pixel distance

def soft(f):
    return max(0.0, min(1.0, 0.5 - f / AA))

def ring(f, w=2.2, off=1.0):
    """soft outline band just inside a shape edge (f in pixels, negative inside)."""
    return max(0.0, min(1.0, (off + w - abs(f)) / (2 * w)))

def star_field(px, py, cx, cy, r):
    dx, dy = px - cx, py - cy
    ang = atan2(dy, dx)
    d = sqrt(dx * dx + dy * dy)
    u = abs(cos(ang * 2.0))
    rr = r * (0.30 + 0.70 * (u ** 0.55))
    return d - rr

# ---------- palette ----------
BG_TOP = (252, 240, 246)
BG_BOT = (255, 224, 238)
FLOOR = (250, 205, 226)
KITTY = (255, 246, 224)
KITTY_SHADE = (243, 214, 178)
OUTLINE = (214, 160, 130)
WHITE = (255, 255, 255)
EYE = (72, 52, 72)
BLUSH = (255, 170, 184)
MOUTH = (186, 116, 128)
TONGUE = (255, 150, 164)
BUNNY = (226, 208, 250)
BUNNY_SHADE = (196, 172, 232)
BUNNY_OUT = (150, 122, 196)
NOSE = (255, 160, 172)
HEART = (255, 118, 150)
SPARK = (255, 202, 120)

# geometry
KX, KY = 512, 470        # head centre
HRX, HRY = 292, 258      # head radii
TILT = 0.10
BX, BY = 512, 852        # bunny centre

def scene(px, py):
    t = py / H
    col = mix(BG_TOP, BG_BOT, t)
    g = ellipse_field(px, py, 512, 1060, 500, 190)
    if g < AA:
        col = mix(col, FLOOR, soft(g) * 0.9)
    col_acc = col
    acc = 0.0
    def over(cov, c):
        nonlocal col_acc, acc
        col_acc = mix(col_acc, c, cov)
        acc = acc + cov * (1 - acc)

    # --- background hearts & sparkles ---
    for hx, hy, hs in ((240, 250, 26), (780, 290, 20), (300, 150, 15), (720, 160, 13), (140, 430, 12), (890, 460, 15), (500, 90, 11)):
        f = heart_field(px, py, hx, hy, hs)
        if f < AA: over(soft(f) * 0.9, HEART)
    for sx, sy, sr in ((410, 200, 24), (650, 110, 19), (170, 620, 16), (860, 640, 20)):
        f = star_field(px, py, sx, sy, sr)
        if f < AA: over(soft(f) * 0.8, SPARK)

    # --- kitten body ---
    f = ellipse_field(px, py, KX, 905, 205, 155)
    if f < AA:
        over(soft(f), KITTY)
        over(ring(f) * 0.8, OUTLINE)
        f2 = ellipse_field(px, py, KX, 860, 185, 110)
        if f2 < AA: over(soft(f2) * 0.2, KITTY_SHADE)

    # --- bunny plush (in front of body) ---
    f = ellipse_field(px, py, BX, BY, 122, 128)
    if f < AA:
        over(soft(f), BUNNY)
        over(ring(f) * 0.7, BUNNY_OUT)
    # bunny ears
    f = capsule_field(px, py, BX - 55, BY - 95, BX - 78, BY - 215, 30)
    if f < AA:
        over(soft(f), BUNNY)
        over(ring(f) * 0.6, BUNNY_OUT)
        fi = capsule_field(px, py, BX - 55, BY - 100, BX - 76, BY - 205, 17)
        if fi < AA: over(soft(fi) * 0.8, BUNNY_SHADE)
    f = capsule_field(px, py, BX + 55, BY - 95, BX + 84, BY - 208, 30)
    if f < AA:
        over(soft(f), BUNNY)
        over(ring(f) * 0.6, BUNNY_OUT)
        fi = capsule_field(px, py, BX + 55, BY - 100, BX + 81, BY - 198, 17)
        if fi < AA: over(soft(fi) * 0.8, BUNNY_SHADE)
    # bunny face
    for ex in (BX - 30, BX + 30):
        f = circle_field(px, py, ex, BY - 38, 8)
        if f < AA: over(soft(f), EYE)
        f = circle_field(px, py, ex - 2, BY - 41, 2.6)
        if f < AA: over(soft(f), WHITE)
    f = capsule_field(px, py, BX - 8, BY - 16, BX + 8, BY - 16, 3.5)
    if f < AA: over(soft(f), MOUTH)
    for bx_ in (BX - 52, BX + 52):
        f = ellipse_field(px, py, bx_, BY - 22, 15, 9)
        if f < AA: over(soft(f) * 0.65, BLUSH)
    # plush stitch
    for i in range(3):
        f = capsule_field(px, py, BX - 12 + i * 12, BY + 42, BX - 8 + i * 12, BY + 58, 2.2)
        if f < AA: over(soft(f) * 0.8, BUNNY_OUT)

    # --- kitten front paws hugging bunny ---
    for px0, py0, ang in ((BX - 112, BY - 34, 0.55), (BX + 112, BY - 34, -0.55)):
        f = ellipse_field(px, py, px0, py0, 58, 46, rot=ang)
        if f < AA:
            over(soft(f), KITTY)
            over(ring(f) * 0.8, OUTLINE)
        for k in (-1, 0, 1):
            x1 = px0 + k * 16 * cos(ang) - 6 * sin(ang)
            y1 = py0 + k * 16 * sin(ang) + 6 * cos(ang)
            f = capsule_field(px, py, x1, y1, x1 + 16 * cos(ang), y1 + 16 * sin(ang), 2.2)
            if f < AA: over(soft(f) * 0.5, KITTY_SHADE)

    # --- kitten tail ---
    f = capsule_field(px, py, KX + 175, 940, KX + 275, 850, 38)
    if f < AA:
        over(soft(f), KITTY)
        over(ring(f) * 0.8, OUTLINE)

    # --- kitten head ---
    for ex, ey, rot in ((KX - 195, KY - 232, -0.5), (KX + 205, KY - 238, 0.45)):
        f = ellipse_field(px, py, ex, ey, 84, 112, rot=rot)
        if f < AA:
            over(soft(f), KITTY)
            over(ring(f) * 0.8, OUTLINE)
            fi = ellipse_field(px, py, ex, ey + 14, 42, 56, rot=rot)
            if fi < AA: over(soft(fi) * 0.6, (255, 205, 205))
    f = ellipse_field(px, py, KX, KY, HRX, HRY, rot=TILT)
    if f < AA:
        over(soft(f), KITTY)
        over(ring(f) * 0.9, OUTLINE)
    # forehead stripes
    for dx in (-60, 0, 60):
        f = capsule_field(px, py, KX + dx, KY - 240, KX + dx, KY - 182, 8)
        if f < AA: over(soft(f) * 0.45, KITTY_SHADE)

    # muzzle
    for mx in (KX - 62, KX + 62):
        f = ellipse_field(px, py, mx, KY + 92, 100, 82)
        if f < AA: over(soft(f) * 0.9, WHITE)

    # eyes
    def eye(ex, ey):
        f = ellipse_field(px, py, ex, ey, 60, 72, rot=TILT)
        if f < AA:
            over(soft(f), EYE)
            f3 = ellipse_field(px, py, ex, ey - 26, 44, 30, rot=TILT)
            if f3 < AA: over(soft(f3) * 0.55, (120, 96, 130))
            f4 = ellipse_field(px, py, ex - 20, ey - 26, 15, 19, rot=TILT)
            if f4 < AA: over(soft(f4), WHITE)
            f5 = circle_field(px, py, ex + 18, ey + 26, 8)
            if f5 < AA: over(soft(f5), WHITE)
            f6 = circle_field(px, py, ex + 2, ey + 10, 4.5)
            if f6 < AA: over(soft(f6), (255, 205, 225))
    eye(KX - 132, KY - 22)
    eye(KX + 138, KY - 14)

    # nose (tiny heart)
    f = heart_field(px, py, KX + 4, KY + 58, 15)
    if f < AA: over(soft(f) * 0.95, NOSE)
    # w-mouth + blep
    f = capsule_field(px, py, KX - 24, KY + 92, KX - 7, KY + 100, 3.2)
    if f < AA: over(soft(f), MOUTH)
    f = capsule_field(px, py, KX + 24, KY + 92, KX + 7, KY + 100, 3.2)
    if f < AA: over(soft(f), MOUTH)
    f = ellipse_field(px, py, KX + 2, KY + 120, 24, 18)
    if f < AA: over(soft(f), TONGUE)
    f = capsule_field(px, py, KX - 15, KY + 107, KX + 19, KY + 108, 2.2)
    if f < AA: over(soft(f), MOUTH)
    # blush
    for bx_, by in ((KX - 228, KY + 66), (KX + 240, KY + 58)):
        f = ellipse_field(px, py, bx_, by, 40, 24, rot=TILT)
        if f < AA: over(soft(f) * 0.7, BLUSH)
    # whiskers
    for k in (-1, 1):
        for dy in (-8, 8, 24):
            x1 = KX + k * 258
            y1 = KY + 78 + dy
            f = capsule_field(px, py, x1, y1, x1 + k * 85, y1 + dy * 0.4 - 5, 2.0)
            if f < AA: over(soft(f) * 0.55, (235, 185, 195))

    return col_acc

def render():
    rows = []
    for y in range(H):
        row = bytearray()
        for x in range(W):
            col = scene(x, y)
            row += bytes((min(255, max(0, int(col[0]))), min(255, max(0, int(col[1]))), min(255, max(0, int(col[2])))))
        rows.append(bytes(row))
    return b"".join(rows)

def png(data):
    def chunk(t, d):
        c = struct.pack(">I", len(d)) + t + d
        return c + struct.pack(">I", zlib.crc32(t + d) & 0xFFFFFFFF)
    ihdr = struct.pack(">IIBBBBB", W, H, 8, 2, 0, 0, 0)
    raw = b"".join(b"\x00" + data[i * W * 3:(i + 1) * W * 3] for i in range(H))
    return (b"\x89PNG\r\n\x1a\n" + chunk(b"IHDR", ihdr)
            + chunk(b"IDAT", zlib.compress(raw, 6)) + chunk(b"IEND", b""))

if __name__ == "__main__":
    out = os.environ.get("MAKEFUN_OUTPUT_DIR", "/output")
    os.makedirs(out, exist_ok=True)
    pix = render()
    with open(os.path.join(out, "entry.png"), "wb") as fh:
        fh.write(png(pix))
    print("wrote", os.path.join(out, "entry.png"))
