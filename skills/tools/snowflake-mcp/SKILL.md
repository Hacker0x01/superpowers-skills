---
name: Snowflake MCP Best Practices
description: Safe and efficient patterns for interacting with Snowflake via MCP tools
when_to_use: when using any Snowflake MCP tool (mcp__snowflake__*) for queries, object management, or data exploration
version: 1.0.0
---

# Snowflake MCP Best Practices

## Overview

Snowflake queries consume compute credits and can return massive datasets. Always use limits, filters, and specialized tools to minimize cost and response size.

## Critical Rules

### 1. Always Use LIMIT Unless User Explicitly Requests Full Results

**Default LIMIT: 100 rows**

```ruby
# ✅ CORRECT - Always include LIMIT
mcp__snowflake__run_snowflake_query(
  statement: "SELECT * FROM large_table LIMIT 100"
)

mcp__snowflake__query_semantic_view(
  database_name: "analytics",
  schema_name: "core",
  view_name: "metrics",
  metrics: [...],
  limit: 100
)

# ❌ WRONG - Unbounded query
mcp__snowflake__run_snowflake_query(
  statement: "SELECT * FROM large_table"
)
```

**When to use different limits:**
- Exploratory queries: LIMIT 10-20
- Data validation: LIMIT 100 (default)
- Reports: LIMIT 1000
- User explicitly requests "all data": No limit, but confirm first

### 2. Use Specialized Tools Over Raw SQL

**Tool Priority:**
1. Specialized object tools (`describe_object`, `list_objects`)
2. Semantic view tools (`query_semantic_view`)
3. Raw SQL (`run_snowflake_query`) as last resort

```ruby
# ✅ CORRECT - Use specialized tool
mcp__snowflake__list_objects(
  object_type: "table",
  database_name: "analytics",
  like: "user%"
)

# ❌ WRONG - Raw SQL for metadata
mcp__snowflake__run_snowflake_query(
  statement: "SHOW TABLES LIKE 'user%' IN analytics"
)
```

**Why:**
- Type safety and validation
- Better error messages
- Automatic result formatting
- Cost-optimized queries

### 3. Filter Early, Select Late

```ruby
# ✅ CORRECT - WHERE before SELECT, minimal columns
mcp__snowflake__run_snowflake_query(
  statement: <<~SQL
    SELECT user_id, transaction_date, amount
    FROM transactions
    WHERE transaction_date >= DATEADD(day, -7, CURRENT_DATE)
      AND amount > 100
    LIMIT 100
  SQL
)

# ❌ WRONG - SELECT * then filter
mcp__snowflake__run_snowflake_query(
  statement: <<~SQL
    SELECT * FROM transactions
    WHERE transaction_date >= DATEADD(day, -7, CURRENT_DATE)
    LIMIT 100
  SQL
)
```

**Filter checklist:**
- ✅ WHERE clause to reduce scanned rows
- ✅ Specific columns (not SELECT *)
- ✅ Date range filters on partitioned columns
- ✅ LIMIT clause

### 4. Use Filters on List Operations

```ruby
# ✅ CORRECT - Use like/starts_with
mcp__snowflake__list_objects(
  object_type: "table",
  database_name: "analytics",
  starts_with: "fact_"  # or like: "fact%"
)

# ❌ WRONG - List everything then filter in code
mcp__snowflake__list_objects(
  object_type: "table",
  database_name: "analytics"
)
# ... then filtering results in code
```

## Quick Reference

### Querying Data

```ruby
# Exploratory query (10-20 rows)
mcp__snowflake__run_snowflake_query(
  statement: "SELECT * FROM users LIMIT 10"
)

# Data validation (100 rows default)
mcp__snowflake__run_snowflake_query(
  statement: <<~SQL
    SELECT user_id, email, created_at
    FROM users
    WHERE created_at >= CURRENT_DATE - 30
    LIMIT 100
  SQL
)

# Semantic view query (always use limit)
mcp__snowflake__query_semantic_view(
  database_name: "analytics",
  schema_name: "metrics",
  view_name: "user_engagement",
  dimensions: [
    {table: "users", name: "user_id"},
    {table: "users", name: "cohort"}
  ],
  metrics: [
    {table: "engagement", name: "daily_active"}
  ],
  where_clause: "cohort = '2024-Q1'",
  limit: 100
)
```

### Object Management

```ruby
# List with filters
mcp__snowflake__list_objects(
  object_type: "table",
  database_name: "analytics",
  schema_name: "core",
  starts_with: "dim_"  # Filter by prefix
)

# Describe specific object
mcp__snowflake__describe_object(
  object_type: "table",
  target_object: {
    database_name: "analytics",
    schema_name: "core",
    name: "fact_transactions"
  }
)

# Create with explicit mode
mcp__snowflake__create_object(
  object_type: "table",
  mode: "if_not_exists",  # or "error_if_exists", "replace"
  target_object: {
    database_name: "analytics",
    schema_name: "staging",
    name: "temp_analysis",
    columns: [
      {name: "id", datatype: "NUMBER", nullable: false},
      {name: "data", datatype: "VARCHAR", nullable: true}
    ]
  }
)
```

### Semantic Views

```ruby
# Always check what's available first
mcp__snowflake__list_semantic_views(
  database_name: "analytics",
  schema_name: "metrics",
  like: "user"  # Filter by keyword
)

# Describe before querying
mcp__snowflake__describe_semantic_view(
  database_name: "analytics",
  schema_name: "metrics",
  view_name: "user_metrics"
)

# Show available dimensions and metrics
mcp__snowflake__show_semantic_dimensions(
  database_name: "analytics",
  schema_name: "metrics",
  view_name: "user_metrics"
)

mcp__snowflake__show_semantic_metrics(
  database_name: "analytics",
  schema_name: "metrics",
  view_name: "user_metrics"
)
```

## Tool-Specific Patterns

### run_snowflake_query

**Use when:**
- Need complex SQL features
- Specialized tools don't cover use case
- Writing data (INSERT, UPDATE, DELETE)

**Always include:**
```ruby
mcp__snowflake__run_snowflake_query(
  statement: <<~SQL
    SELECT column1, column2, column3  -- Specific columns
    FROM table_name
    WHERE date_column >= CURRENT_DATE - 7  -- Filter early
      AND other_condition
    ORDER BY column1  -- Optional
    LIMIT 100  -- MANDATORY unless user requests full results
  SQL
)
```

**Cost considerations:**
- Each query consumes warehouse compute credits
- Larger result sets = more data transfer costs
- Unbounded queries can run for minutes and cost dollars

### query_semantic_view

**Use when:**
- Semantic views exist for your use case
- Need pre-defined metrics and dimensions
- Want optimized, validated queries

**Pattern:**
```ruby
# 1. Discover available views
views = mcp__snowflake__list_semantic_views(
  database_name: "analytics",
  like: "keyword"
)

# 2. Describe to understand structure
structure = mcp__snowflake__describe_semantic_view(
  database_name: "analytics",
  schema_name: "core",
  view_name: "chosen_view"
)

# 3. Query with limit
result = mcp__snowflake__query_semantic_view(
  database_name: "analytics",
  schema_name: "core",
  view_name: "chosen_view",
  dimensions: [{table: "t1", name: "dim1"}],
  metrics: [{table: "t1", name: "metric1"}],
  limit: 100  # Always include
)
```

**Cannot combine FACTS and METRICS in same query** - they're mutually exclusive.

### list_objects

**Use when:**
- Discovering available databases, schemas, tables
- Finding objects matching patterns

**Always filter:**
```ruby
# ✅ Good - Filter with starts_with or like
mcp__snowflake__list_objects(
  object_type: "table",
  database_name: "analytics",
  starts_with: "fact_"
)

# ⚠️ Acceptable - List all (but be aware of scale)
mcp__snowflake__list_objects(
  object_type: "database"
)

# ❌ Bad - List all tables in large database
mcp__snowflake__list_objects(
  object_type: "table",
  database_name: "huge_production_db"
)
```

### create_object / create_or_alter_object

**Use when:**
- Creating databases, schemas, tables, views
- Need programmatic DDL

**Choose mode carefully:**
```ruby
mcp__snowflake__create_object(
  object_type: "table",
  mode: "if_not_exists",  # Safe: only create if missing
  # mode: "error_if_exists",  # Fail if exists (default)
  # mode: "replace",  # DANGEROUS: drops and recreates
  target_object: {...}
)
```

**Always use `create_or_alter_object` for idempotent operations:**
```ruby
mcp__snowflake__create_or_alter_object(
  object_type: "warehouse",
  target_object: {
    name: "analytics_wh",
    warehouse_size: "SMALL",
    auto_suspend: 300
  }
)
```

## Common Pitfalls

### 1. Unbounded Queries

```ruby
# ❌ NEVER DO THIS
mcp__snowflake__run_snowflake_query(
  statement: "SELECT * FROM billion_row_table"
)

# ✅ ALWAYS ADD LIMIT
mcp__snowflake__run_snowflake_query(
  statement: "SELECT * FROM billion_row_table LIMIT 100"
)
```

**Cost:** Unbounded query on 1B rows could:
- Run for 10+ minutes
- Cost $50+ in compute credits
- Return gigabytes of data
- Timeout or crash MCP connection

### 2. Using Raw SQL for Metadata

```ruby
# ❌ Wrong tool for the job
mcp__snowflake__run_snowflake_query(
  statement: "SHOW TABLES IN analytics.core"
)

# ✅ Use specialized tool
mcp__snowflake__list_objects(
  object_type: "table",
  database_name: "analytics",
  schema_name: "core"
)
```

### 3. SELECT * Instead of Specific Columns

```ruby
# ❌ Wasteful
mcp__snowflake__run_snowflake_query(
  statement: "SELECT * FROM wide_table LIMIT 100"
)

# ✅ Efficient
mcp__snowflake__run_snowflake_query(
  statement: "SELECT user_id, name, email FROM wide_table LIMIT 100"
)
```

**Why it matters:**
- 100 columns × 100 rows = 10,000 values transferred
- 3 columns × 100 rows = 300 values transferred
- 33× less data = 33× faster + cheaper

### 4. No WHERE Clause on Large Tables

```ruby
# ❌ Scans entire table
mcp__snowflake__run_snowflake_query(
  statement: "SELECT user_id FROM users LIMIT 100"
)

# ✅ Filters before selecting
mcp__snowflake__run_snowflake_query(
  statement: <<~SQL
    SELECT user_id
    FROM users
    WHERE created_at >= CURRENT_DATE - 30
    LIMIT 100
  SQL
)
```

### 5. Not Checking Object Existence Before Create

```ruby
# ❌ Will fail if table exists
mcp__snowflake__create_object(
  object_type: "table",
  target_object: {...}
)

# ✅ Safe creation
mcp__snowflake__create_object(
  object_type: "table",
  mode: "if_not_exists",
  target_object: {...}
)

# ✅ Or use upsert
mcp__snowflake__create_or_alter_object(
  object_type: "table",
  target_object: {...}
)
```

### 6. Forgetting Semantic View Constraints

```ruby
# ❌ Will fail - cannot mix facts and metrics
mcp__snowflake__query_semantic_view(
  database_name: "analytics",
  schema_name: "core",
  view_name: "metrics",
  facts: [{table: "t1", name: "fact1"}],
  metrics: [{table: "t1", name: "metric1"}],  # ERROR
  limit: 100
)

# ✅ Choose one or the other
mcp__snowflake__query_semantic_view(
  database_name: "analytics",
  schema_name: "core",
  view_name: "metrics",
  dimensions: [{table: "t1", name: "dim1"}],
  metrics: [{table: "t1", name: "metric1"}],
  limit: 100
)
```

## Decision Tree

**Need to query data?**
1. Is there a semantic view? → `query_semantic_view` (with limit)
2. Need complex SQL? → `run_snowflake_query` (with LIMIT clause)
3. Exploring data? → `run_snowflake_query` with LIMIT 10-20

**Need object metadata?**
1. List objects? → `list_objects` (with filters)
2. Describe specific object? → `describe_object`
3. Get DDL? → `get_semantic_view_ddl`

**Need to create/modify objects?**
1. Idempotent operation? → `create_or_alter_object`
2. Safe creation? → `create_object` with `mode: "if_not_exists"`
3. Complex DDL? → `run_snowflake_query` (DDL statement)

**Always ask yourself:**
- Do I have a LIMIT clause? (if querying data)
- Do I have WHERE filters? (if large table)
- Am I using the most specific tool?
- Am I selecting only needed columns?

## Cost Awareness

Every Snowflake operation has cost implications:

1. **Warehouse time**: Billed per-second with 60s minimum
2. **Data scanned**: Full table scans are expensive
3. **Data transferred**: Large result sets cost money
4. **Storage**: Creating tables consumes storage credits

**Be a good Snowflake citizen:**
- Use LIMIT liberally
- Filter aggressively with WHERE
- Select specific columns, not *
- Use specialized tools that optimize queries
- Consider warehouse size for query complexity

## Summary Checklist

Before executing any Snowflake MCP tool:

- [ ] If querying data, do I have a LIMIT clause? (default: 100)
- [ ] If large table, do I have WHERE filters?
- [ ] Am I using the most specialized tool available?
- [ ] Am I selecting specific columns (not SELECT *)?
- [ ] If using list_objects, do I have like/starts_with filters?
- [ ] If creating objects, am I handling existence correctly?
- [ ] Have I considered the cost of this operation?

**Remember: Snowflake charges for compute time. Every unbounded query costs real money.**
