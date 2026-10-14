# AI Product Factory PostgreSQL

This chart deploys one PostgreSQL 17 StatefulSet for AI Product Factory. It
uses an existing Kubernetes Secret and never stores passwords in Git.

Required Secret: `factory-postgresql-secrets`

| Key | Purpose |
|---|---|
| `postgres-password` | PostgreSQL bootstrap superuser |
| `migration-password` | `factory_migrator` login and database owner |
| `innovation-password` | `innovation_man` runtime login |
| `sales-password` | `sales_man` runtime login |
| `manager-password` | `manager_man` runtime login |

The initialization script creates the three group roles expected by the
factory migrations and grants them to the corresponding runtime logins. It
runs only when PostgreSQL initializes an empty data directory. Changing the
Secret later does not rotate passwords in an existing database.

Internal endpoint:

```text
factory-postgresql.ai-product-factory.svc.cluster.local:5432
```

The default `local-path` storage is node-local and does not provide database
high availability. Backups must be configured before production use.
