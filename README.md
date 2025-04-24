## Script: Monitor_Long_Running_Queries.sql

**Description**:
This script captures real-time information about **currently running user queries**, including performance metrics, progress, and **wait types**. It's a powerful tool for DBAs to diagnose slow performance, blocking, or resource bottlenecks.

---

### 🔧 Query:
```sql
SELECT 
    r.session_id,
    DB_NAME(r.database_id) AS DatabaseName,
    r.command,
    r.status,
    r.wait_type,                                           
    r.percent_complete,
    DATEADD(MILLISECOND, r.estimated_completion_time, GETDATE()) AS Estimated_Completion,
    r.cpu_time,
    r.total_elapsed_time / 1000.0 AS ElapsedSeconds,
    r.reads,
    r.writes,
    r.logical_reads,
    r.blocking_session_id,
    t.text AS QueryText
FROM 
    sys.dm_exec_requests r
CROSS APPLY 
    sys.dm_exec_sql_text(r.sql_handle) t
WHERE 
    DB_NAME(r.database_id) NOT IN ('master', 'msdb', 'tempdb', 'model')
ORDER BY 
    r.total_elapsed_time DESC;
```

---

### 📋 Output Columns:

| Column             | Description                                          |
|--------------------|------------------------------------------------------|
| `session_id`       | Session running the query                            |
| `DatabaseName`     | User database name                                   |
| `command`          | Type of SQL command (e.g., SELECT, BACKUP)           |
| `status`           | Request status (e.g., running, suspended)            |
| `wait_type`        | Type of wait (e.g., LCK_M_SCH_S, IO_COMPLETION)      |
| `percent_complete` | For operations like backups or restores              |
| `Estimated_Completion` | Predicted end time if applicable                 |
| `cpu_time`         | Total CPU time used (in ms)                          |
| `ElapsedSeconds`   | Time since the query started (in seconds)            |
| `reads`, `writes`  | Physical I/O stats                                   |
| `logical_reads`    | Number of pages read from buffer pool                |
| `blocking_session_id` | Session that is blocking this one (if any)       |
| `QueryText`        | Full SQL text of the currently running command       |

---

### ✅ Use Case:
- Live query and workload monitoring
- Detect long-running and blocked sessions
- Identify queries waiting on resources (locks, memory, I/O)
- Improve troubleshooting response time

---

### 🔍 Common Wait Types to Watch:
| Wait Type         | Meaning                          |
|-------------------|----------------------------------|
| `LCK_M_...`       | Lock waits (e.g., row/table lock)|
| `CXPACKET`        | Parallelism-related wait         |
| `PAGEIOLATCH_*`   | Disk I/O wait                    |
| `ASYNC_NETWORK_IO`| Query result not being consumed  |
| `RESOURCE_SEMAPHORE` | Memory-related wait          |

