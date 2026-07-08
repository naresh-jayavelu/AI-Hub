---
name: liquibase-script
description: 'Liquibase best practices for changesets, backup/rollback requirements, replaceable scripts, changeset IDs, and database schema change guidelines.'
---

# Liquibase Best Practices Skill

## Skill Metadata
- **Name**: liquibase-script
- **Domain**: Liquibase, database schema migrations
- **Purpose**: Provides comprehensive Liquibase best practices for database change management

## When to Use This Skill

### Auto-Trigger Conditions (file patterns):
- `**/liquibase/**/*.xml` - Liquibase changesets
- `**/*-dbsetup/**` - Database setup modules

### Manual Trigger Keywords:
- "liquibase best practices"
- "liquibase guidelines"
- "table adjustment"
- "liquibase script"
- "changeset"
- "databaseChangeLog"

### Use During:
- Code review of database changes involving structural updates
- Create, update, delete operations on database objects (tables, views, procedures)
- Writing Liquibase changesets
- Database schema changes

---

## Liquibase Version
Refer to your project's `pom.xml` or `build.gradle` for the configured Liquibase version. Official documentation: https://docs.liquibase.com

## Best Practices for Liquibase Scripts

### 1. Naming Convention
Format: `TICKET-xxxxx-Short-Script-Description.xml`

**Special characters are not allowed.**

**Note**: Ticket number can be omitted for **replaceable** scripts.

### 2. Release Folder Structure
Organize Liquibase scripts by release:
- `25.1.0/`
- `25.2.0/`
- `replaceable/` (for views, procedures, triggers, recurrent grants)

Create a new folder when development for the next release begins.

### 3. Replaceable Scripts
Use the `replaceable/` folder for objects that can change over time:
- Views
- Stored procedures
- Triggers
- Recurrent grants (automatic permissions)

**Required properties**:
```xml
<changeSet id="2026-02-02-14.00" author="AUTHOR" runOnChange="true" >
    <createView replaceIfExists="true" schemaName="..." viewName="...">
        ...
    </createView>
</changeSet>
```
- `runOnChange="true"` - Rerun changeset when content changes
- `replaceIfExists="true"` - Replace existing object

**Recurrent calls** (grants):
```xml
<changeSet id="..." author="..." runAlways="true" runOnChange="true">
    <sql>GRANT SELECT ON ... TO ...;</sql>
</changeSet>
```
- `runAlways="true"` - Run at every deployment
- ⚠️ Use carefully! Most scripts don't need to run at each deployment.

### 4. Unique Changeset IDs
The `id` attribute must be:
- **Unique** across all changesets
- **Updated before commit**

**Format example**: `<changeSet id="2026-02-02-14.00" author="AUTHOR">`

**For replaceable scripts**: Update `id` and `author` every time the SQL/content changes.

### 5. No preConditions Tag
**Do not use `preConditions` tag** - it can hide execution errors.

### 6. Rollback Tags Required
Add `<rollback>` tag for all changes.

**Exceptions** (auto-rollback available in Liquibase):
- Create table
- Create view
- Operations with built-in Liquibase rollback support (see: https://docs.liquibase.com/change-types)

### 7. Column Length Considerations
When using databases that encode characters differently (e.g., multi-byte encodings), ensure column lengths account for the byte multiplier for the target character encoding.

### 8. Add Descriptive Comments
Use `<comment>` tags to clearly describe the purpose of instructions.

### 9. No Changes After Execution
The script committed must be **exactly the same** as executed on the dev/staging database.

**Rule**: Do not modify script name or content after execution on a shared environment.

**If updates needed**: Perform DB cleanup and rerun the script.

### 10. Verify Pod Execution
Always check that the migration job/pod has no errors after running migrations.

### 11. Document Successful Execution
When opening a PR with DB changes, include a screenshot or log showing successful execution on the dev database.

## Error Handling
Search for database error codes in the official documentation for your database engine.

## Common Liquibase Pitfalls / Mistakes to Avoid
- Liquibase scripts with Windows line endings (`\r\n`) causing runtime failures on Linux.
- Changeset IDs that are not unique across the entire changelog.
- Using `preConditions` which can mask errors.
- Modifying a changeset after it has been applied to a shared environment.
