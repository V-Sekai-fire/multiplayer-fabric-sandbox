#!/usr/bin/env python3
"""Build RISC-V ELF sandbox programs for multiplayer-fabric-sandbox.

Usage:
  scons                             # build all programs
  scons riscv_cxx=riscv64-...      # override RISC-V C++ compiler
  scons -j4                         # parallel build

The monorepo layout assumed:
  multiplayer-fabric/
    multiplayer-fabric-godot/       (Godot engine fork + sandbox module)
    multiplayer-fabric-sandbox/     ← this directory
    multiplayer-fabric-taskweft/    (taskweft standalone headers)
"""
import os

# ── Directory layout ──────────────────────────────────────────────────────────
script_dir = Dir('#').abspath                         # multiplayer-fabric-sandbox/
monorepo   = os.path.dirname(script_dir)              # multiplayer-fabric/
godot_dir  = os.path.join(monorepo, 'multiplayer-fabric-godot')
api_dir    = os.path.join(godot_dir, 'modules', 'sandbox', 'program', 'cpp', 'docker', 'api')
sim_dir    = os.path.join(godot_dir, 'modules', 'multiplayer_fabric_mmog')
tw_dir     = os.path.join(monorepo, 'multiplayer-fabric-taskweft', 'standalone')

# ── Toolchain ─────────────────────────────────────────────────────────────────
riscv_cxx = ARGUMENTS.get('riscv_cxx', 'riscv64-linux-gnu-g++-14')

cxx_flags = [
    '-O2', '-std=gnu++23', '-DVERSION=10',
    '-fno-stack-protector', '-fno-threadsafe-statics',
    '-march=rv64gc_zba_zbb_zbs_zbc', '-mabi=lp64d',
]

wrap_syms = [
    'memcpy', 'memset', 'memcmp', 'memmove',
    'strlen', 'strcmp', 'strncmp',
    'malloc', 'calloc', 'realloc', 'free',
]
wrap_flags = ['-Wl,' + ','.join('--wrap=' + s for s in wrap_syms)]

env = Environment(
    CXX      = riscv_cxx,
    CXXFLAGS = cxx_flags,
    LINKFLAGS = ['-static'] + wrap_flags + cxx_flags,
    CPPPATH  = [api_dir],
)

# ── Shared API objects (compiled once, linked into every program) ─────────────
api_srcs = Glob(os.path.join(api_dir, '*.cpp'))
api_objs = [env.Object(src) for src in api_srcs]

# ── Per-program builder ───────────────────────────────────────────────────────
def sandbox_program(name, src, extra_includes=()):
    e = env.Clone()
    if extra_includes:
        e.Append(CPPPATH=list(extra_includes))
    obj = e.Object(source=src)
    return e.Program(target=name, source=[obj] + api_objs)

# ── Programs ──────────────────────────────────────────────────────────────────
sandbox_program('jellygrid_swarm',      'jellygrid_swarm.cpp',      [sim_dir])
sandbox_program('jellygrid_power_node', 'jellygrid_power_node.cpp', [sim_dir])
sandbox_program('jellygrid_current',    'jellygrid_current.cpp',    [sim_dir])

# taskweft standalone headers live in multiplayer-fabric-taskweft/standalone/
sandbox_program('taskweft_planner', 'taskweft_planner.cpp', [tw_dir])
