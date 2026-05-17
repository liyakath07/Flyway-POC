# Flyway POC – Database Migration Automation

# Objective

The objective of this POC is to automate database schema changes using Flyway by codifying database changes and enabling repeatable, version-controlled migrations that can later be integrated into CI/CD pipelines.

This approach eliminates manual database change execution and aligns database management with DevOps and Infrastructure-as-Code practices.

---

# Technology Stack

| Component           | Technology                     |
| ------------------- | ------------------------------ |
| Database            | PostgreSQL 15                  |
| Migration Tool      | Flyway OSS                     |
| Container Runtime   | Docker                         |
| Orchestration       | Docker Compose                 |
| Versioning Approach | SQL-based Versioned Migrations |

---

# Architecture

```text
Developer
   ↓
Creates SQL Migration
   ↓
Version Controlled in Git
   ↓
Flyway Executes Migration
   ↓
PostgreSQL Schema Updated
   ↓
Migration History Tracked
```

---

# Step 1 – PostgreSQL Setup Using Docker Compose

Created PostgreSQL container locally using Docker Compose.

## docker-compose.yml

```yaml
services:
  postgres:
    image: postgres:15
    container_name: flyway-postgres

    environment:
      POSTGRES_USER: flyway
      POSTGRES_PASSWORD: flyway123
      POSTGRES_DB: appdb

    ports:
      - "5432:5432"

    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

## Start PostgreSQL

```bash
docker compose up -d
```

## Verification

Verified:

* PostgreSQL container running successfully
* DB connectivity working properly
<img width="1914" height="654" alt="image" src="https://github.com/user-attachments/assets/2ebefe4f-2968-476f-8fb2-a9ce00a12b05" />

---

# Step 2 – Initial Database Migration

Created migration directory:

```bash
mkdir sql
```

Created first migration file:

```text
V1__create_users_table.sql
```

## Migration Script

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(100),
    email VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```
<img width="735" height="244" alt="image" src="https://github.com/user-attachments/assets/93681824-443a-43bb-bd0f-6f62cffb10ef" />

---

# Step 3 – Execute Flyway Migration

Pulled Flyway Docker image:

```bash
docker pull flyway/flyway
```

Executed migration:

```bash
docker run --rm \
  --network flyway-poc_default \
  -v $(pwd)/sql:/flyway/sql \
  flyway/flyway \
  -url=jdbc:postgresql://postgres:5432/appdb \
  -user=flyway \
  -password=flyway123 \
  migrate
```

## Flyway Actions Performed

Flyway automatically:

* connected to PostgreSQL
* scanned migration directory
* detected pending migration
* created flyway_schema_history table
* executed migration
* tracked migration metadata

<img width="1831" height="455" alt="image" src="https://github.com/user-attachments/assets/1fa21945-4124-451c-bbf6-003e587b4c14" />


---

# Step 4 – Schema Verification

Verified newly created schema objects.

## PostgreSQL Verification

```sql
\dt
```

Observed tables:

* users
* flyway_schema_history

## Users Table Structure

```sql
\d users
```

## Flyway History Verification

```sql
SELECT installed_rank, version, description, script, success
FROM flyway_schema_history;
```
<img width="1243" height="871" alt="image" src="https://github.com/user-attachments/assets/f2cfc2f2-a409-4dfd-a9cd-f39f9b4b61c4" />

---

# Step 5 – Idempotent Migration Behavior

Re-executed Flyway migration command without adding new migrations.

## Result

Flyway correctly detected:

* schema already up to date
* no pending migrations

Output:

```text
Schema "public" is up to date. No migration necessary.
```

## Observation

This demonstrates:

* idempotent migration execution
* safe repeated execution
* CI/CD compatibility
<img width="972" height="369" alt="image" src="https://github.com/user-attachments/assets/1cb15b6c-8d51-41f0-ab3b-18f936805306" />

---

# Step 6 – Incremental Schema Evolution

Created second migration:

```text
V2__add_phone_column.sql
```

## Migration Script

```sql
ALTER TABLE users
ADD COLUMN phone VARCHAR(20);
```

Executed Flyway migrate again.

## Result

Flyway:

* detected V1 already executed
* applied only V2 migration
* updated flyway_schema_history table

## Verification

Verified:

* phone column added successfully
* migration history updated

<img width="1815" height="411" alt="image" src="https://github.com/user-attachments/assets/dc5931a7-9ae2-49a2-84e9-ba5b2ef20aee" />

---

# Step 7 – Migration Failure Simulation

Created intentionally broken migration:

```text
V3__broken_migration.sql
```

## Broken SQL

```sql
ALTER TABL users
ADD COLUMN address VARCHAR(255);
```

## Result

Flyway failed migration execution due to SQL syntax error.

Observed:

* migration execution failure
* schema rollback
* no partial schema modification
* no failed version entry persisted

## Important Observation

PostgreSQL transactional DDL support enabled automatic rollback.

This demonstrated:

* safe migration failure handling
* transactional migration execution
* production-safe rollback behavior

<img width="1297" height="941" alt="image" src="https://github.com/user-attachments/assets/f46054ce-7063-474e-af86-86364b8c9dc3" />

---

# Step 8 – Recovery from Failed Migration

Corrected migration script:

```sql
ALTER TABLE users
ADD COLUMN address VARCHAR(255);
```

Re-executed Flyway migration.

## Result

Flyway:

* retried pending migration
* successfully applied V3
* updated migration history

## Verification

Verified:

* address column created
* migration history updated successfully
<img width="1316" height="414" alt="image" src="https://github.com/user-attachments/assets/37d11856-8cab-4239-92a2-1d12cbeceb3d" />

---

# Step 9 – Migration Immutability / Checksum Validation

Modified already executed migration:

```text
V1__create_users_table.sql
```

Added additional comment to simulate unauthorized modification.

Re-executed Flyway migrate.

## Result

Flyway validation failed with checksum mismatch error.

Observed:

* migration immutability enforcement
* checksum validation protection
* tamper detection capability

## Important Observation

Flyway prevents modification of already executed migrations, ensuring:

* schema consistency
* environment integrity
* reliable deployment history

<img width="1906" height="490" alt="image" src="https://github.com/user-attachments/assets/709cef1a-09bb-4877-90ae-5b740dd439a1" />

---

# Key Concepts Demonstrated

| Concept                         | Status    |
| ------------------------------- | --------- |
| Database-as-Code                | Completed |
| Versioned Schema Migrations     | Completed |
| Automated Migration Execution   | Completed |
| Incremental Schema Evolution    | Completed |
| Schema History Tracking         | Completed |
| Idempotent Migration Execution  | Completed |
| Transactional Rollback Handling | Completed |
| Migration Failure Recovery      | Completed |
| Checksum Validation             | Completed |
| Migration Immutability          | Completed |

---

# Benefits Observed

* Eliminates manual database change execution
* Enables version-controlled schema management
* Provides repeatable and automated deployments
* Prevents schema drift between environments
* Enables CI/CD integration readiness
* Provides migration audit history
* Detects unauthorized migration modifications
* Supports safe incremental schema evolution

---

# Current Limitation

Current POC execution is manual using Docker commands.

Next phase will focus on:

* CI/CD integration
* Jenkins pipeline automation
* Git-triggered migration execution
* Automated deployment workflows

---

# Proposed Next Phase

## CI/CD Integration Workflow

```text
Developer Pushes Migration
           ↓
Git Repository
           ↓
Jenkins Pipeline Trigger
           ↓
Flyway Validate
           ↓
Flyway Migrate
           ↓
Database Updated Automatically
```

---

# Conclusion

This POC successfully demonstrated database migration automation using Flyway with PostgreSQL.

Database schema changes were codified, version-controlled, validated, and executed automatically while maintaining migration history and schema integrity.

The implementation aligns with DevOps principles and provides a strong foundation for future CI/CD-based database deployment automation.
