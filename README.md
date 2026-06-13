# Task 1: Local Highly-Available PostgreSQL Cluster with Docker

## Overview

This project sets up a local highly-available PostgreSQL cluster using Docker with 3 containers:

- 1 monitor node (`pg-monitor`)
- 2 database nodes (`pg-node1`, `pg-node2`)

Uses image: `livingdocs/postgres:15.3`

## Setup Instructions

### 1. Start the containers

```bash
docker compose up -d
```

### 2. Initialize the monitor

```bash
docker exec -it pg-monitor bash

pg_autoctl create monitor \
  --pgdata /var/lib/postgresql/monitor \
  --pgport 5432 \
  --hostname monitor \
  --auth trust \
  --ssl-self-signed

pg_autoctl run --pgdata /var/lib/postgresql/monitor &
```

### 3. Initialize node1

```bash
docker exec -it pg-node1 bash

pg_autoctl create postgres \
  --pgdata /var/lib/postgresql/data \
  --pgport 5432 \
  --hostname node1 \
  --auth trust \
  --ssl-self-signed \
  --monitor postgresql://autoctl_node@monitor:5432/pg_auto_failover?sslmode=require

pg_autoctl run --pgdata /var/lib/postgresql/data &
```

### 4. Initialize node2

```bash
docker exec -it pg-node2 bash

pg_autoctl create postgres \
  --pgdata /var/lib/postgresql/data \
  --pgport 5432 \
  --hostname node2 \
  --auth trust \
  --ssl-self-signed \
  --monitor postgresql://autoctl_node@monitor:5432/pg_auto_failover?sslmode=require

pg_autoctl run --pgdata /var/lib/postgresql/data &
```

## Cluster State

After initialization, verify the cluster state from one of the nodes:

```bash
docker exec -it pg-node1 bash
pg_autoctl show state
```

You should see the monitor and worker nodes registered and the failover state for the cluster.

## Notes

- The `monitor` service is exposed on host port `5432`.
- `node1` is mapped to host port `5433` and `node2` is mapped to host port `5434`.
- Each container starts with `sleep infinity` so you can connect and initialize the cluster manually.
