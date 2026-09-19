# Role Reference

This page describes each role in the pgEdge Ansible Collection and provides
links to detailed documentation for each role.

## Role Execution Order

Roles must be applied in a specific order to satisfy dependencies. The
following list shows the correct execution sequence:

1. [`init_server`](roles/init_server.md) - Prepares all servers in the
   cluster.
2. [`install_repos`](roles/install_repos.md) - Installs the pgEdge and PGDG
   package repositories.
3. [`install_pgedge`](roles/install_pgedge.md) - Installs pgEdge packages on
   pgEdge nodes.
4. [`setup_postgres`](roles/setup_postgres.md) - Initializes PostgreSQL on
   pgEdge nodes.
5. [`install_etcd`](roles/install_etcd.md) - Installs etcd on pgEdge nodes
   (HA only).
6. [`install_patroni`](roles/install_patroni.md) - Installs Patroni on pgEdge
   nodes (HA only).
7. [`install_backrest`](roles/install_backrest.md) - Installs PgBackRest on
   backup-capable nodes.
8. [`setup_etcd`](roles/setup_etcd.md) - Configures and starts etcd (HA only).
9. [`setup_patroni`](roles/setup_patroni.md) - Configures Patroni and starts
   the HA cluster (HA only).
10. [`setup_haproxy`](roles/setup_haproxy.md) - Configures HAProxy on haproxy
    nodes (HA only).
11. [`setup_pgedge`](roles/setup_pgedge.md) - Creates Spock nodes and
    establishes subscriptions.
12. [`setup_backrest`](roles/setup_backrest.md) - Configures PgBackRest and
    runs the first backup.

## Role Categories

The roles in this collection fall into four categories.

### Configuration Foundation

| Role | Description |
|------|-------------|
| [`role_config`](roles/role_config.md) | Provides shared variables and computed values to all other roles. You do not call this role directly. |

### Server Preparation

| Role | Description |
|------|-------------|
| [`init_server`](roles/init_server.md) | Initializes servers with required packages and system configuration. |
| [`install_repos`](roles/install_repos.md) | Configures the pgEdge and PGDG package repositories. |

### Software Installation

| Role | Description |
|------|-------------|
| [`install_pgedge`](roles/install_pgedge.md) | Installs Postgres with pgEdge extensions. |
| [`install_etcd`](roles/install_etcd.md) | Installs etcd for HA cluster coordination. |
| [`install_patroni`](roles/install_patroni.md) | Installs Patroni for HA management. |
| [`install_backrest`](roles/install_backrest.md) | Installs PgBackRest for backup and restore operations. |

### Service Configuration

| Role | Description |
|------|-------------|
| [`setup_postgres`](roles/setup_postgres.md) | Initializes and configures Postgres instances. |
| [`setup_etcd`](roles/setup_etcd.md) | Configures and starts etcd clusters. |
| [`setup_patroni`](roles/setup_patroni.md) | Configures and starts Patroni. |
| [`setup_haproxy`](roles/setup_haproxy.md) | Installs and configures HAProxy. |
| [`setup_pgedge`](roles/setup_pgedge.md) | Establishes Spock replication between nodes. |
| [`setup_backrest`](roles/setup_backrest.md) | Configures PgBackRest and schedules backups. |

### ColdFront Tiered Storage

These roles are optional and not part of the numbered execution order
above. `install_lakekeeper`/`setup_lakekeeper` build a dedicated
Lakekeeper host, independent of any pgEdge cluster.
`install_coldfront`/`setup_coldfront` add to the `pgedge` play, after
`setup_pgedge`, gated behind `coldfront_enabled`.

| Role | Description |
|------|-------------|
| [`install_lakekeeper`](roles/install_lakekeeper.md) | Installs Lakekeeper and its standalone Postgres instance. |
| [`setup_lakekeeper`](roles/setup_lakekeeper.md) | Configures Lakekeeper's catalog and bootstraps the warehouse. |
| [`install_coldfront`](roles/install_coldfront.md) | Installs the ColdFront tiered-storage extension. |
| [`setup_coldfront`](roles/setup_coldfront.md) | Configures the ColdFront tiered-storage extension. |

See the [ColdFront Configuration](configuration/coldfront.md) reference
for how these roles fit together.
