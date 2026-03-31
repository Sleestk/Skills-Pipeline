# Database Intermediate

## Cascade Deletes

Five options for foreign key constraint deletes:

| Option | Behavior |
|---|---|
| `CASCADE` | Deleting parent deletes all child rows |
| `RESTRICT` | Prevents parent delete if children exist (immediate) |
| `SET NULL` | Sets FK columns in children to NULL |
| `SET DEFAULT` | Sets FK columns in children to their defaults |
| `NO ACTION` | Like RESTRICT but can be deferred to end of transaction |

```sql
alter table child_table
add constraint fk_parent foreign key (parent_id) references parent_table (id)
  on delete cascade;
```

### RESTRICT vs NO ACTION
- Both prevent deletion if references exist
- `NO ACTION INITIALLY DEFERRED` defers the check until end of transaction, allowing other cascades to run first
- `NO ACTION` is the default if you don't specify anything

## Enums

```sql
create type mood as enum ('happy', 'sad', 'excited', 'calm');

create table person (
  id serial primary key,
  name text,
  current_mood mood
);

insert into person (name, current_mood) values ('Alice', 'happy');

-- Add values
alter type mood add value 'content';

-- List values
select enum_range(null::mood);
```

**When to use enums**: Small, fixed, rarely-changing lists (continents, departments). For frequently changing values, use a lookup table with foreign keys instead.

## Database Functions

### Simple function
```sql
create or replace function hello_world()
returns text
language sql
as $$
  select 'hello world';
$$;
```

```js
const { data, error } = await supabase.rpc('hello_world')
```

### Returning data sets
```sql
create or replace function get_planets()
returns setof planets
language sql
as $$
  select * from planets;
$$;
```

```js
// Can chain filters on set-returning functions
const { data, error } = supabase.rpc('get_planets').eq('id', 1)
```

### With parameters (plpgsql)
```sql
create or replace function add_planet(name text)
returns bigint
language plpgsql
as $$
declare
  new_row bigint;
begin
  insert into planets(name) values (add_planet.name) returning id into new_row;
  return new_row;
end;
$$;
```

```js
const { data, error } = await supabase.rpc('add_planet', { name: 'Jakku' })
```

### Security: definer vs invoker
- Default is `invoker` (runs as calling user) — **preferred**
- If using `security definer`, always set `search_path = ''` and use fully qualified table names (`public.table`)

### Function privileges
```sql
-- Revoke from everyone
revoke execute on function public.hello_world from public;
revoke execute on function public.hello_world from anon;

-- Grant to specific role
grant execute on function public.hello_world to authenticated;
```

### Debugging with RAISE
```sql
raise log 'Value: %', my_var;
raise warning 'Warning: %', my_var;
raise exception 'Error: %', my_var;  -- stops execution
```

## Triggers

### Creating a trigger (2 parts: function + trigger)
```sql
-- 1. Trigger function
create function update_salary_log()
returns trigger
language plpgsql
as $$
begin
  insert into salary_log(employee_id, old_salary, new_salary)
  values (new.id, old.salary, new.salary);
  return new;
end;
$$;

-- 2. Trigger object
create trigger salary_update_trigger
after update on employees
for each row
execute function update_salary_log();
```

### Trigger variables
- `NEW` — new row data (INSERT, UPDATE)
- `OLD` — old row data (UPDATE, DELETE)
- `TG_OP` — operation (`INSERT`, `UPDATE`, `DELETE`, `TRUNCATE`)
- `TG_TABLE_NAME` — table name
- `TG_WHEN` — `BEFORE` or `AFTER`

### Types
- `BEFORE` — executes before the change
- `AFTER` — executes after the change

### Frequency
- `FOR EACH ROW` — once per affected row
- `FOR EACH STATEMENT` — once for the entire operation

### Dropping
```sql
drop trigger "trigger_name" on "table_name";
```

## Database Webhooks

Send HTTP requests on table events using the `pg_net` extension (async, non-blocking).

```sql
create trigger "my_webhook" after insert
on "public"."my_table" for each row
execute function "supabase_functions"."http_request"(
  'http://host.docker.internal:3000',
  'POST',
  '{"Content-Type":"application/json"}',
  '{}',
  '1000'
);
```

### Payload types
```typescript
type InsertPayload = { type: 'INSERT'; table: string; schema: string; record: TableRecord; old_record: null }
type UpdatePayload = { type: 'UPDATE'; table: string; schema: string; record: TableRecord; old_record: TableRecord }
type DeletePayload = { type: 'DELETE'; table: string; schema: string; record: null; old_record: TableRecord }
```

### Local development
Use `host.docker.internal` instead of `localhost` since Postgres runs in Docker.

## Event Triggers

Fire on database-level DDL events (not row events). Only the `postgres` role can create them.

### Prevent dropping tables
```sql
CREATE OR REPLACE FUNCTION dont_drop_function()
  RETURNS event_trigger LANGUAGE plpgsql AS $$
DECLARE obj record;
BEGIN
    FOR obj IN SELECT * FROM pg_event_trigger_dropped_objects() LOOP
        IF obj.object_type = 'table' THEN
            RAISE EXCEPTION 'Tables in this schema are protected and cannot be dropped';
        END IF;
    END LOOP;
END;
$$;

CREATE EVENT TRIGGER dont_drop_trigger ON sql_drop
EXECUTE FUNCTION dont_drop_function();
```

### Firing events
- `ddl_command_start` — before DDL
- `ddl_command_end` — after DDL
- `sql_drop` — before `ddl_command_end` for DROP commands
- `table_rewrite` — before ALTER TABLE rewrites

### Disable / Drop
```sql
ALTER EVENT TRIGGER dont_drop_trigger DISABLE;
DROP EVENT TRIGGER dont_drop_trigger;
```