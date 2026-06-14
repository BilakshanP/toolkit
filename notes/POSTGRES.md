## PostgreSQL

### Connect

```bash
psql -U postgres                           # local
psql -h host -p 5432 -U user -d dbname     # remote
```

### Databases

```sql
CREATE DATABASE mydb;
DROP DATABASE mydb;
\l                  -- list databases
\c mydb             -- connect to database
```

### Schemas

```sql
CREATE SCHEMA app;
DROP SCHEMA app CASCADE;
\dn                 -- list schemas
SET search_path TO app, public;
```

### Tables

```sql
CREATE TABLE users (
    id          BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email       TEXT NOT NULL UNIQUE,
    name        TEXT NOT NULL,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- With foreign key
CREATE TABLE posts (
    id          BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    user_id     BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    title       TEXT NOT NULL,
    body        TEXT,
    published   BOOLEAN NOT NULL DEFAULT false,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);

DROP TABLE posts;
DROP TABLE IF EXISTS posts CASCADE;

\dt                 -- list tables
\d users            -- describe table
```

### Constraints

```sql
-- PRIMARY KEY

-- Inline
CREATE TABLE users (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY
);

-- Named constraint
CREATE TABLE users (
    id BIGINT GENERATED ALWAYS AS IDENTITY,
    CONSTRAINT pk_users PRIMARY KEY (id)
);

-- Composite primary key
CREATE TABLE post_tags (
    post_id BIGINT NOT NULL,
    tag_id  BIGINT NOT NULL,
    PRIMARY KEY (post_id, tag_id)
);

-- Add after table creation
ALTER TABLE users ADD PRIMARY KEY (id);
ALTER TABLE users ADD CONSTRAINT pk_users PRIMARY KEY (id);


-- UNIQUE

-- Inline
CREATE TABLE users (
    email TEXT NOT NULL UNIQUE
);

-- Named constraint
CREATE TABLE users (
    email TEXT NOT NULL,
    CONSTRAINT uq_users_email UNIQUE (email)
);

-- Composite unique
CREATE TABLE memberships (
    user_id BIGINT NOT NULL,
    org_id  BIGINT NOT NULL,
    CONSTRAINT uq_membership UNIQUE (user_id, org_id)
);

-- Add after table creation
ALTER TABLE users ADD UNIQUE (email);
ALTER TABLE users ADD CONSTRAINT uq_users_email UNIQUE (email);


-- FOREIGN KEY

-- Inline
CREATE TABLE posts (
    user_id BIGINT NOT NULL REFERENCES users(id)
);

-- Inline with actions
CREATE TABLE posts (
    user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE ON UPDATE CASCADE
);

-- Named constraint
CREATE TABLE posts (
    user_id BIGINT NOT NULL,
    CONSTRAINT fk_posts_user FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Composite foreign key
CREATE TABLE order_items (
    order_id   BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    CONSTRAINT fk_order_items FOREIGN KEY (order_id, product_id)
        REFERENCES order_products(order_id, product_id)
);

-- Add after table creation
ALTER TABLE posts ADD CONSTRAINT fk_posts_user
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE;

-- ON DELETE options: CASCADE, SET NULL, SET DEFAULT, RESTRICT, NO ACTION


-- CHECK

-- Inline
CREATE TABLE users (
    age INT CHECK (age >= 0)
);

-- Named
CREATE TABLE users (
    age INT,
    CONSTRAINT chk_age CHECK (age >= 0 AND age <= 150)
);

-- Multi-column check
CREATE TABLE events (
    start_at TIMESTAMPTZ NOT NULL,
    end_at   TIMESTAMPTZ NOT NULL,
    CONSTRAINT chk_dates CHECK (end_at > start_at)
);

-- Add after table creation
ALTER TABLE users ADD CONSTRAINT chk_age CHECK (age >= 0);


-- NOT NULL (only inline or via ALTER)
CREATE TABLE users (
    name TEXT NOT NULL
);
ALTER TABLE users ALTER COLUMN name SET NOT NULL;
ALTER TABLE users ALTER COLUMN name DROP NOT NULL;


-- Drop any constraint
ALTER TABLE users DROP CONSTRAINT constraint_name;
```

### Alter Table

```sql
ALTER TABLE users ADD COLUMN bio TEXT;
ALTER TABLE users DROP COLUMN bio;
ALTER TABLE users RENAME COLUMN name TO display_name;
ALTER TABLE users ALTER COLUMN email SET NOT NULL;
ALTER TABLE users ADD CONSTRAINT email_check CHECK (email LIKE '%@%');
```

### CRUD

```sql
-- Insert
INSERT INTO users (email, name) VALUES ('a@b.com', 'Alice');
INSERT INTO users (email, name) VALUES ('b@c.com', 'Bob') RETURNING *;

-- Select
SELECT * FROM users;
SELECT id, name FROM users WHERE email = 'a@b.com';
SELECT * FROM users ORDER BY created_at DESC LIMIT 10 OFFSET 0;

-- Update
UPDATE users SET name = 'Alice B' WHERE id = 1 RETURNING *;

-- Delete
DELETE FROM users WHERE id = 1;

-- Upsert
INSERT INTO users (email, name) VALUES ('a@b.com', 'Alice')
ON CONFLICT (email) DO UPDATE SET name = EXCLUDED.name;
```

### Filtering

```sql
WHERE status = 'active'
WHERE age BETWEEN 18 AND 65
WHERE name LIKE 'A%'                       -- case-sensitive
WHERE name ILIKE 'a%'                      -- case-insensitive
WHERE id IN (1, 2, 3)
WHERE bio IS NULL
WHERE bio IS NOT NULL
WHERE age > 18 AND status = 'active'
```

### Joins

```sql
-- Inner join (only matching rows)
SELECT u.name, p.title
FROM users u
JOIN posts p ON p.user_id = u.id;

-- Left outer join (all left, NULLs for no match)
SELECT u.name, p.title
FROM users u
LEFT JOIN posts p ON p.user_id = u.id;

-- Right outer join (all right, NULLs for no match)
SELECT u.name, p.title
FROM users u
RIGHT JOIN posts p ON p.user_id = u.id;

-- Full outer join (all from both, NULLs where no match)
SELECT u.name, p.title
FROM users u
FULL JOIN posts p ON p.user_id = u.id;

-- Cross join (cartesian product)
SELECT u.name, r.role
FROM users u
CROSS JOIN roles r;

-- Self join
SELECT e.name AS employee, m.name AS manager
FROM employees e
JOIN employees m ON e.manager_id = m.id;

-- Left excluding (left only, no match on right)
SELECT u.name
FROM users u
LEFT JOIN posts p ON p.user_id = u.id
WHERE p.id IS NULL;

-- Right excluding
SELECT p.title
FROM users u
RIGHT JOIN posts p ON p.user_id = u.id
WHERE u.id IS NULL;

-- Full outer excluding (rows that have no match on either side)
SELECT u.name, p.title
FROM users u
FULL JOIN posts p ON p.user_id = u.id
WHERE u.id IS NULL OR p.id IS NULL;

-- Semi join (exists — rows in left that have a match in right)
SELECT u.name
FROM users u
WHERE EXISTS (SELECT 1 FROM posts p WHERE p.user_id = u.id);

-- Anti join (not exists — rows in left with no match in right)
SELECT u.name
FROM users u
WHERE NOT EXISTS (SELECT 1 FROM posts p WHERE p.user_id = u.id);
```

### Aggregates

```sql
SELECT COUNT(*) FROM users;
SELECT user_id, COUNT(*) AS post_count
FROM posts
GROUP BY user_id
HAVING COUNT(*) > 5;

SELECT
    DATE_TRUNC('month', created_at) AS month,
    COUNT(*)
FROM users
GROUP BY month
ORDER BY month;
```

### Common Functions

```sql
-- String
LENGTH(text), LOWER(text), UPPER(text), TRIM(text)
CONCAT(a, b), a || b
SUBSTRING(text FROM 1 FOR 5)
SPLIT_PART('a.b.c', '.', 2)               -- 'b'

-- Date/Time
now(), CURRENT_DATE, CURRENT_TIMESTAMP
AGE(timestamp)                             -- interval since
DATE_TRUNC('hour', timestamp)
EXTRACT(YEAR FROM timestamp)
timestamp + INTERVAL '1 day'

-- Coalesce
COALESCE(nullable_col, 'default')
NULLIF(a, b)                               -- NULL if a = b
```

### Indexes

```sql
CREATE INDEX idx_users_email ON users (email);
CREATE UNIQUE INDEX idx_users_email ON users (email);
CREATE INDEX idx_posts_user_created ON posts (user_id, created_at DESC);

-- Partial index
CREATE INDEX idx_published_posts ON posts (user_id) WHERE published = true;

-- GIN (for arrays, jsonb, full-text)
CREATE INDEX idx_tags ON posts USING GIN (tags);

DROP INDEX idx_users_email;
\di                 -- list indexes
```

### JSON/JSONB

```sql
CREATE TABLE events (
    id    BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    data  JSONB NOT NULL
);

-- Access
SELECT data->>'name' FROM events;          -- text
SELECT data->'address'->>'city' FROM events;
SELECT data @> '{"type": "click"}' FROM events;  -- contains

-- Index
CREATE INDEX idx_events_type ON events USING GIN (data);

-- Update nested value
UPDATE events SET data = jsonb_set(data, '{name}', '"new"');
```

### CTEs (Common Table Expressions)

```sql
-- Basic
WITH active_users AS (
    SELECT * FROM users WHERE status = 'active'
)
SELECT u.name, COUNT(p.id) AS posts
FROM active_users u
LEFT JOIN posts p ON p.user_id = u.id
GROUP BY u.name;

-- Multiple CTEs
WITH
    recent_posts AS (
        SELECT * FROM posts WHERE created_at > now() - INTERVAL '7 days'
    ),
    post_counts AS (
        SELECT user_id, COUNT(*) AS cnt FROM recent_posts GROUP BY user_id
    )
SELECT u.name, pc.cnt
FROM users u
JOIN post_counts pc ON pc.user_id = u.id;

-- Recursive (tree traversal, e.g. org chart)
WITH RECURSIVE tree AS (
    -- base case
    SELECT id, name, manager_id, 0 AS depth
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- recursive step
    SELECT e.id, e.name, e.manager_id, t.depth + 1
    FROM employees e
    JOIN tree t ON e.manager_id = t.id
)
SELECT * FROM tree ORDER BY depth, name;

-- Recursive (generate series alternative)
WITH RECURSIVE dates AS (
    SELECT DATE '2024-01-01' AS d
    UNION ALL
    SELECT d + 1 FROM dates WHERE d < '2024-01-31'
)
SELECT * FROM dates;
```

### Window Functions

```sql
-- ROW_NUMBER (unique rank, no ties)
SELECT name, score,
    ROW_NUMBER() OVER (ORDER BY score DESC) AS rn
FROM players;

-- RANK (ties get same rank, gaps after)
-- 1, 2, 2, 4
SELECT name, score,
    RANK() OVER (ORDER BY score DESC) AS rnk
FROM players;

-- DENSE_RANK (ties get same rank, no gaps)
-- 1, 2, 2, 3
SELECT name, score,
    DENSE_RANK() OVER (ORDER BY score DESC) AS drnk
FROM players;

-- PARTITION BY (rank within groups)
SELECT user_id, title, created_at,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC) AS rn
FROM posts;

-- Get latest post per user
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC) AS rn
    FROM posts
) sub WHERE rn = 1;

-- LAG / LEAD (access previous / next row)
SELECT date, revenue,
    LAG(revenue) OVER (ORDER BY date) AS prev_revenue,
    LEAD(revenue) OVER (ORDER BY date) AS next_revenue,
    revenue - LAG(revenue) OVER (ORDER BY date) AS daily_change
FROM daily_sales;

-- Running total
SELECT id, amount,
    SUM(amount) OVER (ORDER BY created_at) AS running_total
FROM transactions;

-- Moving average (last 7 rows)
SELECT date, revenue,
    AVG(revenue) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS ma_7
FROM daily_sales;

-- NTILE (divide into N buckets)
SELECT name, score,
    NTILE(4) OVER (ORDER BY score DESC) AS quartile
FROM players;

-- FIRST_VALUE / LAST_VALUE
SELECT name, score,
    FIRST_VALUE(name) OVER (ORDER BY score DESC) AS top_player
FROM players;

-- COUNT/SUM/AVG as window (without collapsing rows)
SELECT name, department, salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg,
    salary - AVG(salary) OVER (PARTITION BY department) AS diff_from_avg
FROM employees;
```

### Transactions

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

-- Rollback on error
BEGIN;
-- ... operations ...
ROLLBACK;

-- Savepoints
BEGIN;
SAVEPOINT sp1;
-- ... risky operation ...
ROLLBACK TO sp1;
COMMIT;
```

### Views

```sql
CREATE VIEW active_users AS
SELECT * FROM users WHERE status = 'active';

CREATE MATERIALIZED VIEW user_stats AS
SELECT user_id, COUNT(*) AS post_count
FROM posts
GROUP BY user_id;

REFRESH MATERIALIZED VIEW user_stats;
```

### Useful psql Commands

```
\l              -- list databases
\c dbname       -- connect
\dt             -- list tables
\d table        -- describe table
\di             -- list indexes
\dn             -- list schemas
\df             -- list functions
\du             -- list roles
\x              -- toggle expanded output
\timing         -- toggle query timing
\i file.sql     -- execute file
\copy           -- client-side COPY
```

### Performance

```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'a@b.com';
VACUUM ANALYZE users;                      -- update planner stats
```

### Full-Text Search

```sql
-- Basic search
SELECT * FROM posts
WHERE to_tsvector('english', title || ' ' || body) @@ to_tsquery('english', 'rust & async');

-- Stored tsvector column (faster)
ALTER TABLE posts ADD COLUMN search_vec TSVECTOR
GENERATED ALWAYS AS (to_tsvector('english', title || ' ' || COALESCE(body, ''))) STORED;

CREATE INDEX idx_posts_search ON posts USING GIN (search_vec);

SELECT * FROM posts WHERE search_vec @@ to_tsquery('english', 'rust & async');

-- Ranking results
SELECT title, ts_rank(search_vec, q) AS rank
FROM posts, to_tsquery('english', 'rust & async') q
WHERE search_vec @@ q
ORDER BY rank DESC;

-- Query syntax
to_tsquery('word')                         -- single word
to_tsquery('rust & async')                 -- AND
to_tsquery('rust | go')                    -- OR
to_tsquery('!javascript')                  -- NOT
to_tsquery('web <-> server')               -- adjacent (phrase)
plainto_tsquery('rust async web')          -- auto AND between words
websearch_to_tsquery('rust "async web" -js')  -- google-like syntax
```

### Roles & Permissions

```sql
-- Create role
CREATE ROLE app_user LOGIN PASSWORD 'secret';
CREATE ROLE readonly NOLOGIN;              -- group role

-- Grant privileges
GRANT CONNECT ON DATABASE mydb TO app_user;
GRANT USAGE ON SCHEMA public TO app_user;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly;
GRANT INSERT, UPDATE, DELETE ON users, posts TO app_user;

-- Grant role membership
GRANT readonly TO app_user;

-- Default privileges (for future tables)
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT ON TABLES TO readonly;

-- Revoke
REVOKE DELETE ON users FROM app_user;

-- Row-level security
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;

CREATE POLICY user_posts ON posts
    FOR ALL
    USING (user_id = current_setting('app.user_id')::BIGINT);

-- Set context before queries
SET app.user_id = '42';
SELECT * FROM posts;                       -- only sees user 42's posts

\du                 -- list roles
```

### Extensions

```sql
-- List available / installed
SELECT * FROM pg_available_extensions;
\dx                 -- list installed extensions

-- UUID
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
SELECT uuid_generate_v4();
-- Or use built-in (PG 13+):
SELECT gen_random_uuid();

-- pgcrypto
CREATE EXTENSION IF NOT EXISTS pgcrypto;
SELECT crypt('password', gen_salt('bf'));   -- bcrypt hash
SELECT crypt('password', hash) = hash;     -- verify

-- pgvector (similarity search)
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE embeddings (
    id      BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    content TEXT NOT NULL,
    vec     VECTOR(1536)                   -- OpenAI dimension
);

CREATE INDEX idx_embeddings_vec ON embeddings USING ivfflat (vec vector_cosine_ops)
WITH (lists = 100);

-- Nearest neighbors
SELECT content, vec <=> '[0.1, 0.2, ...]'::VECTOR AS distance
FROM embeddings
ORDER BY distance
LIMIT 5;

-- pg_trgm (fuzzy matching)
CREATE EXTENSION IF NOT EXISTS pg_trgm;

CREATE INDEX idx_users_name_trgm ON users USING GIN (name gin_trgm_ops);

SELECT * FROM users WHERE name % 'alice';            -- similarity
SELECT * FROM users WHERE name ILIKE '%ali%';        -- uses trgm index

-- tablefunc (crosstab/pivot)
CREATE EXTENSION IF NOT EXISTS tablefunc;
```
