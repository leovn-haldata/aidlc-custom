# Data Model: <change-id>

## Change Reference
- **Design**: [design.md](./design.md)
- **Mode**: full-spec

## Entity Relationships

_Describe the entities and their relationships (1:1, 1:N, N:M)._

### Entity: _name_

| Field | Type | Constraints | Description |
|-------|------|------------|-------------|
| id | UUID / SERIAL | PK, NOT NULL | Primary key |
| _field_ | _type_ | _constraints_ | _description_ |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation timestamp |
| updated_at | TIMESTAMP | NOT NULL | Last update timestamp |

**Relationships**:
- Has many _OtherEntity_ (via _foreign_key_)

### Entity: _name_

| Field | Type | Constraints | Description |
|-------|------|------------|-------------|
| id | UUID / SERIAL | PK, NOT NULL | Primary key |
| _parent_id_ | UUID | FK → _Parent_.id, NOT NULL | Foreign key |

## Indexes

| Table | Index Name | Columns | Type | Rationale |
|-------|-----------|---------|------|-----------|
| _table_ | _idx_name_ | _col1, col2_ | BTREE / GIN | _Why needed_ |

## Migration Plan

### Up Migration
```sql
-- Migration: <change-id>_up
-- Description: _what this migration does_
```

### Down Migration
```sql
-- Migration: <change-id>_down
-- Description: _how to revert_
```

## Data Validation Rules

| Entity | Field | Rule | Error Message |
|--------|-------|------|---------------|
| _entity_ | _field_ | _validation rule_ | _message_ |

## Seed Data (if applicable)

_Any initial data required for the feature to function._
