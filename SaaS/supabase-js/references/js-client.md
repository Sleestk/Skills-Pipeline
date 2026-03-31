# JavaScript Client Reference (`@supabase/supabase-js`)

## Installing

```bash
npm install @supabase/supabase-js
```

## Initializing

```js
import { createClient } from '@supabase/supabase-js'

const supabase = createClient('https://<project>.supabase.co', '<anon-key>')
```

With options:
```js
const supabase = createClient(supabaseUrl, supabaseKey, {
  auth: {
    flowType: 'pkce',           // default for @supabase/ssr
    autoRefreshToken: true,
    persistSession: true,
    detectSessionInUrl: true,
  },
})
```

## TypeScript Support

Generate types from your database:
```bash
npx supabase gen types typescript --project-id <ref> > database.types.ts
```

Use with the client:
```ts
import { createClient } from '@supabase/supabase-js'
import { Database } from './database.types'

const supabase = createClient<Database>(supabaseUrl, supabaseKey)
```

---

## Database Operations

### Fetch data — `select()`
```js
// All rows
const { data, error } = await supabase.from('characters').select()

// Specific columns
const { data, error } = await supabase.from('characters').select('name')

// With related tables (joins)
const { data, error } = await supabase.from('sections').select(`
  name,
  instruments ( name )
`)

// Inner join
const { data, error } = await supabase
  .from('instruments')
  .select('name, sections!inner(name)')
  .eq('sections.name', 'woodwinds')

// Count only
const { count, error } = await supabase
  .from('characters')
  .select('*', { count: 'exact', head: true })

// JSON fields
const { data, error } = await supabase.from('users').select(`
  id, name, address->city
`)

// Rename columns
const { data, error } = await supabase.from('messages').select(`
  content,
  from:sender_id(name),
  to:receiver_id(name)
`)
```

Default max: 1000 rows. Change in API settings. Use `range()` for pagination.

### Insert data — `insert()`
```js
// Single row
const { data, error } = await supabase
  .from('characters')
  .insert({ name: 'Frodo' })
  .select()

// Multiple rows
const { data, error } = await supabase
  .from('characters')
  .insert([
    { name: 'Frodo' },
    { name: 'Sam' },
  ])
  .select()
```

### Update data — `update()`
```js
const { data, error } = await supabase
  .from('characters')
  .update({ name: 'Gandalf the White' })
  .eq('id', 1)
  .select()
```

### Upsert data — `upsert()`
```js
const { data, error } = await supabase
  .from('characters')
  .upsert({ id: 1, name: 'Gandalf the White' })
  .select()
```

### Delete data — `delete()`
```js
const { data, error } = await supabase
  .from('characters')
  .delete()
  .eq('id', 1)
```

### Call a Postgres function — `rpc()`
```js
// No params
const { data, error } = await supabase.rpc('hello_world')

// With params
const { data, error } = await supabase.rpc('add_planet', { name: 'Jakku' })

// Chain filters on set-returning functions
const { data, error } = await supabase.rpc('get_planets').eq('id', 1)
```

---

## Filters

Chain after `.select()`, `.update()`, `.delete()`, or `.rpc()`.

| Method | SQL equivalent | Example |
|---|---|---|
| `.eq(col, val)` | `= val` | `.eq('id', 1)` |
| `.neq(col, val)` | `!= val` | `.neq('status', 'deleted')` |
| `.gt(col, val)` | `> val` | `.gt('age', 18)` |
| `.gte(col, val)` | `>= val` | `.gte('score', 90)` |
| `.lt(col, val)` | `< val` | `.lt('price', 100)` |
| `.lte(col, val)` | `<= val` | `.lte('price', 50)` |
| `.like(col, pattern)` | `LIKE pattern` | `.like('name', '%odo%')` |
| `.ilike(col, pattern)` | `ILIKE pattern` | `.ilike('name', '%ODO%')` |
| `.is(col, val)` | `IS val` | `.is('deleted', null)` |
| `.in(col, arr)` | `IN (arr)` | `.in('id', [1, 2, 3])` |
| `.contains(col, val)` | `@> val` | `.contains('tags', ['js'])` |
| `.containedBy(col, val)` | `<@ val` | `.containedBy('tags', ['js','ts'])` |
| `.not(col, op, val)` | `NOT (col op val)` | `.not('status', 'eq', 'deleted')` |
| `.or(filters)` | `OR` | `.or('id.eq.1,name.eq.Frodo')` |
| `.filter(col, op, val)` | custom | `.filter('id', 'in', '(1,2,3)')` |
| `.match(obj)` | multiple `=` | `.match({ name: 'Frodo', age: 50 })` |
| `.textSearch(col, query)` | `@@` | `.textSearch('content', 'cat & dog')` |

### Filtering on joined tables
```js
const { data, error } = await supabase
  .from('instruments')
  .select('name, sections!inner(name)')
  .eq('sections.name', 'woodwinds')
```

---

## Modifiers

Chain after filters.

| Method | Purpose | Example |
|---|---|---|
| `.select()` after insert/update | Return inserted/updated rows | `.insert({...}).select()` |
| `.order(col, { ascending })` | Sort results | `.order('created_at', { ascending: false })` |
| `.limit(n)` | Max rows | `.limit(10)` |
| `.range(from, to)` | Pagination (0-indexed, inclusive) | `.range(0, 9)` |
| `.single()` | Return single object (errors if != 1 row) | `.select().eq('id',1).single()` |
| `.maybeSingle()` | Return single object or null | `.select().eq('id',1).maybeSingle()` |
| `.abortSignal(signal)` | Cancel request | `.select().abortSignal(controller.signal)` |

---

## Auth Methods

### Sign up
```js
const { data, error } = await supabase.auth.signUp({
  email: 'user@example.com',
  password: 'password',
  options: { data: { first_name: 'John' } },
})
```

### Sign in with password
```js
const { data, error } = await supabase.auth.signInWithPassword({
  email: 'user@example.com',
  password: 'password',
})
```

### Sign in with OTP
```js
const { data, error } = await supabase.auth.signInWithOtp({
  email: 'user@example.com',
})
```

### Sign in with OAuth
```js
const { data, error } = await supabase.auth.signInWithOAuth({
  provider: 'google',
  options: { redirectTo: 'https://example.com/auth/callback' },
})
```

### Sign out
```js
const { error } = await supabase.auth.signOut()
```

### Get user (validates JWT with server)
```js
const { data: { user } } = await supabase.auth.getUser()
```

### Get session (local, no server validation)
```js
const { data: { session } } = await supabase.auth.getSession()
```

### Update user
```js
const { data, error } = await supabase.auth.updateUser({
  data: { first_name: 'Jane' },
})
```

### Reset password
```js
await supabase.auth.resetPasswordForEmail('user@example.com', {
  redirectTo: 'https://example.com/update-password',
})
```

### Verify OTP
```js
const { data, error } = await supabase.auth.verifyOtp({
  email: 'user@example.com',
  token: '123456',
  type: 'email',
})
```

### Exchange code for session (PKCE)
```js
const { data, error } = await supabase.auth.exchangeCodeForSession(code)
```

### Listen to auth events
```js
const { data: { subscription } } = supabase.auth.onAuthStateChange((event, session) => {
  console.log(event, session)
})
// Unsubscribe later
subscription.unsubscribe()
```