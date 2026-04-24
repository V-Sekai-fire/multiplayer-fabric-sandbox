# AGENTS.md — multiplayer-fabric-sandbox

Guidance for AI coding agents working in this submodule.

## What this is

C++20 RISC-V sandbox programs compiled for the `godot-sandbox` module inside
`multiplayer-fabric-godot`. Produces RISC-V ELF kernels that run inside the
sandboxed Godot extension. Changing a public kernel symbol requires updating
the binding table in the godot module at the same time.

## Build and test

```sh
# Requires RISC-V cross-compiler (see skills/devops/riscv-toolchain-setup)
cmake -B build -DCMAKE_TOOLCHAIN_FILE=riscv-toolchain.cmake
cmake --build build
ctest --test-dir build        # CMake / CTest + ASAN
```

## Key files

| Path | Purpose |
|------|---------|
| `CMakeLists.txt` | Build config; adds jellygrid and taskweft programs |
| `riscv-toolchain.cmake` | Cross-compilation toolchain file |
| `build.sh` | Convenience wrapper around cmake configure + build |
| `jellygrid_swarm.cpp` | Swarm simulation kernel |
| `jellygrid_power_node.cpp` | Power-node simulation kernel |
| `jellygrid_current.cpp` | Current-flow simulation kernel |
| `taskweft_planner.cpp` | HTN planner kernel (uses taskweft standalone headers) |

## Dependencies

- `godot-sandbox` cmake infrastructure (referenced via relative path from
  the godot fork's `modules/sandbox/`)
- `multiplayer_fabric_mmog` sim headers (pure C++, no Godot or api.hpp)
- `multiplayer-fabric-taskweft/standalone/` headers (header-only C++)

## Conventions

- All kernels must be RISC-V compatible — no host-only intrinsics.
- Test integration by building the godot fork with the updated sandbox ELF.
- Commit message style: sentence case, no `type(scope):` prefix.
  Example: `Add power-node current propagation to jellygrid kernel`
