# MATLAB → ArcGIS FGDB pipeline

![Pipeline architecture](matlab_arcpy_pipeline.svg)

## What it does
Converts large batches of MATLAB-exported HDF5/SIO simulation output (hydrodynamic, temperature,
velocity, scalar, and boundary datasets) into ArcGIS File Geodatabases, one geodatabase per input
file, at production scale — hundreds of files per scenario, each requiring its own resumable job.

## Architecture, stage by stage
1. **Config CSV** — each scenario is defined as a list of input files in a CSV job list.
2. **CLI entrypoint** (`matlab_to_arcpy.py`) — sets the correct multiprocessing start method
   per OS (`spawn` on Windows, `fork` on Unix) and kicks off one `ScenarioProcessor` per CSV.
3. **`ScenarioProcessor`** — loads a persisted `state.json`, filters out files already marked
   complete, and only submits the remaining work.
4. **`ProcessPoolExecutor`** — parallel worker processes, sized dynamically from CPU count and
   available RAM per file (memory-aware scheduling, not a fixed worker count).
5. **Factory dispatch (`create_processor`)** — opens the file, inspects which fields are present,
   and matches it to the correct registered processor class (Temperature, Hydrodynamic,
   MultiDimensional, Velocity, Scalar, Boundary) — a strategy/factory pattern rather than
   hardcoded branching.
6. **`register_all_processors.py`** — a lightweight plugin registry: importing this module
   registers every processor implementation with the factory, so adding a new data type means
   adding a new module, not touching the dispatch logic.
7. **Output** — each processor writes an ArcGIS shapefile via `arcpy`/`fiona`, merged into a
   per-file File Geodatabase.

Two cross-cutting concerns run alongside the main flow:
- **`ProgressMonitor`** — a background thread reading off an `mp.Queue`, aggregating per-file
  progress into console output, log files, and JSON snapshots — pluggable via custom handlers
  (a console progress bar and a websocket handler are both implemented as swappable handlers).
- **Checkpointing** — every file's processing state is persisted, so a killed or crashed run
  resumes mid-batch instead of reprocessing everything.

## Feature summary

| Requirement | What this project demonstrates |
|---|---|
| Python proficiency | Full OOP pipeline: base processor class, six subclasses, factory pattern, CLI via argparse |
| Task queues / distributed processing | `ProcessPoolExecutor` + `mp.Queue` coordination — not Celery/Kafka, but the same producer/consumer, backpressure-aware shape |
| System optimization at scale | Worker count derived from CPU cores and RAM budget per file, not a static number |
| Implementation & maintainability | Plugin-style processor registry (`register_all_processors.py`) — new data types register themselves rather than requiring changes to dispatch code |
| Reliability under long-running jobs | Per-file JSON checkpointing enables resume after failure — critical when a scenario run can be hours long |
| ArcGIS (preferred skill) | Direct `arcpy` integration, File Geodatabase creation, ArcGIS Pro conda environment |

## Celery Like?
This isn't Celery/RabbitMQ — it's Python's built-in `multiprocessing` and `concurrent.futures`.
The talking point: the underlying problem (queue of jobs, N parallel workers, progress reporting,
resumability on failure) is the same shape a Celery-based system solves, just built with stdlib
primitives because the deployment target was a single powerful workstation/laptop rather than a
distributed broker setup. "I'd ramp up quickly on Celery specifically."
