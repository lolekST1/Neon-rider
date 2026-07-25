# slider — neon rider

A single-file HTML5 / JavaScript arcade game inspired by **Slider**. Ride a glowing,
procedurally generated neon line, launch off crests, flip through the air, grab diamonds,
and land on your wheels — one wrong landing and it's game over.

## Play

Open `index.html` in any modern browser. No build step, no dependencies.

## Controls (one button)

- **Hold** `Space` / **click** / **tap** &nbsp;→&nbsp; accelerate on the ground; **flip** while
  airborne. The bike moves **only** while you hold — release and it coasts to a stop. It never
  drives on its own.
- Land on your wheels, roughly matching the slope, to keep going and earn style points.
  Land upside-down or at a bad angle and the bike explodes.

## Features

- Procedural terrain from layered smooth value-noise (endless, difficulty ramps with distance).
- Rigid-body physics: two wheels colliding with the track independently, suspension-softened
  impacts, ballistic launches off crests, backflip torque in the air, and landings that pivot
  from the rear wheel onto the front — no snapping. A crash only happens when the chassis or
  the rider touches the line; wheels-first is always safe.
- Neon / cyberpunk styling with glow, a parallax grid backdrop and drifting geometric shapes.
- Track, vehicle and title colours cycle per level.
- Collectible diamonds, distance score, floating praise (`smooth!`, `great!`, `aced!`, `WOW!`).
- A `last score` marker hangs on the track where you crashed last run.
- Explosion particle effects and screen shake.
- Web Audio API sound effects (jump, gem, land, crash) and a looping synth-wave backing track.
- Persistent best distance / gem count via `localStorage`.

Everything lives in `index.html` — pure Canvas 2D, no external libraries.
