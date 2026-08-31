# ZeroForge

A zero-dependency local task engine that understands dependencies, constraints, priorities, and deadlines to determine what you can work on next.

ZeroForge is a lightweight, local-first task manager designed for developers and small teams who need more than a simple to-do list. Instead of only tracking tasks, it models task relationships and computes what is actually ready to work on next based on dependencies, priority, and urgency.

[![Zero Dependencies](https://img.shields.io/badge/dependencies-0%20runtime-brightgreen.svg)](#zero-dependency-proof)
[![Standard Library](https://img.shields.io/badge/python-3.9%2B%20stdlib%20only-blue.svg)](#standard-library-substitutions)
[![License: MIT](https://img.shields.io/badge/license-MIT-purple.svg)](LICENSE)

---

## Overview

Traditional task apps answer:

> What tasks do I have?

ZeroForge answers:

> Given my tasks, dependencies, deadlines, priorities, and constraints, what can I actually work on right now?

A task graph is at the core of the project. ZeroForge uses dependency-aware logic to determine whether something is ready, blocked, or scheduled next.

```text
Design Database ──► Build API ──► Write Tests ──► Deploy
```

If `Build API` depends on `Design Database`, it cannot begin until the earlier task is completed.

---

## Why ZeroForge?

- Dependency-aware task planning
- Ready vs blocked logic based on task relationships
- SQLite-backed local persistence
- Cycle prevention and graph validation
- Command-line, REPL, and guided wizard workflows
- Zero third-party runtime dependencies

---

## Features

- Dependency graph engine with cycle detection
- Dynamic readiness checks without stale state flags
- Deterministic scheduling based on priority, urgency, deadlines, and age
- SQLite persistence with ACID guarantees
- Terminal-friendly DAG visualization
- Interactive REPL with command history and fuzzy matching
- Guided wizard for beginner-friendly onboarding
- 100% Python standard library implementation

---

## Download and Quick Start

### Download the ZIP

This website provides the latest release package:

- [Download ZeroForge v1.0.0 ZIP](releases/ZeroForge-v1.0.0.zip)

### Windows

Double-click `run.bat` or run it from Command Prompt / PowerShell:

```cmd
run.bat
```

### macOS / Linux

```bash
./run.sh
# or
python3 -m zeroforge
```

### Direct Launcher

Running ZeroForge without arguments launches the interface selector:

```bash
python -m zeroforge
```

Example output:

```text
============================================================
ZEROFORGE
Dependency-Aware Task Engine
============================================================

Choose an interface:

  1. Command Line
     Run individual ZeroForge commands.

  2. Interactive REPL
     Work continuously inside ZeroForge.

  3. Guided Wizard
     Manage tasks through a guided interface.

  4. Exit

Select an option [1-4]:
```

---

## Three Ways to Use ZeroForge

### 1) Guided Wizard

Best for new users.

```bash
python -m zeroforge wizard
```

The wizard offers a step-by-step interface with task creation, viewing tasks, marking tasks complete, checking ready work, and inspecting dependency graphs.

### 2) Interactive REPL

Best for power users who want to work continuously inside ZeroForge.

```bash
python -m zeroforge repl
```

Example:

```text
zf > add Design Database --priority high
[OK] Created task #1: Design Database
   Priority  : HIGH

zf > add Build API --priority critical --after 1
[OK] Created task #2: Build API

zf > ready
  #1  HIGH      READY        Design Database

zf > done db
[OK] Completed task #1: Design Database
  Tasks now ready:
    #2  CRITICAL  Build API
```

Features include:

- command history with arrow keys
- tab completion
- aliases such as `ls`, `rm`, and `q`
- fuzzy task matching like `done db`

### 3) Direct Commands

Best for scripts and automation.

```bash
python -m zeroforge add "Design Database" --priority high
python -m zeroforge add "Build API" --priority critical --after 1
python -m zeroforge add "Write Tests" --priority high --after 2
python -m zeroforge add "Deploy" --priority critical --after 3
python -m zeroforge ready
python -m zeroforge plan
python -m zeroforge graph
```

---

## Core Workflow Demo

```bash
# 1. Create tasks with priorities and dependencies
python -m zeroforge add "Design Database" --priority high
python -m zeroforge add "Build API" --priority critical --after 1
python -m zeroforge add "Write Tests" --priority high --after 2
python -m zeroforge add "Deploy" --priority critical --after 3

# 2. Inspect what is immediately actionable
python -m zeroforge ready

# 3. View what is currently blocked and why
python -m zeroforge blocked

# 4. View dependency graph
python -m zeroforge graph

# 5. Generate a complete execution plan
python -m zeroforge plan

# 6. Complete a task and see what unlocks next
python -m zeroforge done 1
python -m zeroforge ready

# 7. Cycle protection example
python -m zeroforge dep add 1 --on 4
# Output: ERROR: Dependency cycle detected: #1 -> #4 -> #3 -> #2 -> #1
```

---

## Command Reference

### Task Management

```bash
python -m zeroforge add "Task Name"
python -m zeroforge add "Task Name" --priority high
python -m zeroforge done 1
python -m zeroforge list
python -m zeroforge delete 1
python -m zeroforge update 1 "Updated task name"
```

### Dependencies

```bash
python -m zeroforge dep add 1 --on 2
python -m zeroforge dep remove 1 --on 2
python -m zeroforge blocked
python -m zeroforge graph
```

### Planning and Readiness

```bash
python -m zeroforge ready
python -m zeroforge plan
python -m zeroforge schedule
```

### System and Health

```bash
python -m zeroforge health
python -m zeroforge --version
python -m zeroforge version
```

### REPL Commands

```text
zf > add Design Database --priority high
zf > list
zf > ready
zf > done db
zf > quit
```

---

## Architecture Overview

```text
                      +-------------------------+
                      |       CLI Parser        |  (argparse)
                      +------------+------------+
                                   |
                      +------------v------------+
                      |     Core Engine         |  (Business Orchestration)
                      +---+--------+--------+---+
                          |        |        |
        +-----------------+        |        +-----------------+
        |                          |                          |
+-------v-------+          +-------v-------+          +-------v-------+
|  Dependency   |          |   Scheduler   |          | SQLite Store  |
|  Graph Engine |          |  (Multi-Key)  |          | (sqlite3 WAL) |
+---------------+          +---------------+          +---------------+

Optional UI Layers:
+-------v-------+          +-------v-------+
|  Interactive  |          |   Guided      |
|  REPL Shell   |          |   Wizard      |
+---------------+          +---------------+
```

---

## Health Check

Verify your environment, database, graph logic, and interfaces:

```bash
python -m zeroforge health
```

Example output:

```text
============================================================
                 ZEROFORGE HEALTH CHECK
============================================================

  Environment
    [OK]    Python Version (>= 3.9)      v3.13.8
    [OK]    Runtime Dependencies         0 third-party (100% stdlib)

  Core Components
    [OK]    Database & Migrations        SQLite WAL mode + Foreign keys initialized
    [OK]    Task Storage (CRUD)          CRUD & cascade operations verified
    [OK]    Dependency Graph             3-Color DFS cycle prevention & topological sort verified
    [OK]    Planner / Scheduler          5-key deterministic multi-tier ranking verified

  User Interfaces
    [OK]    CLI Interface                Argparse command parser verified
    [OK]    Interactive REPL             REPL commands, token parser & fuzzy search ready
    [OK]    Guided Wizard                Guided wizard & natural date parser ready

------------------------------------------------------------
  Overall Status: HEALTHY
------------------------------------------------------------
```

---

## Version

```bash
python -m zeroforge --version
# or
python -m zeroforge version
```

Output:

```text
ZeroForge v1.0.0
```

---

## Running Tests

```bash
python -m unittest discover tests
```

---

## Zero-Dependency Proof

ZeroForge uses only Python's standard library. No third-party runtime libraries are required.

```bash
python -c "import zeroforge; print('ZeroForge runs with 0 third-party dependencies!')"
```

---

## Standard Library Substitutions

| External Package | ZeroForge Replacement | Purpose |
|---|---|---|
| `click` / `typer` | `argparse` | CLI parsing |
| `sqlalchemy` / `peewee` | `sqlite3` | Relational persistence |
| `networkx` | custom graph engine | Dependency logic |
| `pydantic` | dataclasses + validation | Domain models |
| `rich` / `tabulate` | custom formatting utilities | Output formatting |
| `pytest` | `unittest` | Testing |
| `python-dateutil` | `datetime` | Natural date handling |
| `IPython` / `prompt_toolkit` | custom REPL | Interactive shell |

---

## License

MIT License. See [LICENSE](LICENSE) for details.

---

## Project Notes

This repository includes the static website used to distribute the ZeroForge project and explain how to download, extract, run, and use the released ZIP version.

The project is intentionally minimal, fast, and local-first. It is designed to work without external services or heavy tooling.
