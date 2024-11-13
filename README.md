# Gedis KV

Gedis KV is a small Redis-like key-value database written in Go.

It exposes a Redis-compatible TCP protocol using `redcon`, stores data in memory, and persists write operations through an append-only binary log so the database can replay state on restart.

## Features

- Redis-style TCP server
- In-memory key-value storage
- Append-only binary log for SET and DEL recovery
- Thread-safe reads and writes with `sync.RWMutex`
- Basic TTL support for keys
- SET options: `NX`, `XX`, `EX`, `PX`, `EXAT`, `PXAT`
- Structured logging with Zap

## Supported Commands

- `PING`
- `QUIT`
- `SET key value`
- `GET key`
- `DEL key [key ...]`

## Requirements

- Go 1.18+
- Make

## Installation

```bash
git clone https://github.com/Subhasish-Behera/Gedis_KV.git
cd Gedis_KV
go mod download
```

## Build

```bash
make build
```

This creates the server binary at:

```text
./bin/server
```

## Run

```bash
make run
```

By default, the server listens on:

```text
localhost:3000
```

and stores its append-only log at:

```text
default.db
```

You can also pass custom flags directly:

```bash
./bin/server -addr localhost:3000 -db-path ./gedis.db
```

## Usage

Using `redis-cli`:

```bash
redis-cli -p 3000
```

```redis
PING
SET name gedis
GET name
DEL name
QUIT
```

Example with TTL:

```redis
SET session abc123 EX 60
GET session
```

Example with conditional writes:

```redis
SET user alice NX
SET user bob XX
```

## Persistence Model

Gedis KV keeps the active dataset in memory. Mutating commands are written to an append-only binary log:

- `SET` appends a set event
- `DEL` appends a delete event
- On startup, the database replays the log to rebuild state

The binlog is flushed on close and periodically synced in the background.

## Project Layout

```text
cmd/
  main.go              # server entrypoint

pkg/
  server.go            # Redis protocol command handling
  db.go                # in-memory database and persistence replay
  binlog.go            # append-only log lifecycle
  binlog_reader.go     # binary log reader
  binlog_writer.go     # binary log writer
  db_test.go           # database tests
```

## Testing

```bash
make test
```

or:

```bash
go test ./...
```
