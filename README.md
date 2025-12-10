# A Niche Database - JavaScript Version

A lightweight SQLite-inspired embedded database implemented in Node.js. The system is designed as an educational and experimental database engine that focuses on learning data storage, indexing, transactions, concurrency, durability modes, CLI tooling, and performance benchmarking. The goal is to gradually evolve from a simple append-only log to a page-based, indexed, transactional database with SQL support, a CLI shell, and optional server mode.

PROJECT OBJECTIVES

- Build a real database engine with meaningful internals.

- Implement a working storage layer before abstractions or UI.

- Support a minimal SQL subset for inserts, reads, updates, deletes, and table definitions.

- Offer a CLI tool that feels similar to sqlite3, with “init”, “exec”, and “shell” commands.

- Eventually support binary packaging and Docker deployment.

- Maintain performance benchmarks and track results over time.

- Evolve toward a page-based architecture and WAL-based durability, as seen in real systems.

---

## DATA STORAGE PHILOSOPHY

The system uses an append-only log to record operations. Each write is appended to the end of the database file rather than modifying existing records. On startup, the engine rebuilds in-memory state by replaying all operations recorded in the file. This design simplifies early implementation because sequential appends are efficient and recovery from partial writes is easy to reason about.

Over time, the database will evolve toward a hybrid model, where the main data file represents the current state (via snapshots or paged structures), and a log or WAL records recent operations for durability and crash recovery. This aligns with the behavior of real production databases.

---

## FILE FORMAT

A database is represented as a single file with the following layout:

### 1. Header section:

- Magic bytes identifying the file as belonging to this database.

- File format version number to ensure compatibility with future changes.

### 2. Log records section:

- Each record contains a fixed-length integer representing the size of the record.

- The record data follows as a UTF-8 encoded JSON structure describing the operation.

This layout supports sequential scanning and avoids delimiter ambiguity. A record can be parsed by reading its length first, followed by that many bytes of JSON data.

---

## REPLAY ON OPEN

When a database file is opened, the engine reads the header to validate the file, then scans the sequence of log records in order. Each record is applied to an in-memory structure, such as a Map for key/value state or table data. This process reconstructs the latest state from the log. Later optimization phases will reduce startup time through snapshots or paging.

---

## DEFAULT FILE LOCATIONS

Actual users specify where database files are stored. The CLI will always allow explicit file paths for init, queries, compaction, and shell interactions.

Development and automated testing will use a folder inside the repository named test-data. This folder contains only temporary database files generated during testing or benchmarking. These files are not intended for production usage.

---

## DURABILITY MODES

Writing to disk does not guarantee immediate persistence because operating systems buffer writes in memory. When durability is critical, an explicit fsync call ensures that data has been physically flushed to disk before processing continues. This makes writes slower but safer under crash conditions.

Two durability modes are planned:

### 1. Safe mode:

- Uses periodic or transaction-level fsync operations.

- Slower but crash-safe.

### Fast mode:

- Relies on OS buffering.

- Very fast, but recent writes may be lost on crash.

Initial implementation may choose either default mode, with future versions exposing an explicit runtime setting.

---

## SQL FEATURES (INCREMENTAL)

The first version of the system will support operations at the key/value level only. Next phases will introduce tables, schemas, primary keys, basic constraints, and a minimal SQL execution engine. Indexing and query planning will be added later to improve performance for selective reads.

---

## CLI DESIGN

The system includes a command-line tool with subcommands:

- init: create a new database file
- exec: execute a query or operation against a database file
- shell: launch an interactive REPL-style interface
- info: show metadata, header, record count, and file size
- compact: rebuild a clean state representation by writing the latest state into a new file

The long-term goal is for the CLI to behave similarly to sqlite3 in terms of basic user experience.

---

## BENCHMARKING

A dedicated benchmarks folder exists to measure insert-rate, random-read performance, and mixed workloads. Benchmark output logs operations per second, latency, and final file size. Benchmark results will be version-controlled to avoid regression and to track storage and execution improvements across development milestones.

## VERSIONING

The repository, library, CLI tool, Docker image, and binaries will follow semantic versioning. The file format version stored inside each database ensures that incompatible versions are detected at open time and prevents accidental corruption. Migration utilities may be introduced later.

## FUTURE EXPANSION

- Page-based storage layout for faster reads and predictable IO patterns.

- Indexed table structures using B-trees or similar.

- Write-ahead logging or WAL for safe transactional semantics.

- Server mode for multi-client workloads.

- TypeScript migration for stronger internal guarantees.

- Admin panel for browser-based inspection and dashboards.

- Docker packaging for isolated deployments.

- GitHub CI/CD workflows to automatically test, build, and release binaries on tag.

## THIS PROJECT IS FOR LEARNING

This engine is meant to teach real-world concepts behind database design, performance tradeoffs, indexing, concurrency, durability, benchmarking, SQL parsing, CLI ergonomics, and CI/CD automation. The goal is not production-grade perfection, but correctness, clarity, experimentation, and architectural evolution.
