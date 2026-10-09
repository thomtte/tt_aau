# Thomas Tellarini — shared rules for every folder

Thomas is a Senior/Lead Architect at AAU ANASTAS (Bahrain heritage and cultural projects). Very proficient in Rhino, base level in Grasshopper: explain GH wiring step by step.

**Replies: short and synthetic.** Say what's done and the one next step. No option lists or long background unless asked.

## Folders
- `1_CAREER/` — portfolio, job applications, job search.
- `2_DBMM/` — Dilmun Burial Mounds Museum (BACA).
- `3_AALI_POTTERY_HUB/` — A'ali Pottery Hub (BACA): Grasshopper scripts, image prompts, references.
Every project folder has the same numbered layout (office style: CAPS, underscores, no spaces). Create all folders even if empty:
- `0_INBOX/` — Thomas drops raw material here; sort it into the folders below.
- `1_REFERENCE/` — client, tender and design documents. Read-only.
- `2_NOTES/` — working notes and project memory (overview, ways of working, standards).
- `3_SCRIPTS/` — Grasshopper/Rhino and other Python scripts; `CLAUDE.md` index; `1_DOCS/` per-script docs.
- `4_IMAGE_PROMPTS/` — `1_PROMPTS/` (in git), `2_IMAGES/` (screenshots + results, not in git).
- `5_OUTPUTS/` — deliverables Claude produces.
**Project memory:** conversations don't share context, so files are the memory. Whenever a design decision is made or a direction changes, record it in the project's `2_NOTES/` in the same turn (date, decision, why, where the references/scripts are). Keep script indexes in `3_SCRIPTS/CLAUDE.md` current. **Save only successful outcomes:** record a prompt, script approach or lesson only after Thomas has confirmed it worked; never log failed or untested attempts, and remove saved items that turn out to have failed.
`CLAUDE.md` and `.claude/` stay at the project root. New project → next number, same layout. Subfolders are numbered too.

## Grasshopper / Rhino Python (all projects)
- Rhino 8 on macOS, Grasshopper **Python 3 Script component** (CPython via pythonnet). Document units **millimetres**. Tolerance: TODO.
- Scripts live in the project's `3_SCRIPTS/`, short plain names (`railing.py`, `facade.py`). Versions are git commits, never file names (no `_v7`, `_final`).
- Thomas links the component to the file with a 3-line stub, so Claude edits the file and Thomas presses F5:
  ```python
  import os
  path = os.path.expanduser("~/Desktop/[claude]/<PROJECT>/3_SCRIPTS/<name>.py")
  exec(open(path).read())
  ```
- Header docstring in every script: purpose, version, inputs (name, type hint, access, default), outputs, what changed.
- Read inputs with a helper (`_input("Name")` / `get(name, default)`) from globals, so only essential inputs must exist. Never define a class/function/global with an input's name (GH injects inputs as globals and they clash).
- Report through an `Info` output: version first, counts, checks, errors (wrap the build in try/except, traceback to Info).
- Prefer RhinoCommon over `rhinoscriptsyntax`. Keep pure math in plain functions so it can be tested outside Rhino. Code is not run in Rhino by Claude: say which parts need Thomas's test.

### Pitfalls (do not regress)
1. List outputs: assign `System.Collections.Generic.List[...]`, never a Python list (GH shows one `Null`).
2. The first output of the Rhino 8 component is the console `out`; renaming it doesn't bind a variable. An output that never passes data: delete it and add a fresh one with a new name.
3. Don't mutate RhinoCommon structs in place (Point3d.Transform, BoundingBox.Union, Transform.Mxx); build new values. Prefer an explicit `dot()` over `v1 * v2`.
4. `BrepFace.NormalAt` already accounts for `OrientationIsReversed`; verify direction geometrically.
5. No solid booleans (coincident faces drop walls, curved-plan booleans crash Rhino). Build solids directly from curves/sections; closed solids can't be joined, so loft one solid from sections.
6. Native crashes can't be caught: guard every loop (max count, min step), check a piece budget before building, add a `Run` toggle. A huge curve length usually means a units mix-up.
7. Heavy input curves (hundreds of control points) make everything slow: rebuild within ~1 mm first.
8. Point inputs need List Access (+ Flatten) or GH runs the script once per point. Accept a single item, a Python list or a .NET list.
9. Users draw curves "backwards": auto-detect direction.
10. `rs.ApplyBoxMapping` doesn't exist in this Rhino 8: use `Rhino.Render.TextureMapping.CreateBoxMapping` + `doc.Objects.ModifyTextureMapping`.
11. Rendering: georeferenced models (~km from origin) render wrong in Cycles — render a copy moved near 0,0,0; GH previews don't render, bake first.

## Image prompts — Nano Banana / Leonardo (all projects)
- Two passes: Pass 1 on the Rhino screenshot (materials, trees, paths, sky, light); Pass 2 on the Pass 1 result (ground, low planting, people), opening with "keep everything exactly as it is".
- Pass 1 order: image type + place → preserve-geometry block (camera, framing, edges overlay exactly; change only materials, textures, light) → stereotomy (joints only where drawn) → one paragraph per material/element → sky → light and colour, then restate the preserved camera.
- After the preserve-geometry block, **no geometric words** (shapes, courses, curvature, rings): geometry in the prompt overrides preservation and causes drift.
- Name each placeholder and what it becomes "in exactly the same position, height and outline". Say what things are NOT when the model confuses them.
- House grading: neutral-cool, 6000–6500 K, soft cool-grey shadows, faint cool haze; no yellow/golden/sepia, no dusty warm air.
- After each round: Thomas says what drifted → fix the prompt → add a line to the project's prompt lessons.

## Lessons learned (general)
<!-- One line per non-obvious finding that applies beyond one project. -->
