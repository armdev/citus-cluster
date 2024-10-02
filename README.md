Here's an improved and detailed `README.md` file for your Citus cluster setup, including necessary explanations, detailed steps, and additional comments.

---

# Citus Cluster

This project sets up a distributed PostgreSQL cluster using [Citus](https://citusdata.com/), with a coordinator and two worker nodes, managed via Docker Compose. Citus transforms PostgreSQL into a horizontally scalable database, enabling high performance for large-scale workloads.

## Prerequisites

- Docker
- Docker Compose

## Getting Started

Follow the steps below to start your Citus cluster with two worker nodes and run distributed queries.

### 1. Build and Start the Cluster

Run the following command to build and start the Citus coordinator and worker nodes using Docker Compose:

```bash
docker compose --compatibility up -d --build
```

This command will:
- Build the Docker images (if not already built).
- Start the Citus coordinator (`coordinator`) and two worker nodes (`worker1`, `worker2`) in detached mode (`-d`).
- Ensure compatibility with resource limits if you're running on constrained environments.

### 2. Connect to the Coordinator

To connect to the PostgreSQL database running on the coordinator node:

```bash
docker exec -it coordinator psql -U postgres
```

This command opens a `psql` shell inside the `coordinator` container, where you can manage and distribute your database.

### 3. Add Workers to the Citus Cluster

Once connected to the coordinator, you need to add the two worker nodes (`worker1`, `worker2`) to the cluster:

```sql
-- Add worker1 to the cluster
SELECT * FROM master_add_node('worker1', 5432);

-- Add worker2 to the cluster
SELECT * FROM master_add_node('worker2', 5432);
```

These commands will register the worker nodes with the coordinator, enabling the cluster to distribute data and queries.

### 4. Verify Worker Nodes

To ensure the worker nodes are correctly added and active, run:

```sql
SELECT * FROM master_get_active_worker_nodes();
```

This will return a list of all active worker nodes in the Citus cluster.

### 5. Access pgAdmin

You can manage and monitor your PostgreSQL databases, including the Citus cluster, using pgAdmin. It is accessible at:

- **URL**: [http://localhost:5050](http://localhost:5050)

Create connections for the following:
- Coordinator (`coordinator`)
- Worker 1 (`worker1`)
- Worker 2 (`worker2`)

### 6. Create and Distribute a Table

In the coordinator node, create a sample table called `events` and distribute it across the worker nodes:

```sql
-- Create the events table on the coordinator
CREATE TABLE events (
  device_id bigint,
  event_id bigserial,
  event_time timestamptz DEFAULT now(),
  data jsonb NOT NULL,
  PRIMARY KEY (device_id, event_id)
);

-- Distribute the events table by the device_id column
SELECT create_distributed_table('events', 'device_id');
```

This will distribute the `events` table across shards placed on both worker nodes.

### 7. Insert Data

You can insert sample data into the `events` table, which will be automatically sharded and distributed across the workers:

```sql
-- Insert 1,000,000 records with random data
INSERT INTO events (device_id, data)
SELECT s % 100, ('{"measurement":'||random()||'}')::jsonb
FROM generate_series(1, 1000000) s;
```

This query generates 1,000,000 rows of random JSON data, distributing the rows based on the `device_id` across the worker nodes.

### 8. Check Data on Workers

After inserting the data, you can connect to each worker (`worker1` and `worker2`) to verify the distribution of the `events` table data.

Use `psql` or pgAdmin to run queries on each worker node to confirm that the data has been correctly distributed.

### Additional Information

For more information about Citus and sharding with PostgreSQL, refer to the following resources:

- [Citus GitHub Repository](https://github.com/citusdata/citus/)
- [Citus: Sharding Your First Table](https://www.cybertec-postgresql.com/en/citus-sharding-your-first-table/)

---

This `README.md` provides detailed steps to set up a Citus cluster using Docker Compose, distribute a table across worker nodes, and insert and verify data.