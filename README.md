# Multi-Level Feedback Queue Scheduler

A simulation of an operating system's CPU scheduler, written in C for Linux. A master process
(`oss`) launches worker processes and schedules them with a **multi-level feedback queue (MLFQ)**:
three priority queues where each lower queue gets a longer time slice, and processes that use their
whole slice are moved down a level.

*Course project for CMP SCI 4760 (Operating Systems), University of Missouri–St. Louis.*

## How it works

```
 Queue 0  (highest)  quantum 10 ms  ── used full slice ─▶ demote
 Queue 1             quantum 20 ms  ── used full slice ─▶ demote
 Queue 2  (lowest)   quantum 40 ms  ── stays in queue 2
```

- **Processes:** `oss` launches workers with `fork()`/`execl()`, up to 40 in total and at most 18 at
  once. Each new process starts in queue 0.
- **Scheduling:** each round, `oss` takes the first process from the highest non-empty queue and
  sends it a time slice over a message queue. The slice doubles at each level (10, 20, then 40 ms of
  simulated time).
- **Workers:** a worker uses part or all of its slice and reports back how much time it used, and
  whether it's finished (about a 1 in 5 chance each time it runs).
- **Feedback:** a process that isn't finished moves down one queue. Queue 2 is the floor.
- **Shared state:** a simulated system clock and the process control block (PCB) table live in
  System V shared memory. The clock advances by each process's CPU time plus scheduling overhead.
- **Logging:** process terminations and their simulated times are written to `log.txt`, and the
  scheduling decisions are printed to the console.
- **Cleanup:** a `SIGINT` handler removes the shared memory and message queue.

## Build and run

Requires Linux (or WSL) with `gcc` and `make`.

```bash
make        # builds ./oss and ./worker
./oss       # runs the scheduler
make clean
```

## Files

| File | Purpose |
| --- | --- |
| `oss.c` | The scheduler: process launching, the three queues, time slices, demotion |
| `worker.c` | A simulated user process that uses some of its slice and may finish |
| `shared.h` | Shared definitions: clock, PCB, message format, limits |
| `Makefile` | Builds both programs |

## What I'd improve next

- **Aging:** move processes that have waited a long time in a low queue back up, so they can't starve.
- **I/O blocking:** use the PCB's `blocked` field so processes can block on simulated I/O and come
  back later.
- **Statistics:** report average wait time, turnaround time, and CPU utilization at the end.
