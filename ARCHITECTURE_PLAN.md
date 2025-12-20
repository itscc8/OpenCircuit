# OpenCircuit "Ultimate EDA" Architecture Plan

This document acknowledges the approved **Ultimate EDA** specification and outlines an execution strategy that keeps the existing vanilla JavaScript + HTML5 Canvas stack while targeting 60 FPS rendering. No new features are implemented here—this is a roadmap to guide phased delivery.

## Guiding principles
- Preserve the current JSON graph model (components, ports, wires) as the single source of truth.
- Keep simulation and rendering decoupled; prioritize event-driven work to protect frame rate.
- Prefer data-driven configuration (bit widths, parameters, bundles) over bespoke components.
- Instrument every new engine feature with tests and UI probes to keep regressions visible.

## Phase 1: Core simulation engine (foundation)
- **Propagation delay queue:** Replace instant propagation with a future-event queue keyed by `tick + time_cost`; gates schedule output updates when inputs change and honor glitches.
- **Dirty-component scheduling:** Maintain a queue of components whose inputs changed; process only dirty nodes and their fan-out per tick.
- **Clock + DFFs:** Add a global clock generator component and D-Flip Flop that latches on rising edges only; make edge detection explicit in the scheduler.
- **Memory primitives:** Back RAM/ROM with `Uint8Array`, expose byte-level load/store, and surface a lightweight hex editor modal for live inspection.

## Phase 2: Hierarchy & abstraction
- **Parameterized modules:** Extend custom components to accept a `parameters` object (e.g., bit widths); regenerate ports and internal wiring from parameters.
- **Dive-in editing:** Double-click swaps the viewport to the subcircuit while preserving simulation state; edits update all instances of the custom chip type.
- **Bus interfaces:** Introduce `Bundle` to group wires, render as a thick line, and supply splitter/merger helpers for bit breakout/merge.
- **Verilog/SystemVerilog export:** Traverse the JSON graph to emit structural Verilog with sanitized identifiers, modules, assigns, and wires.

## Phase 3: Canvas rendering & routing
- **Orthogonal auto-routing:** Use Manhattan-style pathfinding (A*/Lee) for wires with dynamic re-route on drag.
- **Wire visualization toggle:** Context menu flag to animate selected wires with marching-ants flow indicators.
- **Named nets:** Tag wires; identical tags share connectivity even without a visual link.
- **Minimap + semantic zoom:** HUD overlay showing viewport; simplify rendering (no text/pin detail) below zoom thresholds for FPS stability.

## Phase 4: Analysis, debugging, quality
- **Integrated logic analyzer:** Bottom panel with scrolling history, triggers, and cursors for tick-level inspection.
- **Design rule checks:** Periodic/on-demand scan for short circuits and floating inputs, surfaced as canvas warnings.
- **Testbench sequencer:** UI to run `{Tick, Input, ExpectedOutput}` sequences and display pass/fail.
- **Conditional breakpoints:** Pause simulation when user-defined logic expressions evaluate true.
- **Coverage heatmap:** Track per-gate toggle counts and render post-run heatmap (grey/green/red).

## Phase 5: Advanced tooling & UX
- **FSM editor:** Graph-mode editor that synthesizes states/transitions into gates + DFFs.
- **Integrated assembler:** Regex-driven ISA definitions compiled into binary and injected into a selected ROM.
- **Spotlight search:** `Ctrl+K` palette indexing component IDs, net names, and actions with fuzzy jump/execute.
- **Undo/redo via command pattern:** Wrap canvas actions (place/move/delete/wire) as commands pushed to a bidirectional stack.
- **Shortcut mapper:** Settings UI to remap keys (defaults: W wire, R rotate, Del remove, Space sim toggle, S select).

## Immediate next steps
1. Profile the current tick/double-buffer loop and map gate types to default `time_cost` values.
2. Design the future-event queue and dirty-component scheduler interfaces (enqueue, cancel, commit).
3. Specify the global clock component contract and DFF edge-detection semantics.
4. Define RAM/ROM data shapes and hex-editor UI contract; ensure saves/loads stay JSON-compatible.
5. Add targeted tests around the new scheduler before layering higher-phase features.
