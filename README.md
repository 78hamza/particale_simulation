# Particale simulation (C + python) -- README
## Project overview
A small physics-based particale simulation implemented primarily in C (for low-level control and performance), with optional Python helpers for prototyping, testing, and visualization.
### Goals:
- Practice low-level programming in C (memory, pointers, performance modular code).
- Impliment and test physics models (Newtonian mechanics, pairwise gravity, collisions, spriings, drag)
- Compare numerical integrators (Euler, semi-implicit euler, Verlet, RK4)and understand stability/accuracy tradeoffs.
- Produce an interactive visualization (SDL2/OpenGL)

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Scope and features
Core features (minimus viable product)

- Basic `Particale` type with position, velocity, acceleration, mass.
- Vector math utilities (Vector2 operations)
- Time-stepping integration (start with semi-implicit Euler or velocity verlet)
- Pairwise gravitational interaction (naive O(n^2))
- Boundary handling (reflecting box) and simple collision handling.
- 2D visualization using SDL2 (draw particles, track FPS basic UI controls: pause, add particle)

Optional/advanced/features

- spatial partitioning (uniform grid, quadtree) or Barnes-Hut to reduce O(n^2)
- soft body/ spring-mass grid, cloth simulation
- GPU acceleration (OpenGL/CUDA) or SIMD optimations.
- Recording, export frames, save & load scenes.
- Small ML model (python) to predict short-term trajectories from recent frames.

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Math and physics (what we will impliment)
### Fundamental law:
- Newton's second law: F = m * a
### Common forces:
- Pairwise gravity (point masses):
    - F = G * m1 * m2 / (r^2 + ε^2) * r_hat (ε = softening to avoid singularities)
- Drag / damping:
    - Linear: F_d = -b * v.
    - Quadratic: F_d = -c * v * |v| (usefull for realistic air resistance at higher speeds)
- Spring (Hooke's law):
    - F = -k * (x - x_rest) (for spring-mass systems)
- Collision impuluse (elastic collision):
    - Relative velocity along normal: v_rel = v2 - v1
    - impulse scalar: j = -(1 + e) * (v_rel . n) / (1/m1 + 1 /m2) where e is coaficient of restitution.
    - velocity updates: v1 += (j / m1) * n; v2 -= (j / m2) * n;
### Numerical integration (time stepping)
- explicit euler (simple, unstable for still systems):
    - v_{t + dt} = v_t + a_t * dt;
    - x_{t+dt} = x_t + v_t * dt;
- semi-implicit (sysmplecity) eulewr (better energy behavior than explicit euler)
    - v_{t + dt} = v_t + a_t * dt
    - v_{t+dt} = x_t + v_{t+dt} * dt
- velocity verlet (good for conservation systems)
    - x_{t+dt} = x_t + v_t * dt + 0.5 * a_t * dt^2
    - compute a_{t+dt}
    - v_{t+dt} = v_t + 0.5 * (a_t + a_{t + dt} * dt)
- RK4 (Runge-Kutta 4th order) : high accuracy for smooth ODEs, more computting per step.

### Practicael notes
- choose dt small enough for stability, for still spinrg or csoe gravitaitonsl encounter you ma need adaptive dt or softening
- use doubel precicion (double) for simulating term ε for gravitatinal forde to avoitd (r -> 0)
singularitie.


## Progamming choice adn properties
Primary language: C
- Rationale: low-level control, performance, ploitner and memory practice.
- recommender stadard : -std=c11 or c17
- use double for positions/velocities, size_t for counts, malloc/realloc/freed for dyanmic arrays.
- use modular files: math/ (vectors), physics/ (forces, integrators), sim/ (practicle manager), render/ (SDL2), utili (time, logging)
- use tools: gcc/clang, make , valgring (memory chekcs), gdb (debugging), perf or callgring (profiling)

Helper language: python
- Rational: quick prototyping, plotting, numericla checks.
- uses jupter notebooks to sompare integrators, small scripts that load simulation dupms (CSV) and plot energy, trajectories.

Rendering
- start with SDL2 (2D drawing easy to set up) later option : OpenGL or gfx for better performance.

## Project structure

particle-sim/
├─ README.md
├─ Makefile
├─ include/
│ ├─ vector2.h
│ ├─ particle.h
│ └─ ...
├─ src/
│ ├─ vector2.c
│ ├─ particle.c
│ ├─ physics.c
│ ├─ integrators.c
│ ├─ render_sdl.c
│ └─ main.c
├─ python/ # notebooks & plotting helpers
├─ test/ # unit tests and small scenario scripts
├─ assets/ # optional images/fonts
└─ docs/

## Developement method and step by step plan
phase 0 -- setup
    - install toolchian: gcc/clang, SDL2 dev libs, python with numpy and matplotlib
    - create git repo and initial Makefile.
#### phase 1 -- math primitive and single particle
    - implement Vector2 operations and unit tests.
    - implemetn Particle struct and a single-particle integrator (simi-implicit euler)
    - acceptance: printed positions match analytic solution for constant acceleration.

#### phase 2 -- Muli-particle and pairwise forces
    - implement naive (O(n^2)) force loop for gravity
    - add softening parameter
    - acceptance: conserve momentum; for two-body case reproduce expected orbit shapes.
#### phase 3 -- conllisions and boudaries
    - add boundary reflection and impulse-based collisions.
    - acceptance: colling equal-mass particles swap velocites for head on eleastic collisions.
#### phase 4 -- Rendering
    - integreate SDL2, render particels with basic UI (pause, add particle, toggle trails)
    - acceptance: real-time rendering at reasonalbel FPS for N up to 1000 (depending on machine)
#### phase 5 -- Optimization
    - implemeent univofrm grid neighbor search or Barnes-hut.
    - Profile and optimize hotspot. Optionally add multi-threading.
#### phase 6 -- Polish
    - add Ui controles, save/load, snapthots, documentations, README finalization.

## What the final result look like 
- A deskto program that open a window with moving particles
- on-screen controles for pausing, adding/removing particles, toggling forces.
- Configuration simulation paramteres (G, dt, damping, collisions, integrator, choice)
- Ability to run headless and export postions to CSV for offline plotting.

## Suggested extras/ futur directions
- colro by speed or by potential/kinetic energy. add trails to visualize orbits.
- implement barnes-hut for O(n log(n)) N-body performance.
- GPU acceleration (CUDA/OpenGL) or WebAssembly (compile to wasm + run in browser)
- Add intercative parametre sliders (ImGui) to explore sensitivity to dt, G, restitution e.
- Record small ML dataset (last k states) -> predict next state, train a small model in python to compare with physics integrator.

