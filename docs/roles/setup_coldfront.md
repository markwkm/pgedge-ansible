# setup_coldfront

The `setup_coldfront` role configures Postgres for the ColdFront extension,
creates the additional database ColdFront's tiered storage lives in, and
points it at the Lakekeeper catalog and the S3-compatible object store
behind it.

The role performs the following tasks on inventory hosts:

- Check that the object store and the Lakekeeper catalog are reachable
  from this host, since both are used at query time, not just at setup
  time.
- Create the `coldfront_duckdb_role` NOLOGIN role and the `coldfront_db`
  database.
- Append ColdFront's Postgres configuration — extending
  `shared_preload_libraries` rather than overwriting it — and restart
  Postgres only when that configuration actually changed.
- Create the `pg_duckdb` and `coldfront` extensions in `coldfront_db`.
- Call `coldfront.set_storage_secret()` with the configured object-store
  credentials, an idempotent upsert run on every pass.
- Write the archiver, partitioner, and compactor configuration to
  `/etc/pgedge/coldfront/config.yaml`.

## Role Dependencies

This role requires the following roles for normal operation:

- `role_config` provides shared configuration variables to the role.
- `install_coldfront` installs the ColdFront packages.
- `setup_postgres` and `setup_pgedge` bring up the pgEdge Distributed
  Postgres node this role installs into.

## When to Use

Execute this role immediately after `install_coldfront`, gated behind the
same `coldfront_enabled` condition:

```yaml
- hosts: pgedge
  collections:
    - pgedge.platform
  roles:
    - setup_pgedge
    - role: install_coldfront
      when: coldfront_enabled
    - role: setup_coldfront
      when: coldfront_enabled
```

## Configuration

This role uses the following parameters from the inventory file, in
addition to those `install_coldfront` reads:

| Parameter | Use Case |
|-----------|----------|
| `lakekeeper_host` / `lakekeeper_port` / `lakekeeper_warehouse` | Locates the Iceberg catalog this node writes through. |
| `coldfront_s3_*` | Locates and authenticates to the S3-compatible object store. |
| `coldfront_mesh` | Whether this node is part of a Spock mesh (see below). Defaults to `false`. |
| `pgedge_preload_libraries` | The base `shared_preload_libraries` list this role extends rather than overwrites. |
| `db_user` / `db_password` | The cluster's existing admin user, reused as ColdFront's own database connection identity rather than a bespoke role. |

See the [ColdFront Configuration](../configuration/coldfront.md) reference
for descriptions and defaults.

## How It Works

### Extending, not overwriting, shared_preload_libraries

This role appends its own `postgresql.conf` block after `setup_postgres`'s
own block, using a distinct marker so the two coexist, and computes
`shared_preload_libraries` as `pgedge_preload_libraries` (Spock, Snowflake,
and `pg_stat_statements` by default) plus `pg_duckdb` and `coldfront`, so
enabling ColdFront never silently disables Spock replication.

### Vanilla and mesh modes

ColdFront upstream distinguishes a single-node "vanilla" mode, with a
local advisory-lock bakery, from an N-node Spock "mesh" mode that
coordinates cold-tier commits with the Ricart-Agrawala protocol instead.
`coldfront_mesh: false` (the default) matches vanilla: no
`coldfront.dblink_self` DSN is set, since that GUC is only meaningful once
Spock actually names a peer to negotiate with. Getting the vanilla
configuration solid is the current focus; setting `coldfront_mesh: true`,
and everything a real multi-zone Spock mesh needs beyond it, is follow-up
work not yet validated against a live multi-node cluster.

### DuckDB extension loading

`duckdb.autoinstall_known_extensions` is set to `false`, per the
`pgedge-coldfront-duckdb-extensions` package's own shipped sample config
(`coldfront-duckdb-extensions.conf.sample`). With it `true`, a missing
extension file makes DuckDB silently fetch the unpatched upstream build,
which 409s under concurrency and writes manifests strict Iceberg readers
reject.

## Artifacts

| File | New / Modified | Explanation |
|------|----------------|-------------|
| `/etc/pgedge/coldfront/config.yaml` | New | Archiver, partitioner, and compactor configuration: the Postgres DSN, the Iceberg catalog endpoint, and the S3 connection. |

## Idempotency

This role is idempotent and safe to re-run on inventory hosts. Database,
role, and extension creation are all no-ops once already done. The
Postgres configuration block only triggers a restart when its content
actually changes.
