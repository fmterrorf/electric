---
title: Postgres
description: >-
  With logical replication enabled.
sidebar_position: 30
---

ElectricSQL requires a [PostgreSQL](https://www.postgresql.org/download) database. Postgres is the world's most advanced open source relational database.

## Compatibility

ElectricSQL works with standard Postgres [version >= 13.0](https://www.postgresql.org/support/versioning/) with [logical&nbsp;replication](https://www.postgresql.org/docs/current/logical-replication.html) enabled. Note that you don't need to install any extensions or run any unsafe code.

## Hosting

Many managed hosting providers support logical replication (either out of the box or as an option to enable).

This includes, for example:

- [AWS RDS](https://repost.aws/knowledge-center/rds-postgresql-use-logical-replication) and [Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.Replication.Logical.html) (including Aurora Serverless v2)
- [Crunchy Data](https://www.crunchydata.com) who have a free tier and logical replication enabled by default

There's a [long list of Postgres hosting providers here](https://www.postgresql.org/support/professional_hosting/).

### Self-host / run locally

You can run your own Postgres anywhere you like. See the [Postgres Server Administration](https://www.postgresql.org/docs/current/admin.html) docs for more information.

#### Docker

For example, to run using Docker:

```shell
docker run \
    -e "POSTGRES_PASSWORD=..." \
    postgres -c "wal_level=logical"
```

#### Homebrew

To run locally using Homebrew, first install and start the service:

```shell
brew install postgresql
brew services start postgresql
```

Enable logical replication:

```sql
psql -U postgres \
    -c 'ALTER SYSTEM SET wal_level = logical'
brew services restart postgresql
```

Verify the wal_level is logical:

```shell
psql -U postgres -c 'show wal_level'
```

## Permissions

The [Electric sync service](./service.md) currently needs to connect to Postgres as a database user with `LOGIN` and `SUPERUSER` permissions.

For example:

```sql
CREATE ROLE electric
  WITH LOGIN
       PASSWORD '...'
       SUPERUSER;
```

Note that we are working to remove the `SUPERUSER` requirement.

<details>
  <summary>
    More details on future permissions
  </summary>
  <div>

In future, the permissions required will be a minimum of:

- `LOGIN`
- `REPLICATION`

And then either `ALL` on the database and public schema or at a minimum:

- `CONNECT`, `CREATE` and `TEMPORARY` on the database
- `CREATE`, `EXECUTE on ALL` and `USAGE` on the `public` schema

Plus `ALTER DEFAULT PRIVILEGES` to grant the same permissions on any new tables in the public schema.

For example, to create an `electric` user with the necessary permissions:

```sql
CREATE ROLE electric
  WITH LOGIN
    PASSWORD '...'
    REPLICATION;

GRANT ALL
  ON DATABASE '...' 
  TO electric;

GRANT ALL 
  ON ALL TABLES 
  IN SCHEMA public 
  TO electric;

ALTER DEFAULT PRIVILEGES 
  IN SCHEMA public 
  GRANT ALL 
    ON TABLES 
    TO electric;
```

This will remove the need for `SUPERUSER`, which will increase the compatibility with hosting providers such as [Cloud SQL](https://cloud.google.com/sql) and [AlloyDB](https://cloud.google.com/alloydb) that won't grant superuser (or a reduced version like [`rds_superuser`](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.PostgreSQL.CommonDBATasks.Roles.html#Appendix.PostgreSQL.CommonDBATasks.Roles.rds_superuser)).

  </div>
</details>

