# Session: clone FreeCADgo and assess a Go port

Session ID: `325aa5c1-2a15-4817-8da7-6daf99f8706e`

## Summary

1. **Cloned the repo.** Found `Noahs-ark21/FreeCADgo` (a fork of `FreeCAD/FreeCAD`) and cloned it to `~/projs/FreeCADgo`.
   - `origin` is the fork and `upstream` is `FreeCAD/FreeCAD`, which `gh repo clone` added automatically.
   - On `main`, level with `origin/main`. HEAD was `66644dee63` ("Security: Update Python libs post-eMuhdra").
   - Submodules are **not initialised**: `src/3rdParty/GSL`, `src/3rdParty/OndselSolver`, `src/3rdParty/coin`, `src/3rdParty/pivy`, `src/Mod/AddonManager`. Run `git submodule update --init --recursive` before building.
   - The `.gitmodules` "multiple configurations" warnings come from one historical commit and are harmless.

2. **Assessed converting FreeCAD to Go.** Recommendation: don't do a full port.

### Codebase size (tracked files, excluding `src/3rdParty`)

| Language | Lines |
|---|---|
| C++ (`.cpp` + `.h`) | ~1.28M |
| Python | ~780k |
| Go | 0 |

Largest areas: Fem 333k, Gui 277k, CAM 241k, Part 168k, Sketcher 136k, TechDraw 127k, BIM 112k, App 95k, Base 53k.

Key dependencies (from `pixi.toml`): OCCT 8.0, Qt 6.11 and PySide6, Python 3.13, VTK, SMESH, Eigen, Boost, Xerces-C and HDF5. Coin3D and pivy are submodules.

### Main blockers

- OCCT, the geometry kernel, has no Go equivalent and is millions of lines of C++.
- cgo can only call C, so OCCT, Qt, VTK and SMESH would each need C wrappers, plus object-lifetime management across Go's garbage collector.
- The GUI needs Qt 6, 310 `.ui` files and Coin3D. Go's Qt bindings are community-maintained, and Go has no Coin3D equivalent.
- The embedded Python API is the plugin system for macros, addons and workbenches.
- The numeric code relies on Eigen (Go's `gonum` is weaker), and models need regression tests against real `.FCStd` files.

### Rough estimate

About 80–100 engineer-years for the C++ alone, assuming ~16k tested lines per engineer per year. AI translation would reduce it, but verification remains the bottleneck.

### Options presented

| Approach | Effort | Result |
|---|---|---|
| Go tools around FreeCAD (headless `FreeCADCmd`, read/write `.FCStd` zip+XML) | Weeks | Go CLI/API and services, with all FreeCAD features |
| Small Go CAD core (C wrapper over a subset of OCCT) | About 1–2 engineer-years | Parametric parts in Go, no GUI/FEM/CAM |
| Full port with OCCT/Qt/Python behind wrappers | Decades | Little gain over C++ |

Recommended starting with the first option. Asked the user why they want Go; they haven't answered yet.

## Next

- Decide the goal of the Go work (services/CLI, embeddable modelling, or something else). That decides which option to pursue.
- If option 1: prototype a Go package that reads and writes `.FCStd` files and drives headless `FreeCADCmd`.

**Validate**

- Initialise submodules (`git submodule update --init --recursive`) and confirm the tree builds with pixi.
