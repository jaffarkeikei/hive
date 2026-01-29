# CSV Tool

Read, write, append, query, and inspect CSV files with SQL support powered by DuckDB.

## Description

Provides comprehensive CSV file operations for AI agents. Includes pagination, SQL querying, and metadata inspection.

## Functions

### csv_read

Read CSV file contents with pagination support.

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `path` | str | Yes | - | Path to CSV file (relative to session root) |
| `workspace_id` | str | Yes | - | Workspace identifier |
| `agent_id` | str | Yes | - | Agent identifier |
| `session_id` | str | Yes | - | Session identifier |
| `limit` | int \| None | No | `None` | Max rows to return (None = all) |
| `offset` | int | No | `0` | Number of rows to skip |

**Returns:** Dict with columns, rows, counts, and pagination info.

**Example:**
```python
result = csv_read(
    path="data/customers.csv",
    limit=10,
    offset=20,
    workspace_id="ws1",
    agent_id="ag1",
    session_id="s1"
)
# Returns: {success, path, columns, rows, row_count, total_rows, offset, limit}
```

---

### csv_write

Create a new CSV file with specified columns and data.

| Argument | Type | Required | Description |
|----------|------|----------|-------------|
| `path` | str | Yes | Path for new CSV file |
| `workspace_id` | str | Yes | Workspace identifier |
| `agent_id` | str | Yes | Agent identifier |
| `session_id` | str | Yes | Session identifier |
| `columns` | list[str] | Yes | Column names for header |
| `rows` | list[dict] | Yes | List of row dictionaries |

**Returns:** Dict with success status and metadata.

**Example:**
```python
result = csv_write(
    path="output/report.csv",
    columns=["name", "email", "age"],
    rows=[
        {"name": "Alice", "email": "alice@example.com", "age": 30},
        {"name": "Bob", "email": "bob@example.com", "age": 25}
    ],
    workspace_id="ws1",
    agent_id="ag1",
    session_id="s1"
)
```

---

### csv_append

Append rows to an existing CSV file.

| Argument | Type | Required | Description |
|----------|------|----------|-------------|
| `path` | str | Yes | Path to existing CSV file |
| `workspace_id` | str | Yes | Workspace identifier |
| `agent_id` | str | Yes | Agent identifier |
| `session_id` | str | Yes | Session identifier |
| `rows` | list[dict] | Yes | Rows to append |

**Returns:** Dict with rows appended count and total rows.

**Example:**
```python
result = csv_append(
    path="data/customers.csv",
    rows=[
        {"name": "Charlie", "email": "charlie@example.com", "age": 35}
    ],
    workspace_id="ws1",
    agent_id="ag1",
    session_id="s1"
)
```

---

### csv_info

Get CSV file metadata without reading all data.

| Argument | Type | Required | Description |
|----------|------|----------|-------------|
| `path` | str | Yes | Path to CSV file |
| `workspace_id` | str | Yes | Workspace identifier |
| `agent_id` | str | Yes | Agent identifier |
| `session_id` | str | Yes | Session identifier |

**Returns:** Dict with columns, counts, and file size.

**Example:**
```python
result = csv_info(
    path="data/large_dataset.csv",
    workspace_id="ws1",
    agent_id="ag1",
    session_id="s1"
)
# Returns: {success, path, columns, column_count, total_rows, file_size_bytes}
```

---

### csv_sql

Query CSV files using SQL powered by DuckDB. The CSV is loaded as a table named `data`.

| Argument | Type | Required | Description |
|----------|------|----------|-------------|
| `path` | str | Yes | Path to CSV file |
| `workspace_id` | str | Yes | Workspace identifier |
| `agent_id` | str | Yes | Agent identifier |
| `session_id` | str | Yes | Session identifier |
| `query` | str | Yes | SQL query (SELECT only) |

**Security:** Only SELECT statements allowed. INSERT, UPDATE, DELETE, DROP, etc. are blocked.

**Example:**
```python
# Filter rows
result = csv_sql(
    path="data/products.csv",
    query="SELECT * FROM data WHERE price > 100 AND status = 'active'",
    workspace_id="ws1",
    agent_id="ag1",
    session_id="s1"
)

# Aggregate data
result = csv_sql(
    path="data/sales.csv",
    query="SELECT category, COUNT(*) as count, SUM(amount) as total FROM data GROUP BY category",
    workspace_id="ws1",
    agent_id="ag1",
    session_id="s1"
)

# Sort and limit
result = csv_sql(
    path="data/users.csv",
    query="SELECT name, email, created_at FROM data ORDER BY created_at DESC LIMIT 10",
    workspace_id="ws1",
    agent_id="ag1",
    session_id="s1"
)

# Search text (case-insensitive)
result = csv_sql(
    path="data/articles.csv",
    query="SELECT title, author FROM data WHERE LOWER(title) LIKE '%python%'",
    workspace_id="ws1",
    agent_id="ag1",
    session_id="s1"
)
```

## SQL Query Examples

Common DuckDB SQL patterns for `csv_sql`:

```sql
-- WHERE clause
SELECT * FROM data WHERE age > 18 AND country = 'US'

-- Aggregations
SELECT department, AVG(salary) as avg_salary FROM data GROUP BY department

-- JOINs (when used with DuckDB views)
SELECT a.name, b.order_count FROM data a JOIN orders b ON a.id = b.user_id

-- String operations
SELECT * FROM data WHERE LOWER(email) LIKE '%@gmail.com'

-- Date filtering (if dates are parseable)
SELECT * FROM data WHERE TRY_CAST(created_at AS DATE) > '2024-01-01'

-- Sorting
SELECT * FROM data ORDER BY score DESC, name ASC LIMIT 20
```

## Limitations

- **Encoding:** UTF-8 only
- **File size:** Large files (>1GB) may be slow or cause memory issues
- **SQL:** Only SELECT queries allowed for security
- **DuckDB:** Requires `duckdb` package for `csv_sql`

## Error Handling

All functions return error dicts on failure:
```python
{"error": "Description of what went wrong"}
```

Common errors:
- `File not found: <path>`
- `File must have .csv extension`
- `CSV file is empty or has no headers`
- `Query failed: <SQL error message>`
- `Only SELECT queries are allowed for security reasons`
