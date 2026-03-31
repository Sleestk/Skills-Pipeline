# Access & Security

## Row Level Security (RLS)

RLS adds implicit WHERE clauses to every query on a table. **Must be enabled on all tables in exposed schemas.**

```sql
alter table "table_name" enable row level security;
```

Once enabled, no data is accessible via the API until you create policies.

### Auto-enable RLS on new tables
```sql
CREATE OR REPLACE FUNCTION rls_auto_enable()
RETURNS EVENT_TRIGGER LANGUAGE plpgsql SECURITY DEFINER SET search_path = pg_catalog AS $$
DECLARE cmd record;
BEGIN
  FOR cmd IN SELECT * FROM pg_event_trigger_ddl_commands()
    WHERE command_tag IN ('CREATE TABLE', 'CREATE TABLE AS', 'SELECT INTO')
      AND object_type IN ('table','partitioned table')
  LOOP
    IF cmd.schema_name IN ('public') THEN
      EXECUTE format('alter table if exists %s enable row level security', cmd.object_identity);
    END IF;
  END LOOP;
END;
$$;

CREATE EVENT TRIGGER ensure_rls ON ddl_command_end
WHEN TAG IN ('CREATE TABLE', 'CREATE TABLE AS', 'SELECT INTO')
EXECUTE FUNCTION rls_auto_enable();
```

### Policy examples

#### SELECT — public read
```sql
create policy "Public profiles are visible to everyone."
on profiles for select
to anon
using ( true );
```

#### SELECT — own data only
```sql
create policy "User can see their own profile only."
on profiles for select
to authenticated
using ( (select auth.uid()) = user_id );
```

#### INSERT — own data only
```sql
create policy "Users can create a profile."
on profiles for insert
to authenticated
with check ( (select auth.uid()) = user_id );
```

#### UPDATE — own data only
```sql
create policy "Users can update their own profile."
on profiles for update
to authenticated
using ( (select auth.uid()) = user_id )
with check ( (select auth.uid()) = user_id );
```

#### DELETE — own data only
```sql
create policy "Users can delete a profile."
on profiles for delete
to authenticated
using ( (select auth.uid()) = user_id );
```

### Helper functions
- `auth.uid()` — returns the ID of the requesting user
- `auth.jwt()` — returns the full JWT (access `app_metadata`, `user_metadata`, `aal`, `amr`)

**Important**: `raw_user_meta_data` is editable by the user. Use `raw_app_meta_data` for authorization data.

### Team-based access via JWT
```sql
create policy "User is in team"
on my_table to authenticated
using ( team_id in (select auth.jwt() -> 'app_metadata' -> 'teams'));
```

### MFA enforcement
```sql
create policy "Restrict updates."
on profiles as restrictive for update
to authenticated
using ( (select auth.jwt()->>'aal') = 'aal2' );
```

### Bypassing RLS
- Use **service_role** key (never on client)
- Or grant bypassrls to a role: `alter role "role_name" with bypassrls;`

### RLS Performance Best Practices

1. **Add indexes** on columns used in policies:
   ```sql
   create index userid on test_table using btree (user_id);
   ```

2. **Wrap functions in `select`** for caching:
   ```sql
   -- BAD: auth.uid() called per row
   using ( auth.uid() = user_id );
   -- GOOD: cached per statement
   using ( (select auth.uid()) = user_id );
   ```

3. **Always add client-side filters** that mirror RLS:
   ```js
   // Even though RLS handles it, this helps the query planner
   const { data } = supabase.from('table').select().eq('user_id', userId)
   ```

4. **Use security definer functions** for complex lookups:
   ```sql
   create function private.has_good_role()
   returns boolean language plpgsql security definer as $$
   begin
     return exists (
       select 1 from roles_table
       where (select auth.uid()) = user_id and role = 'good_role'
     );
   end;
   $$;

   create policy "rls_test_select" on test_table to authenticated
   using ( (select private.has_good_role()) );
   ```

5. **Minimize joins** in policies — use `IN` instead:
   ```sql
   -- GOOD: no join to source table
   using ( team_id in (
     select team_id from team_user
     where user_id = (select auth.uid())
   ));
   ```

6. **Specify roles** with `TO`:
   ```sql
   -- Prevents policy from running for anon users
   create policy "..." on test to authenticated using (...);
   ```

## Column Level Security

Restrict which columns a role can access, beyond RLS.

```sql
-- Revoke table-level UPDATE
revoke update on table public.posts from authenticated;

-- Grant column-level UPDATE only for title and content
grant update (title, content) on table public.posts to authenticated;
```

**Note**: With column restrictions, `SELECT *` will fail for restricted roles. Always specify column names explicitly.

## Postgres Roles

### Supabase default roles
| Role | Purpose |
|---|---|
| `postgres` | Admin, full privileges |
| `anon` | Unauthenticated API access |
| `authenticated` | Authenticated API access |
| `service_role` | Bypasses RLS (server-only) |
| `authenticator` | API gateway role (validates JWTs) |

### Creating roles
```sql
create role "role_name";

-- With login
create role "role_name" with login password 'extremely_secure_password';
```

### Granting / Revoking
```sql
GRANT permission_type ON object_name TO role_name;
REVOKE permission_type ON object_name FROM role_name;
```

### Role hierarchy
```sql
create role "child_role" inherit "parent_role";
-- Prevent inheritance
alter role "child_role" noinherit;
```

## Custom Roles

Example: create a `manager` role with full read access to a storage bucket.

```sql
create role 'manager';
grant manager to authenticator;
grant anon to manager;

create policy "Manager can view all files in bucket 'teams'"
on storage.objects for select to manager
using ( bucket_id = 'teams' );
```

To impersonate a custom role, sign a JWT with `{ role: 'manager', sub: USER_ID }` using your JWT_SECRET.