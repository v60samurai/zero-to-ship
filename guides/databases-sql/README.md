# Databases and SQL: Zero to Ship

> A filing cabinet that never forgets, never loses anything, and can find any document in milliseconds, as long as you organized it properly.

## Why Should You Care?

Every app that remembers anything uses a database. Your users, their passwords, their orders, their messages, their settings. When the server restarts, when you deploy a new version, when the power goes out and comes back, the data is still there. That's what a database does. It persists data.

If you're building with AI tools, you're generating database code constantly. Cursor writes Prisma schemas, Claude Code generates SQL queries, v0 scaffolds apps with Supabase tables. The code usually works for simple cases. But the moment you have real users, real data, and real complexity, you hit problems that AI tools create confidently and silently: missing indexes that make pages load in 10 seconds instead of 100 milliseconds, N+1 queries that fire 200 database calls for what should be one, money stored as floats that silently loses cents, and queries that return wrong results because the JOIN was backwards. You can't spot these problems if you don't understand what the database is doing.

SQL has been the language of databases since 1970. It predates the internet, mobile phones, and most programming languages you use daily. It's still the most important technical skill for anyone building data-backed applications. Not because it's trendy, but because it works, and because every ORM, query builder, and database tool generates SQL under the hood. Understanding SQL means understanding what your tools are actually doing.

## The 30-Second Version

A database stores structured data in tables (like spreadsheets with strict rules). Each table has columns (name, email, age) and rows (one per record). SQL (Structured Query Language) is how you talk to the database: SELECT reads data, INSERT adds data, UPDATE changes data, DELETE removes data. Tables connect to each other through relationships (a user "has many" orders, each order "belongs to" a user). Indexes make queries fast by creating lookup shortcuts, like the index at the back of a textbook. Without them, the database reads every single row to find what you asked for.

## The Real Explanation (ELI5)

Imagine you run a library. Not a big fancy one, just a room with books. At first you have 50 books and you pile them on a table. Someone asks for a book about cooking? You scan the pile. Takes 30 seconds. Fine.

Now you have 50,000 books. Piling them on a table doesn't work anymore. So you do three things:

1. **You organize them into shelves by category** (Fiction, Science, Cooking, History). This is like a table. Each shelf has the same kind of thing.

2. **You give each book a card with standard fields**: title, author, year, genre, shelf location. The card always has the same fields. You can't just scribble random notes. This is like a row with columns. Every card (row) follows the same structure (columns).

3. **You create an index**: an alphabetical list of all authors with the shelf locations of their books. Now when someone asks "do you have anything by Amitav Ghosh?" you don't scan 50,000 books. You look up "Ghosh" in the index, find the shelf number, and walk straight there. This is a database index.

Now replace "books" with data, "shelves" with tables, "cards" with rows, "standard fields" with columns, and "alphabetical list" with a database index. That's a relational database.

SQL is how you talk to the librarian:
- "Show me all cooking books from 2024" → `SELECT * FROM books WHERE genre = 'cooking' AND year = 2024`
- "Add a new book" → `INSERT INTO books (title, author, genre, year) VALUES ('The Guide', 'R.K. Narayan', 'fiction', 1958)`
- "Change this book's genre" → `UPDATE books SET genre = 'literary fiction' WHERE id = 42`
- "Remove this book" → `DELETE FROM books WHERE id = 42`

The analogy breaks at scale: a real database handles millions of "books," serves thousands of "visitors" simultaneously, and can answer complex questions across multiple "shelves" in milliseconds. A librarian can't.

## How It Actually Works

### Tables, Rows, and Columns

A table is like a spreadsheet with strict rules:

```
Table: users
┌────┬───────────┬──────────────────────┬──────────┬─────────────────────┐
│ id │ name      │ email                │ role     │ created_at          │
├────┼───────────┼──────────────────────┼──────────┼─────────────────────┤
│  1 │ Priya     │ priya@example.com    │ admin    │ 2025-01-15 10:30:00 │
│  2 │ Rahul     │ rahul@example.com    │ user     │ 2025-02-20 14:22:00 │
│  3 │ Sara      │ sara@example.com     │ user     │ 2025-03-01 09:00:00 │
└────┴───────────┴──────────────────────┴──────────┴─────────────────────┘

Table: orders
┌────┬─────────┬────────┬────────────┬─────────────────────┐
│ id │ user_id │ total  │ status     │ created_at          │
├────┼─────────┼────────┼────────────┼─────────────────────┤
│  1 │       1 │  2500  │ completed  │ 2025-01-20 11:00:00 │
│  2 │       1 │  1800  │ completed  │ 2025-02-15 16:30:00 │
│  3 │       2 │  4200  │ pending    │ 2025-03-10 08:45:00 │
│  4 │       3 │   950  │ cancelled  │ 2025-03-12 13:00:00 │
└────┴─────────┴────────┴────────────┴─────────────────────┘
```

**Where the spreadsheet model breaks down:**

- **Data types are enforced.** You can't put "hello" in an integer column. A spreadsheet lets you put anything anywhere. A database won't.
- **Columns can have constraints.** `email` can be marked as UNIQUE (no duplicates), `name` can be marked as NOT NULL (must have a value), `role` can have a CHECK constraint (must be 'admin' or 'user').
- **Tables reference each other.** The `user_id` column in `orders` points to the `id` column in `users`. This is a foreign key. The database enforces this: you can't create an order for a user that doesn't exist.
- **Every row has a primary key.** Usually `id`, an auto-incrementing integer or a UUID. This is how you uniquely identify each row. No two rows can have the same primary key.

### SQL vs NoSQL

| | SQL (Relational) | NoSQL (Document/Key-Value) |
|---|---|---|
| **Data structure** | Tables with fixed columns | Flexible documents (JSON-like), key-value pairs |
| **Schema** | Defined upfront, enforced | Flexible, can vary per document |
| **Relationships** | Built-in (foreign keys, JOINs) | You manage them yourself |
| **Query language** | SQL (standardized) | Varies per database |
| **Best for** | Structured data with relationships | Unstructured data, extreme scale, caching |
| **Examples** | PostgreSQL, MySQL, SQLite | MongoDB, Redis, DynamoDB, Firebase |

**When to use SQL (most of the time):**
- Your data has clear relationships (users have orders, orders have items).
- You need to query data in flexible ways (filter, join, aggregate, group).
- Data integrity matters (don't want orphaned orders, duplicate emails, or inconsistent balances).
- You're building a web app, SaaS product, CRM, e-commerce store, or anything with "normal" data.

**When to use NoSQL:**
- **Redis:** Caching, session storage, rate limiting. Data you can afford to lose.
- **MongoDB/Firebase:** Rapid prototyping where the schema is genuinely unknown and changing daily. Note: most apps don't stay in this phase for long, and migrating from MongoDB to PostgreSQL later is painful.
- **DynamoDB:** You need extreme scale (millions of writes per second) and can design around key-value access patterns.

**The honest answer:** Use PostgreSQL. It handles 99% of use cases, supports JSON columns for semi-structured data, has full-text search built in, and scales further than most apps will ever need. The projects that genuinely need NoSQL know they need it before asking the question.

### Core SQL

**SELECT (reading data):**

```sql
-- Get all users
SELECT * FROM users;

-- Get specific columns
SELECT name, email FROM users;

-- Filter with WHERE
SELECT * FROM users WHERE role = 'admin';

-- Multiple conditions
SELECT * FROM users WHERE role = 'user' AND created_at > '2025-02-01';

-- Sort results
SELECT * FROM users ORDER BY created_at DESC;

-- Limit results
SELECT * FROM users ORDER BY created_at DESC LIMIT 10;

-- Count rows
SELECT COUNT(*) FROM users WHERE role = 'admin';

-- Aggregate functions
SELECT role, COUNT(*) as user_count FROM users GROUP BY role;
-- Result: admin: 1, user: 2
```

**INSERT (creating data):**

```sql
-- Insert one row
INSERT INTO users (name, email, role)
VALUES ('Amit', 'amit@example.com', 'user');

-- Insert multiple rows
INSERT INTO users (name, email, role) VALUES
  ('Deepa', 'deepa@example.com', 'user'),
  ('Karan', 'karan@example.com', 'admin');

-- Insert and return the created row (PostgreSQL)
INSERT INTO users (name, email, role)
VALUES ('Neha', 'neha@example.com', 'user')
RETURNING id, name, email, created_at;
```

**UPDATE (changing data):**

```sql
-- Update one row
UPDATE users SET role = 'admin' WHERE id = 2;

-- Update multiple columns
UPDATE users SET name = 'Rahul K', email = 'rahulk@example.com' WHERE id = 2;

-- Update multiple rows
UPDATE orders SET status = 'cancelled' WHERE status = 'pending' AND created_at < '2025-01-01';

-- DANGER: forgetting WHERE updates EVERY row
UPDATE users SET role = 'admin';  -- Everyone is now an admin. Oops.
```

**DELETE (removing data):**

```sql
-- Delete one row
DELETE FROM users WHERE id = 4;

-- Delete matching rows
DELETE FROM orders WHERE status = 'cancelled';

-- DANGER: forgetting WHERE deletes EVERYTHING
DELETE FROM users;  -- All users gone. Game over.
```

The `WHERE` clause is your safety net. An UPDATE or DELETE without WHERE affects every row in the table. This is not a hypothetical danger. It happens in production. Some teams require `WHERE` on every UPDATE/DELETE as a policy, and for good reason.

### JOINs: Combining Tables

JOINs let you combine data from two or more tables based on a relationship. Forget Venn diagrams. Look at actual data.

Here are our two tables:

```
users:                           orders:
┌────┬───────┐                   ┌────┬─────────┬────────┐
│ id │ name  │                   │ id │ user_id │ total  │
├────┼───────┤                   ├────┼─────────┼────────┤
│  1 │ Priya │                   │  1 │       1 │  2500  │
│  2 │ Rahul │                   │  2 │       1 │  1800  │
│  3 │ Sara  │                   │  3 │       2 │  4200  │
└────┴───────┘                   │  4 │      99 │   500  │ ← user 99 doesn't exist
                                 └────┴─────────┴────────┘
```

Notice: Sara (id=3) has no orders. Order 4 references user_id=99, who doesn't exist in the users table.

**INNER JOIN:** Only rows where BOTH tables match.

```sql
SELECT users.name, orders.total
FROM users
INNER JOIN orders ON users.id = orders.user_id;
```

```
┌───────┬────────┐
│ name  │ total  │
├───────┼────────┤
│ Priya │  2500  │  ← Priya's order
│ Priya │  1800  │  ← Priya's other order
│ Rahul │  4200  │  ← Rahul's order
└───────┴────────┘
-- Sara is MISSING (no matching orders)
-- Order 4 is MISSING (no matching user)
```

**LEFT JOIN:** All rows from the LEFT table, even if no match on the right.

```sql
SELECT users.name, orders.total
FROM users
LEFT JOIN orders ON users.id = orders.user_id;
```

```
┌───────┬────────┐
│ name  │ total  │
├───────┼────────┤
│ Priya │  2500  │
│ Priya │  1800  │
│ Rahul │  4200  │
│ Sara  │  NULL  │  ← Sara included, but total is NULL (no orders)
└───────┴────────┘
-- Order 4 is still MISSING (LEFT JOIN keeps left table rows, not unmatched right)
```

LEFT JOIN is what you use when you want "all users and their orders, including users with zero orders."

**RIGHT JOIN:** All rows from the RIGHT table, even if no match on the left. It's the mirror of LEFT JOIN. Rarely used because you can just swap the table order and use LEFT JOIN.

```sql
SELECT users.name, orders.total
FROM users
RIGHT JOIN orders ON users.id = orders.user_id;
```

```
┌───────┬────────┐
│ name  │ total  │
├───────┼────────┤
│ Priya │  2500  │
│ Priya │  1800  │
│ Rahul │  4200  │
│ NULL  │   500  │  ← Order 4 included, but name is NULL (user 99 doesn't exist)
└───────┴────────┘
-- Sara is MISSING (RIGHT JOIN keeps right table rows, not unmatched left)
```

**When to use which:**
- **INNER JOIN:** You only want rows where the relationship exists. "Show me users and their orders" (skip users with no orders).
- **LEFT JOIN:** You want ALL rows from the main table regardless. "Show me all users with their orders (if any)." This is the most common JOIN.
- **RIGHT JOIN:** Almost never. Rewrite it as a LEFT JOIN with the tables swapped.

### Indexes: Making Queries Fast

Without an index, the database reads every row in the table to find matches. This is called a full table scan. With 100 rows, it's instant. With 10 million rows, it takes seconds, and seconds matter.

```
WITHOUT INDEX (full table scan):
Query: SELECT * FROM users WHERE email = 'priya@example.com'

  Row 1: email = 'rahul@...'   → no
  Row 2: email = 'sara@...'    → no
  Row 3: email = 'priya@...'   → YES ← found it, but still checks remaining rows
  Row 4: email = 'amit@...'    → no
  ...
  Row 10,000,000: email = '...' → no

  Checked: 10,000,000 rows. Time: ~2 seconds.


WITH INDEX on email column:
Query: SELECT * FROM users WHERE email = 'priya@example.com'

  Index lookup: 'priya@example.com' → Row 3

  Checked: ~3 index entries (binary search). Time: <1 millisecond.
```

**Creating indexes:**

```sql
-- Index on a single column
CREATE INDEX idx_users_email ON users (email);

-- Unique index (also enforces uniqueness)
CREATE UNIQUE INDEX idx_users_email ON users (email);

-- Composite index (for queries that filter on multiple columns)
CREATE INDEX idx_orders_user_status ON orders (user_id, status);

-- Check existing indexes (PostgreSQL)
SELECT indexname, indexdef FROM pg_indexes WHERE tablename = 'users';
```

**When to add an index:**
- On columns you filter by in WHERE clauses (`WHERE email = ...`, `WHERE status = ...`)
- On columns you JOIN on (`ON users.id = orders.user_id`)
- On columns you ORDER BY frequently
- On columns with UNIQUE constraints (the database creates one automatically)

**When NOT to add an index:**
- On tables with fewer than a few thousand rows (full scan is already fast).
- On columns you rarely query by (every index slows down INSERT/UPDATE because the index must be updated too).
- On columns with very low cardinality (a boolean column with only true/false gives the index very little to work with).

**The rule of thumb:** If a query is slow, check if the column you're filtering on has an index. If it doesn't, add one. This single optimization fixes 80% of database performance issues.

### Relationships

Relationships connect tables to each other. There are three types:

**One-to-one:** Each row in table A matches exactly one row in table B.

```sql
-- A user has one profile
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email TEXT UNIQUE NOT NULL
);

CREATE TABLE profiles (
  id SERIAL PRIMARY KEY,
  user_id INTEGER UNIQUE REFERENCES users(id),  -- UNIQUE = one-to-one
  bio TEXT,
  avatar_url TEXT
);
```

Uncommon. Usually you'd just put the profile columns in the users table. Use one-to-one when the second table has lots of optional columns or is accessed separately.

**One-to-many:** Each row in table A can match many rows in table B. The most common relationship.

```sql
-- A user has many orders
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL
);

CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id),  -- foreign key, NOT unique = one-to-many
  total INTEGER NOT NULL,
  status TEXT NOT NULL DEFAULT 'pending'
);

-- Query: get a user's orders
SELECT * FROM orders WHERE user_id = 42;
```

The `REFERENCES users(id)` is a foreign key constraint. It tells the database: "this value must exist in the users table's id column." If you try to create an order with `user_id = 999` and user 999 doesn't exist, the database rejects it.

**Many-to-many:** Rows in table A can match many rows in table B, and vice versa. Requires a junction table (also called a join table or linking table).

```sql
-- A product can be in many orders, and an order can have many products
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  price INTEGER NOT NULL  -- price in paise/cents, never float
);

CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Junction table
CREATE TABLE order_items (
  id SERIAL PRIMARY KEY,
  order_id INTEGER REFERENCES orders(id),
  product_id INTEGER REFERENCES products(id),
  quantity INTEGER NOT NULL DEFAULT 1,
  unit_price INTEGER NOT NULL  -- price at time of purchase
);

-- Query: get all products in order 7
SELECT p.name, oi.quantity, oi.unit_price
FROM order_items oi
JOIN products p ON oi.product_id = p.id
WHERE oi.order_id = 7;

-- Query: get all orders that contain product 3
SELECT o.id, o.created_at, oi.quantity
FROM order_items oi
JOIN orders o ON oi.order_id = o.id
WHERE oi.product_id = 3;
```

The junction table `order_items` holds the relationship. It also holds relationship-specific data: the quantity and the price at the time of purchase (which might differ from the product's current price).

### ORMs: Talking to Databases in Your Language

An ORM (Object-Relational Mapper) lets you interact with the database using your programming language instead of writing raw SQL.

**Prisma (Node.js/TypeScript):**

```typescript
// Define schema in schema.prisma
model User {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  orders    Order[]
  createdAt DateTime @default(now())
}

model Order {
  id        Int      @id @default(autoincrement())
  user      User     @relation(fields: [userId], references: [id])
  userId    Int
  total     Int
  status    String   @default("pending")
}

// Use in code
const users = await prisma.user.findMany({
  where: { role: "admin" },
  include: { orders: true },  // JOINs automatically
  orderBy: { createdAt: "desc" },
  take: 10,
});
```

**Drizzle (Node.js/TypeScript):**

```typescript
// Define schema
const users = pgTable("users", {
  id: serial("id").primaryKey(),
  name: text("name").notNull(),
  email: text("email").unique().notNull(),
  createdAt: timestamp("created_at").defaultNow(),
});

// Query (SQL-like syntax)
const result = await db
  .select()
  .from(users)
  .where(eq(users.role, "admin"))
  .orderBy(desc(users.createdAt))
  .limit(10);
```

**SQLAlchemy (Python):**

```python
class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True)
    name = Column(String, nullable=False)
    email = Column(String, unique=True, nullable=False)
    orders = relationship("Order", back_populates="user")

# Query
admins = session.query(User).filter(User.role == "admin").order_by(User.created_at.desc()).limit(10).all()
```

**When to use raw SQL instead of the ORM:**
- Complex queries with multiple JOINs, subqueries, or window functions. The ORM syntax gets harder to read than SQL.
- Performance-critical queries where you need precise control over what the database does.
- Bulk operations (updating 10,000 rows at once). ORMs often generate one query per row.
- Database-specific features (PostgreSQL full-text search, JSON operations, recursive CTEs).
- When the ORM generates bad queries. Always check the SQL your ORM produces.

**The best approach:** Use the ORM for simple CRUD. Drop to raw SQL for complex queries. Most ORMs support raw SQL alongside their query builder.

### Migrations: Changing the Database Safely

A migration is a versioned script that changes the database schema (add a table, add a column, rename a field, add an index). You can't just edit the database directly in production for three reasons:

1. **Other developers need the same changes.** If you add a column on your machine, nobody else has it. Migrations are code that runs everywhere.
2. **Production databases have data.** You can't drop a table and recreate it. You need to ALTER it. Migrations handle this.
3. **You need to go back.** If a change breaks something, you need to reverse it. Migrations have "up" (apply) and "down" (reverse) scripts.

**Prisma migrations:**

```bash
# Edit your schema.prisma file, then:
npx prisma migrate dev --name add_phone_to_users

# This creates a migration file like:
# prisma/migrations/20250317_add_phone_to_users/migration.sql
# Contents:
# ALTER TABLE "users" ADD COLUMN "phone" TEXT;
```

**Raw SQL migrations (with a tool like golang-migrate or Flyway):**

```sql
-- 001_create_users.up.sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  email TEXT UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- 001_create_users.down.sql
DROP TABLE users;

-- 002_add_phone_to_users.up.sql
ALTER TABLE users ADD COLUMN phone TEXT;

-- 002_add_phone_to_users.down.sql
ALTER TABLE users DROP COLUMN phone;
```

**Migration rules:**
- Never edit a migration that's already been run in production. Create a new one.
- Always test migrations on a copy of production data before running them on production.
- Migrations that delete columns or tables are dangerous. You can't undo data loss. Consider renaming the column first, deploying the code change, then deleting the column in a later migration.
- Keep migrations small. One logical change per migration. "Add users table" is one migration. "Add users table and orders table and create indexes" should be three.

## The Mental Model

**Mental model 1: "A database is a very strict spreadsheet."** Like a spreadsheet, it has rows and columns. Unlike a spreadsheet, every column has a fixed type (you can't put text in a number column), columns can have rules (UNIQUE, NOT NULL, REFERENCES), and multiple sheets (tables) can reference each other with enforced relationships. This strictness is the point: it prevents bad data from ever getting in.

**Mental model 2: "Indexes are the table of contents."** Without an index, the database reads every page (row) to find what you're looking for. With an index, it looks up the topic (value) in the table of contents (index) and jumps straight to the right page (row). Adding an index makes reading faster but makes writing slower (you have to update the table of contents every time you add a page). Most apps read far more than they write, so the tradeoff is almost always worth it.

**Mental model 3: "SQL is declarative. You say what you want, not how to get it."** When you write `SELECT * FROM users WHERE age > 25 ORDER BY name`, you don't tell the database how to scan the table, which algorithm to use for sorting, or how to allocate memory. You describe the result you want, and the database figures out the best way to get it. This is why understanding indexes matters: they change HOW the database executes your query, even though your SQL stays the same.

## Key Concepts (Jargon Decoder)

| Term | Plain English | When You'll See It |
|------|--------------|-------------------|
| Table | A structured collection of data with fixed columns (like a spreadsheet tab) | Defining your data model |
| Row / Record | One entry in a table (like one row in a spreadsheet) | Every query result |
| Column / Field | One property that every row has (like a spreadsheet column header) | Schema definition |
| Primary key | A unique identifier for each row (usually `id`) | Every table must have one |
| Foreign key | A column that references the primary key of another table | Creating relationships between tables |
| Schema | The structure of your database: what tables exist, what columns they have, what types and constraints | Database design, migrations |
| Query | A SQL statement that asks the database to do something | Every database interaction |
| Index | A data structure that makes queries on specific columns faster | Performance optimization |
| JOIN | Combining rows from two or more tables based on a related column | Querying related data |
| Transaction | A group of operations that either ALL succeed or ALL fail | Payments, transfers, multi-step writes |
| Migration | A versioned script that changes the database schema | Deploying schema changes |
| ORM | A library that lets you query the database in your programming language instead of SQL | Prisma, Drizzle, SQLAlchemy |
| Constraint | A rule enforced by the database (UNIQUE, NOT NULL, CHECK, REFERENCES) | Data integrity |
| Normalization | Organizing data to reduce duplication (putting addresses in their own table instead of repeating them) | Schema design |
| N+1 problem | Making 1 query to get a list, then N more queries to get details for each item | Performance bugs |
| Full table scan | When the database reads every row to answer your query (usually means missing index) | Slow query investigation |
| ACID | Atomicity, Consistency, Isolation, Durability, the guarantees a relational database provides | Choosing databases, understanding transactions |
| Soft delete | Marking rows as deleted (setting a `deleted_at` timestamp) instead of actually removing them | Data retention, undo features |

## Common Patterns

### 1. User-Posts-Comments Schema

**When to use it:** Any content-based app (blog, forum, social network, discussion board).

**How it works:** Users have many posts, posts have many comments, comments belong to a user and a post.

**Code sketch:**

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  username TEXT UNIQUE NOT NULL,
  email TEXT UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL REFERENCES users(id),
  title TEXT NOT NULL,
  body TEXT NOT NULL,
  published BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE comments (
  id SERIAL PRIMARY KEY,
  post_id INTEGER NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
  user_id INTEGER NOT NULL REFERENCES users(id),
  body TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Indexes for common queries
CREATE INDEX idx_posts_user_id ON posts (user_id);
CREATE INDEX idx_comments_post_id ON comments (post_id);

-- Get a post with its comments and comment authors
SELECT
  p.title,
  p.body,
  c.body AS comment_body,
  u.username AS comment_author,
  c.created_at AS commented_at
FROM posts p
LEFT JOIN comments c ON c.post_id = p.id
LEFT JOIN users u ON c.user_id = u.id
WHERE p.id = 42
ORDER BY c.created_at ASC;
```

`ON DELETE CASCADE` means: when a post is deleted, automatically delete all its comments too. Without this, deleting a post with comments would fail because of the foreign key constraint.

### 2. E-commerce Schema (Products/Orders/Line Items)

**When to use it:** Any app where users buy things.

**How it works:** Products exist independently. Orders belong to users. Line items connect orders to products (many-to-many through a junction table).

**Code sketch:**

```sql
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  description TEXT,
  price_cents INTEGER NOT NULL,  -- NEVER use float for money
  stock INTEGER NOT NULL DEFAULT 0,
  active BOOLEAN DEFAULT TRUE
);

CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL REFERENCES users(id),
  status TEXT NOT NULL DEFAULT 'pending'
    CHECK (status IN ('pending', 'confirmed', 'shipped', 'delivered', 'cancelled')),
  total_cents INTEGER NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE order_items (
  id SERIAL PRIMARY KEY,
  order_id INTEGER NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  product_id INTEGER NOT NULL REFERENCES products(id),
  quantity INTEGER NOT NULL CHECK (quantity > 0),
  unit_price_cents INTEGER NOT NULL  -- snapshot of price at purchase time
);

CREATE INDEX idx_orders_user_id ON orders (user_id);
CREATE INDEX idx_order_items_order_id ON order_items (order_id);

-- Get order details with product info
SELECT
  o.id AS order_id,
  o.status,
  p.name AS product,
  oi.quantity,
  oi.unit_price_cents,
  (oi.quantity * oi.unit_price_cents) AS line_total_cents
FROM orders o
JOIN order_items oi ON oi.order_id = o.id
JOIN products p ON oi.product_id = p.id
WHERE o.id = 7;
```

Notice: `unit_price_cents` in `order_items` stores the price at purchase time, not a reference to the product's current price. Products change prices. Orders should not retroactively change.

### 3. Full-Text Search

**When to use it:** Users need to search by keywords across text fields (product search, article search, contact search).

**How it works:** PostgreSQL has built-in full-text search. No need for Elasticsearch for most use cases.

**Code sketch:**

```sql
-- Add a search vector column
ALTER TABLE products ADD COLUMN search_vector tsvector;

-- Populate it (combine name and description)
UPDATE products SET search_vector =
  to_tsvector('english', coalesce(name, '') || ' ' || coalesce(description, ''));

-- Create an index on the search vector (GIN index for full-text)
CREATE INDEX idx_products_search ON products USING GIN (search_vector);

-- Search for products
SELECT name, description,
  ts_rank(search_vector, to_tsquery('english', 'wireless & headphones')) AS rank
FROM products
WHERE search_vector @@ to_tsquery('english', 'wireless & headphones')
ORDER BY rank DESC
LIMIT 20;

-- Keep the search vector updated automatically
CREATE OR REPLACE FUNCTION update_search_vector() RETURNS TRIGGER AS $$
BEGIN
  NEW.search_vector := to_tsvector('english',
    coalesce(NEW.name, '') || ' ' || coalesce(NEW.description, ''));
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER products_search_update
  BEFORE INSERT OR UPDATE ON products
  FOR EACH ROW EXECUTE FUNCTION update_search_vector();
```

### 4. Soft Deletes

**When to use it:** You need to "delete" records but keep them recoverable (legal compliance, undo features, audit requirements).

**How it works:** Instead of DELETE, you set a `deleted_at` timestamp. All queries filter out deleted records.

**Code sketch:**

```sql
ALTER TABLE contacts ADD COLUMN deleted_at TIMESTAMP;

-- "Delete" a contact
UPDATE contacts SET deleted_at = NOW() WHERE id = 42;

-- List contacts (only active ones)
SELECT * FROM contacts WHERE deleted_at IS NULL;

-- "Restore" a contact
UPDATE contacts SET deleted_at = NULL WHERE id = 42;

-- Actually purge old deleted records (run periodically)
DELETE FROM contacts WHERE deleted_at < NOW() - INTERVAL '90 days';

-- Partial index: only index non-deleted rows (saves space and speeds up queries)
CREATE INDEX idx_contacts_active ON contacts (email) WHERE deleted_at IS NULL;
```

The downside: every query must include `WHERE deleted_at IS NULL`. If you forget, you'll show deleted records. Many ORMs have built-in soft delete support that adds this filter automatically.

### 5. Audit Logs

**When to use it:** You need to track who changed what and when (compliance, debugging, undo).

**How it works:** A separate table records every change to the main tables. A trigger or application-level code writes to it automatically.

**Code sketch:**

```sql
CREATE TABLE audit_log (
  id SERIAL PRIMARY KEY,
  table_name TEXT NOT NULL,
  record_id INTEGER NOT NULL,
  action TEXT NOT NULL CHECK (action IN ('INSERT', 'UPDATE', 'DELETE')),
  old_data JSONB,
  new_data JSONB,
  changed_by INTEGER REFERENCES users(id),
  changed_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_audit_table_record ON audit_log (table_name, record_id);
CREATE INDEX idx_audit_changed_at ON audit_log (changed_at);

-- PostgreSQL trigger to auto-log changes
CREATE OR REPLACE FUNCTION audit_trigger() RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO audit_log (table_name, record_id, action, old_data, new_data)
  VALUES (
    TG_TABLE_NAME,
    COALESCE(NEW.id, OLD.id),
    TG_OP,
    CASE WHEN TG_OP IN ('UPDATE', 'DELETE') THEN to_jsonb(OLD) END,
    CASE WHEN TG_OP IN ('INSERT', 'UPDATE') THEN to_jsonb(NEW) END
  );
  RETURN COALESCE(NEW, OLD);
END;
$$ LANGUAGE plpgsql;

-- Apply to any table
CREATE TRIGGER audit_contacts
  AFTER INSERT OR UPDATE OR DELETE ON contacts
  FOR EACH ROW EXECUTE FUNCTION audit_trigger();

-- Query: what happened to contact 42?
SELECT action, old_data, new_data, changed_at
FROM audit_log
WHERE table_name = 'contacts' AND record_id = 42
ORDER BY changed_at DESC;
```

## Mistakes Everyone Makes

### 1. No indexes on columns you filter by

**What people do:** Create tables, write queries with WHERE clauses, never add indexes.

**Why it seems right:** "It works fine in development with 50 rows."

**What actually happens:** In production with 100,000+ rows, queries that should take 2 milliseconds take 2 seconds. The database does a full table scan every time. Pages load slowly. Users complain. You throw more server resources at it instead of adding a one-line index.

**What to do instead:** Add an index on every column that appears in a WHERE clause, a JOIN condition, or an ORDER BY. Check query performance with EXPLAIN ANALYZE:

```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 42 AND status = 'pending';
-- Look for "Seq Scan" (bad, full scan) vs "Index Scan" (good, using index)
```

### 2. The N+1 query problem

**What people do:** Fetch a list of items, then loop through and make one query per item to get related data.

**Why it seems right:** "I need the user's orders, and for each order I need the items."

**What actually happens:** You make 1 query to get 100 orders, then 100 queries to get items for each order. That's 101 database round trips. Each one has network latency. The page takes 3 seconds to load instead of 50 milliseconds.

**What to do instead:** Use a JOIN or an IN query to fetch everything in one or two queries.

```python
# BAD: N+1
orders = db.query("SELECT * FROM orders WHERE user_id = 42")
for order in orders:
    items = db.query(f"SELECT * FROM order_items WHERE order_id = {order.id}")
    order.items = items
# Total: 1 + N queries

# GOOD: 1 query with JOIN
orders_with_items = db.query("""
  SELECT o.*, oi.product_id, oi.quantity, oi.unit_price_cents
  FROM orders o
  LEFT JOIN order_items oi ON oi.order_id = o.id
  WHERE o.user_id = 42
""")
# Total: 1 query

# GOOD: 2 queries with IN
orders = db.query("SELECT * FROM orders WHERE user_id = 42")
order_ids = [o.id for o in orders]
items = db.query("SELECT * FROM order_items WHERE order_id = ANY($1)", order_ids)
# Total: 2 queries regardless of how many orders
```

### 3. Storing money as floats

**What people do:** `price FLOAT` or `total DECIMAL(10,2)` (better) or, worst case, `price REAL`.

**Why it seems right:** "Money has decimal places, so a decimal type makes sense."

**What actually happens:** Floating point math is not exact. `0.1 + 0.2 = 0.30000000000000004` in most languages. Over thousands of transactions, tiny errors compound. Your books don't balance. Refunds are off by a cent. Financial audits fail.

**What to do instead:** Store money as integers in the smallest denomination (cents, paise). Rs. 42.50 is stored as `4250`. All math is integer math (exact). Convert to the display format only in the UI layer.

```sql
-- GOOD
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  price_cents INTEGER NOT NULL  -- Rs. 42.50 = 4250
);

-- In your app, display:
-- const displayPrice = (priceCents / 100).toFixed(2)
```

### 4. Not using transactions for multi-step writes

**What people do:** Run multiple INSERT/UPDATE queries separately when they need to happen together.

**Why it seems right:** "Each query works fine individually."

**What actually happens:** Query 1 succeeds (debit the account). Query 2 fails (credit the other account). Now money has disappeared. The database is in an inconsistent state. This is the classic "partial write" problem.

**What to do instead:** Wrap related operations in a transaction. Either all succeed or all fail.

```sql
-- Transfer Rs. 1000 from account 1 to account 2
BEGIN;
  UPDATE accounts SET balance = balance - 100000 WHERE id = 1;
  UPDATE accounts SET balance = balance + 100000 WHERE id = 2;
COMMIT;
-- If either UPDATE fails, both are rolled back. Money is never lost.
```

```python
# SQLAlchemy
with session.begin():
    session.execute(text("UPDATE accounts SET balance = balance - 100000 WHERE id = 1"))
    session.execute(text("UPDATE accounts SET balance = balance + 100000 WHERE id = 2"))
# Auto-commits on success, auto-rolls-back on any exception
```

### 5. Letting the ORM generate horrible queries without checking

**What people do:** Use the ORM's query builder for everything and never look at the generated SQL.

**Why it seems right:** "That's the whole point of an ORM, so I don't have to think about SQL."

**What actually happens:** The ORM generates 5 separate queries when one JOIN would do. Or it loads an entire table into memory and filters in application code. Or it creates a subquery inside a subquery inside a subquery that the database can't optimize. The page is slow and nobody knows why because nobody's looking at the SQL.

**What to do instead:** Enable query logging during development. Look at the SQL your ORM generates. Count the number of queries per page load. If it's more than 5-10, something is wrong.

```python
# SQLAlchemy: enable query logging
import logging
logging.getLogger("sqlalchemy.engine").setLevel(logging.INFO)

# Prisma: enable query logging
const prisma = new PrismaClient({ log: ['query'] })

# Drizzle: enable query logging
const db = drizzle(pool, { logger: true })
```

### 6. No foreign key constraints

**What people do:** Create `user_id INTEGER` columns without `REFERENCES users(id)`.

**Why it seems right:** "I'll just make sure my code always uses valid IDs."

**What actually happens:** A bug creates an order with `user_id = 99999` (doesn't exist). Or you delete a user but their orders still reference them. Now you have orphaned data. JOINs return weird results. Reports are wrong. The app crashes when it tries to display the order's user info.

**What to do instead:** Always add foreign key constraints. They're free protection against data corruption.

```sql
-- The database will reject any order with an invalid user_id
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL REFERENCES users(id),
  total_cents INTEGER NOT NULL
);
```

## Best Practices (The Rules That Prevent 90% of Problems)

1. **Add an index on every column you filter, join, or sort by.** Check with EXPLAIN ANALYZE. If you see "Seq Scan" on a table with more than a few thousand rows, add an index.

2. **Store money as integers (cents/paise), never as floats.** Display formatting happens in the UI, not in the database. This prevents every category of floating-point rounding error.

3. **Use transactions for any operation that touches multiple rows or tables.** If step 1 succeeds and step 2 must also succeed, they belong in a transaction. No exceptions.

4. **Always add foreign key constraints.** They catch bugs at the database level before bad data spreads through your app. The tiny performance cost is worth it.

5. **Check your ORM's generated SQL.** Enable query logging during development. Count queries per page load. If you see N+1 patterns, fix them with JOINs or eager loading.

6. **Never run UPDATE or DELETE without a WHERE clause.** Make it a habit. Some teams even use database-level safeguards to prevent this.

7. **Use migrations for every schema change.** Never manually ALTER a production database. Migrations are versioned, reviewable, reversible, and reproducible.

8. **Name things consistently: `snake_case` for columns and tables, singular table names or plural, just pick one.** `user_id` not `userId`. `created_at` not `createdDate`. Consistency prevents the "was it camelCase or snake_case?" question at 2 AM.

## The "Just Tell Me What to Do" Quickstart

Build a working database with proper schema, relationships, and indexes in under 10 minutes using SQLite (no setup needed).

**Step 1: Install SQLite** (probably already installed)

```bash
# macOS: already installed
sqlite3 --version

# If not:
brew install sqlite
```

**Step 2: Create a database and schema**

```bash
sqlite3 bookstore.db
```

Paste this into the SQLite prompt:

```sql
-- Create tables
CREATE TABLE authors (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT NOT NULL,
  bio TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE books (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  title TEXT NOT NULL,
  author_id INTEGER NOT NULL REFERENCES authors(id),
  price_cents INTEGER NOT NULL,
  genre TEXT NOT NULL,
  published_year INTEGER,
  stock INTEGER NOT NULL DEFAULT 0,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE customers (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT NOT NULL,
  email TEXT UNIQUE NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE orders (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  customer_id INTEGER NOT NULL REFERENCES customers(id),
  total_cents INTEGER NOT NULL,
  status TEXT NOT NULL DEFAULT 'pending',
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE order_items (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  order_id INTEGER NOT NULL REFERENCES orders(id),
  book_id INTEGER NOT NULL REFERENCES books(id),
  quantity INTEGER NOT NULL DEFAULT 1,
  unit_price_cents INTEGER NOT NULL
);

-- Indexes
CREATE INDEX idx_books_author ON books (author_id);
CREATE INDEX idx_books_genre ON books (genre);
CREATE INDEX idx_orders_customer ON orders (customer_id);
CREATE INDEX idx_order_items_order ON order_items (order_id);

-- Seed data
INSERT INTO authors (name, bio) VALUES
  ('Arundhati Roy', 'Indian author and political activist'),
  ('Chetan Bhagat', 'Indian author and columnist'),
  ('Amitav Ghosh', 'Indian author known for historical fiction');

INSERT INTO books (title, author_id, price_cents, genre, published_year, stock) VALUES
  ('The God of Small Things', 1, 49900, 'literary fiction', 1997, 25),
  ('The Ministry of Utmost Happiness', 1, 59900, 'literary fiction', 2017, 15),
  ('Five Point Someone', 2, 19900, 'fiction', 2004, 50),
  ('The Hungry Tide', 3, 44900, 'literary fiction', 2004, 10),
  ('Sea of Poppies', 3, 54900, 'historical fiction', 2008, 20);

INSERT INTO customers (name, email) VALUES
  ('Priya Sharma', 'priya@example.com'),
  ('Rahul Patel', 'rahul@example.com');

INSERT INTO orders (customer_id, total_cents, status) VALUES
  (1, 109800, 'completed'),
  (2, 19900, 'pending');

INSERT INTO order_items (order_id, book_id, quantity, unit_price_cents) VALUES
  (1, 1, 1, 49900),
  (1, 4, 1, 44900),
  (1, 5, 1, 54900),  -- Note: intentionally doesn't match total for demo
  (2, 3, 1, 19900);
```

**Step 3: Run some queries**

```sql
-- All books by Arundhati Roy
SELECT b.title, b.price_cents / 100.0 AS price_rupees, b.published_year
FROM books b
JOIN authors a ON b.author_id = a.id
WHERE a.name = 'Arundhati Roy';

-- Customer order history
SELECT c.name, o.id AS order_id, o.status, o.total_cents / 100.0 AS total_rupees,
       GROUP_CONCAT(b.title, ', ') AS books
FROM customers c
JOIN orders o ON o.customer_id = c.id
JOIN order_items oi ON oi.order_id = o.id
JOIN books b ON oi.book_id = b.id
GROUP BY o.id;

-- Books in stock by genre
SELECT genre, COUNT(*) AS book_count, SUM(stock) AS total_stock
FROM books
GROUP BY genre
ORDER BY total_stock DESC;

-- Most popular books (by order count)
SELECT b.title, COUNT(oi.id) AS times_ordered
FROM books b
LEFT JOIN order_items oi ON oi.book_id = b.id
GROUP BY b.id
ORDER BY times_ordered DESC;
```

**Step 4: Break things on purpose**

```sql
-- Try to create an order for a non-existent customer (should fail with FK constraint)
INSERT INTO orders (customer_id, total_cents) VALUES (999, 10000);

-- Try to insert a duplicate email
INSERT INTO customers (name, email) VALUES ('Test', 'priya@example.com');

-- See what EXPLAIN shows (SQLite version of query planning)
EXPLAIN QUERY PLAN SELECT * FROM books WHERE genre = 'fiction';
-- Should show "USING INDEX idx_books_genre"
```

Exit with `.quit`.

## How to Prompt AI Tools About This

### What context to give AI tools

When asking Claude Code, Cursor, or Copilot to write database code, always include:
- The database (PostgreSQL, MySQL, SQLite)
- The ORM or query method (Prisma, Drizzle, raw SQL, SQLAlchemy)
- The entities and their relationships ("users have many orders, orders have many line items")
- Whether you need migrations or just the schema
- Any constraints (unique email, non-negative prices, valid status values)

### Example prompts that produce good results

**For schema design:**
```
Design a PostgreSQL schema for a task management app. Entities: users,
projects, tasks, comments. A user can belong to many projects. Projects
have many tasks. Tasks have an assignee (user), status (todo/in_progress/done),
priority (low/medium/high/urgent), and due date. Comments belong to a task
and a user. Include proper foreign keys, indexes on columns used in WHERE
clauses, and created_at/updated_at timestamps. Use snake_case. Store all
timestamps as TIMESTAMPTZ.
```

**For complex queries:**
```
Write a PostgreSQL query that returns each project's name, the number of
tasks in each status (todo, in_progress, done), and the percentage of
tasks completed. Only include projects with at least one task. Sort by
completion percentage descending.
```

**For debugging slow queries:**
```
This query takes 3 seconds on a table with 500K rows:
SELECT * FROM orders WHERE status = 'pending' AND created_at > '2025-01-01'
ORDER BY created_at DESC LIMIT 20;

What index should I add? Show me the CREATE INDEX statement and explain
why it helps.
```

### What to watch out for in AI-generated database code

- **Missing indexes.** AI creates tables without indexes. Always ask: "What indexes should I add for common queries?"
- **Floats for money.** AI often uses `DECIMAL` or `FLOAT` for prices. Insist on integer cents.
- **No foreign key constraints.** AI sometimes creates `user_id INTEGER` without `REFERENCES`. Always add them.
- **N+1 queries in ORM code.** AI-generated Prisma/SQLAlchemy code often fetches related data in a loop instead of using `include` or `joinedload`.
- **Missing transactions.** Multi-step operations (create order + create order items + update stock) should be in a transaction. AI often skips this.
- **Wrong JOIN type.** AI defaults to INNER JOIN when you need LEFT JOIN (you want to show users even if they have zero orders).

### Key terms that improve output quality

- "Include foreign key constraints" (gets proper referential integrity)
- "Add indexes for common query patterns" (gets performance-ready schema)
- "Use integer cents for money columns" (prevents float errors)
- "Wrap in a transaction" (gets atomic multi-step operations)
- "Show me the EXPLAIN ANALYZE output" (gets query plan analysis)

## Ship It: Build This

### Bookstore Database with CLI

Build a SQLite-backed bookstore management tool with a command-line interface. Manage books, customers, and orders with proper relationships, indexes, and queries.

**Why this project:** It exercises every concept in this guide. You'll design a schema with one-to-many and many-to-many relationships, write queries with JOINs, add indexes, use transactions for order creation, and handle edge cases (ordering a book with no stock, duplicate customer emails).

**Rough architecture:**
- Python with sqlite3 (standard library, no dependencies)
- Single file, ~300 lines
- 5 tables: authors, books, customers, orders, order_items
- CLI commands: `add-book`, `list-books`, `add-customer`, `create-order`, `order-history`, `stock-report`, `search`
- Proper foreign keys and indexes
- Transactions for order creation (create order + insert items + decrement stock, all or nothing)
- Search across book titles and author names
- Formatted output with prices displayed in rupees (stored as integer paise)

**Key features to build:**
- Schema creation with migrations (track version number)
- CRUD for books and customers
- Order creation with stock validation (can't order more than what's in stock)
- Order history with JOIN across orders, order_items, books, and customers
- Stock report showing low-stock items
- Search by title or author name
- Revenue report (total sales by genre, by month)

**Estimated time:** 3-4 hours for a vibe coder using AI tools.

## Go Deeper (Optional)

- **Learn PostgreSQL-specific features** when you outgrow SQLite: JSONB columns for semi-structured data, array types, full-text search with tsvector, and row-level security for multi-tenant apps.
- **Learn query optimization with EXPLAIN ANALYZE** when you have slow queries in production. Understanding query plans (sequential scan vs index scan vs bitmap scan) lets you fix performance issues with surgical precision instead of guessing.
- **Learn database transactions and isolation levels** when you need to handle concurrent writes correctly. The default isolation level works for most apps, but if two users can edit the same record simultaneously, you need to understand read committed vs serializable.
- **Learn connection pooling** when your app has more than 20 concurrent users. Database connections are expensive. Tools like PgBouncer or built-in pool settings prevent "too many connections" errors.
- **Learn database replication** when you need high availability or need to separate read and write traffic. A read replica handles reporting queries without slowing down your main database.
