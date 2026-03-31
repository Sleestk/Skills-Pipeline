# Database Fundamentals

## Connecting to Your Database

Choose your connection method based on your environment:

| Environment | Method | Port | Notes |
|---|---|---|---|
| Frontend apps | Data APIs (REST/GraphQL via supabase-js) | N/A | Requires RLS enabled |
| Persistent servers (VMs, containers) | Direct connection | 5432 | IPv6 only by default |
| Persistent servers needing IPv4 | Pooler session mode | 5432 | Via Supavisor proxy |
| Serverless/Edge functions | Pooler transaction mode | 6543 | No prepared statements |
| Paid tier, high-performance | Dedicated pooler (PgBouncer) | 6543 | Co-located with DB |

### Connection string formats

```
# Direct connection (IPv6)
postgresql://postgres:[YOUR-PASSWORD]@db.<ref>.supabase.co:5432/postgres

# Pooler session mode (IPv4/IPv6)
postgres://postgres.<ref>:[YOUR-PASSWORD]@aws-0-[REGION].pooler.supabase.com:5432/postgres

# Pooler transaction mode (serverless)
postgres://postgres:[YOUR-PASSWORD]@db.<ref>.supabase.co:6543/postgres
```

Get connection strings from the **Connect** button in your project dashboard.

### Transaction mode limitations
- Does NOT support prepared statements
- Turn off prepared statements in your connection library

### SSL
Always connect with SSL in production. Get your Server root certificate from your dashboard under Database Settings.

## Importing Data

### Small datasets
Use CSV import in the Supabase dashboard (Table Editor → Insert → Import Data from CSV). Limit: 100MB.

### Large datasets — pgloader
```bash
apt-get install pgloader
```

Config file example:
```sql
LOAD DATABASE
    FROM sourcedb://USER:PASSWORD@HOST/SOURCE_DB
    INTO postgres://postgres.xxxx:password@xxxx.pooler.supabase.com:6543/postgres
ALTER SCHEMA 'public' OWNER TO 'postgres';
set wal_buffers = '64MB', max_wal_senders = 0, statement_timeout = 0, work_mem to '2GB';
```

### Large datasets — COPY command
```bash
psql -h DATABASE_URL -p 5432 -d postgres -U postgres \
  -c "\COPY movies FROM './movies.csv' WITH DELIMITER ',' CSV HEADER"
```

### Via Supabase API
Use `supabase.from('table').insert([...])`. Avoid bulk imports via API — use COPY for large sets.

### Preparing for large imports
1. **Back up** your data first
2. **Increase statement timeouts** for the session
3. **Estimate disk size** needed, increase in project settings if needed
4. **Disable triggers** temporarily:
   ```sql
   ALTER TABLE table_name DISABLE TRIGGER ALL;
   -- import data...
   ALTER TABLE table_name ENABLE TRIGGER ALL;
   ```
5. **Rebuild indexes** after import:
   ```sql
   CREATE INDEX index_name ON table_name (column_name);
   ```

## Securing Your Data

- Enable **Row Level Security** (RLS) on all public-facing tables
- Use your **anon key** (safe to expose with RLS enabled)
- **Never expose** your **service_role key** on the client — it bypasses RLS
- The JWT is automatically sent by supabase-js when the user is logged in via Supabase Auth