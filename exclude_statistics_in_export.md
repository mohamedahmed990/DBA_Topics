When you are running Oracle Data Pump (`expdp` or `impdp`), adding the parameter `EXCLUDE=STATISTICS` tells the system **not to export or import the optimizer statistics** (the data about table sizes, row counts, and index distributions used by the database to plan queries).

Here is why database administrators and developers choose to do this:

### 1. It Drastically Speeds Up the Export/Import

Gathering and writing statistics during an export—or rebuild/reapply during an import—takes a massive amount of time, especially on large databases with thousands of objects. Excluding statistics can cut your total migration or backup time by **30% to 50%**.

### 2. Imported Statistics Are Often Already Outdated

If you are moving data from a production system to a development environment, or if the data is going to change significantly immediately after the import, the old statistics are completely useless. It is much more efficient to skip them and generate fresh ones later.

### 3. Avoids "Stale" or Suboptimal Execution Plans

If you import old production statistics into a smaller or differently configured test environment, the Oracle Optimizer might get confused. It might choose highly inefficient query execution plans because it thinks it is working with production-level hardware and data distributions.

---

### What should you do after using it?

Because the Oracle Optimizer relies heavily on statistics to keep your queries running fast, you shouldn't leave the database without them. The standard best practice is to exclude them during the import, and then manually gather fresh, accurate statistics once the data is safely loaded.

You can do this easily using the `DBMS_STATS` package right after your import finishes:

```sql
-- To gather fresh statistics for an entire schema
EXEC DBMS_STATS.GATHER_SCHEMA_STATS('YOUR_SCHEMA_NAME');

```
