---
name: db-manual-sql-script
description: 'Best practices for manual SQL migration scripts, including backup/rollback requirements, migration verification, and data integrity guidelines.'
---

# Manual SQL Script Best Practices Skill

## Skill Metadata
- **Name**: db-manual-sql-script
- **Domain**: Manual SQL scripts, database migrations
- **Purpose**: Provides comprehensive best practices for manual SQL scripts executed directly on the database

## When to Use This Skill

### Auto-Trigger Conditions (file patterns):
- `**/*.sql` - Manual SQL migration scripts

### Manual Trigger Keywords:
- "database best practices"
- "DB guidelines"
- "migration script"
- "manual script"
- "rollback script"
- "backup table"
- "TRUNCATE TABLE"

### Use During:
- Code review of database changes
- Planning database migrations
- Creating manual SQL scripts
- Cleanup scripts

**Note**: This skill focuses on **manual SQL migration scripts**. For automated Liquibase XML changesets, see the `liquibase-script` skill.

---

## Best Practices for Manual Scripts

### Repository Location
Organize manual scripts in a consistent folder structure, for example: `scripts/db/manual/`

### 1. Folder Structure
Split scripts in different folders, according to the microservice/module they belong to. Inside each module, split based on the release they belong to.

**Example**: `db-manual-migration/<service>/R26.Q1/`

### 2. Naming Convention
Format: `ScriptNO_TICKET-xxxxx_Short_Script_Description.sql`

**Special characters are not allowed.**

**Example**: `13_TICKET-67217_currencyDataMigration.sql`

### 3. Three-Section Script Structure
Each script must have at least 3 sections:

**a) Initial validation** — Returns the number of rows that should be affected, or checks the DB state is as expected.

**b) Migration** — SQL instructions that achieve the purpose of the script.

**c) Final validation** — A SELECT statement that checks if the migration was executed successfully. This should be easy to understand and have a comment explaining the expected output.

### 4. Reset ID / Sequences After Manual Inserts
After manual inserts, reset the ID or sequence of the affected tables to avoid primary key conflicts with the application.

### 5. Mandatory Rollback Scripts
Each migration script must have a rollback that can bring the DB to the exact state it was before.

**Naming**: Same as migration + `_rollback` at the end.

**Example**:
- Migration: `13_TICKET-67217_currencyDataMigration.sql`
- Rollback: `13_TICKET-67217_currencyDataMigration_rollback.sql`

**Exceptions**: Cleanup scripts where rollback is not meaningful.

### 6. Create Backup Tables Before Updates
Create backup tables before executing an update that modifies existing data. Include the ticket number in the name of the backup table.

**No backup needed** when:
- Updating from well-known default values (empty string, true, false, 0)
- Rollback is straightforward without a backup

### 7. Update README with Migration Status
Add new migrations to the `README.md` file from the same folder and **keep the execution status up to date**.

### 8. TRUNCATE vs DELETE
Use `TRUNCATE TABLE` (if supported) instead of `DELETE` if the entire content of a table needs to be cleaned up.

**⚠️ WARNING**: A truncate usually **cannot be rolled back** (depends on database). The data deleted is often not in the transaction log.

### 9. REORG / Rebuild After ALTER TABLE (Database-Specific)
For databases that require a reorganize/rebuild operation after altering columns (e.g., DB2), include it in the script.

### 10. Batching for Large Data Changes
For transactions involving a large amount of data, use procedures and split the operations into smaller transactions (batches) to avoid locking and timeout issues.

## Error Handling
Search for the error code in the official documentation for your database engine.

## Common Pitfalls
- Missing rollback scripts
- Not validating before and after migration
- Not resetting sequences/IDs after manual inserts
- Using DELETE when TRUNCATE would be safer and faster (for full table cleanup)
- Not batching large data changes
