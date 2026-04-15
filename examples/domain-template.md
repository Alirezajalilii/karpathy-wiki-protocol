---
type: domain
domain: {domain-name}
status: draft
sources:
  - {source-file-path}
updated: YYYY-MM-DD
tags: []
---
# {Domain Name}

## Overview

{1-2 sentence description of what this domain does.}

## Entities

| Entity | Status | Page |
|--------|--------|------|
| {EntityName} | {implemented/partial/missing} | [[entity:{EntityName}]] |

## Endpoints

See [[endpoints:{domain-name}]] for full endpoint documentation.

| Method | Path | Status |
|--------|------|--------|
| GET | /v2/{domain} | {implemented/stub/missing} |

## Screens

See [[screens:{domain-name}]] for full screen documentation.

| Screen | Status |
|--------|--------|
| {ScreenName} | {implemented/partial/missing} |

## Business Rules

1. {Rule description. Source: {file:line}}

## State Machine

```
{state} → {action} → {state}
```

## Open Gaps

| ID | Severity | Description | Status |
|----|----------|-------------|--------|
| {N} | 🔴/🟡/🔵 | {description} | Open |

## Dependencies

- Depends on: [[domain:{other}]]
- Depended on by: [[domain:{other}]]
