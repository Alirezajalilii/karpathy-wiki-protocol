---
type: entity
domain: {domain-name}
status: draft
sources:
  - {source-file-path}
updated: YYYY-MM-DD
tags: []
---
# {EntityName}

## Schema

| Column | Type | Nullable | Default | Notes |
|--------|------|----------|---------|-------|
| id | UUID | NO | gen_random_uuid() | Primary key |
| created_at | timestamp | NO | NOW() | |
| updated_at | timestamp | NO | NOW() | |

## Relations

| Relation | Target | Type | FK Column |
|----------|--------|------|-----------|
| {name} | [[entity:{Target}]] | ManyToOne | {column} |

## Business Rules

1. {Rule. Source: {file:line}}

## Consumers

- **Endpoints**: [[endpoints:{domain}]] — {which endpoints use this entity}
- **Screens**: [[screens:{domain}]] — {which screens display this entity}
- **Other entities**: {which entities reference this one}

## Indexes

| Name | Columns | Type | Purpose |
|------|---------|------|---------|
| {name} | {columns} | {btree/unique/gin} | {why} |
