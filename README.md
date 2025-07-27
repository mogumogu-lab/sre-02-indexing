# Indexing Test

This repository 

## Command

### Restore Database

```bash
pg_restore -U postgres -d dvdrental /data/dvdrental.tar
```

### Send SQL

```bash
pgbench -h postgres-test -p 5432 -U postgres -d dvdrental -c 60 -T 10 -f test/02_simple_select.sql --no-vacuum
```

## Indexes

### 01. No Index

#### Query

```sql
-- Indexes
CREATE INDEX film_fulltext_idx ON public.film USING gist (fulltext);
CREATE INDEX idx_fk_language_id ON public.film USING btree (language_id);
CREATE INDEX idx_title ON public.film USING btree (title);

-- Remove existing index
DROP INDEX IF EXISTS film_fulltext_idx;
DROP INDEX IF EXISTS idx_fk_language_id;
DROP INDEX IF EXISTS idx_title;

-- Query
EXPLAIN (ANALYZE) SELECT * FROM film WHERE title = 'LOVE ACTUALLY';
EXPLAIN (ANALYZE) SELECT * FROM film WHERE title ILIKE '%LOVE%';
```

#### Result

- `WHERE title = 'LOVE ACTUALLY'`
  - Seq Scan on film  (cost=0.00..100.50 rows=1 width=384) (actual time=0.248..0.249 rows=0 loops=1)
  - Filter: ((title)::text = 'LOVE ACTUALLY'::text)
  - Rows Removed by Filter: 1000
  - Planning Time: 0.069 ms
  - Execution Time: 0.265 ms

- `Where title ILIKE '%LOVE%'`
  - Seq Scan on film  (cost=0.00..100.50 rows=1 width=384) (actual time=0.421..1.066 rows=10 loops=1)
  - Filter: ((title)::text ~~* '%LOVE%'::text)
  - Rows Removed by Filter: 990
  - Planning Time: 0.158 ms
  - Execution Time: 1.081 ms

#### Explanation

- Without indexes, even simple queries require a full table scan, which becomes much slower as the table grows.
- For small tables, the performance impact isn't obvious, but for production-sized data (hundreds of thousands or millions of rows), lack of indexes becomes a major bottleneck.
- Especially for pattern-matching queries like LIKE/ILIKE (with a leading %), a B-Tree index can't be used and a full scan is always required.



### 02. B-Tree Index
#### Query

```sql
CREATE INDEX idx_title ON film(title);

-- 쿼리
EXPLAIN (ANALYZE) SELECT * FROM film WHERE title = 'LOVE ACTUALLY';
EXPLAIN (ANALYZE) SELECT * FROM film WHERE title ILIKE 'LOVE%'; -- (앞부분만, 인덱스 적용)
```

#### Explanation




### 03. Composite B-Tree Index
#### Query

```sql
CREATE INDEX idx_title_rate ON film(title, rental_rate);

-- 쿼리
EXPLAIN (ANALYZE) SELECT * FROM film WHERE title = 'LOVE ACTUALLY' AND rental_rate = 4.99;
EXPLAIN (ANALYZE) SELECT * FROM film WHERE rental_rate = 4.99 AND title = 'LOVE ACTUALLY';
```

#### Explanation




### 04. Partial Index
#### Query

```sql
CREATE INDEX idx_active_customer ON customer(last_name) WHERE active = 1;

-- 쿼리
EXPLAIN (ANALYZE) SELECT * FROM customer WHERE last_name = 'SMITH' AND active = 1;
EXPLAIN (ANALYZE) SELECT * FROM customer WHERE last_name = 'SMITH' AND active = 0; -- (인덱스 미적용 비교)

```

#### Explanation




### 05. Expression Index
#### Query

```sql
CREATE INDEX idx_lower_title ON film(LOWER(title));

-- 쿼리
EXPLAIN (ANALYZE) SELECT * FROM film WHERE LOWER(title) = 'love actually';

```

#### Explanation




### 06. Full Text Index
#### Query

```sql
-- 이미 GIST 인덱스(film_fulltext_idx) 있음
EXPLAIN (ANALYZE) SELECT * FROM film WHERE fulltext @@ plainto_tsquery('love');

```

#### Explanation




### 07 Covering Index 
#### Query

```sql
CREATE INDEX idx_title_include_id ON film(title) INCLUDE (film_id, rental_rate);

-- 쿼리
EXPLAIN (ANALYZE) SELECT film_id, rental_rate FROM film WHERE title = 'LOVE ACTUALLY';

```

#### Explanation




### 08. Composite Index with Order
#### Query

```sql
-- (title, rental_rate) vs (rental_rate, title) 비교
CREATE INDEX idx_title_rate2 ON film(title, rental_rate);
CREATE INDEX idx_rate_title2 ON film(rental_rate, title);

-- 쿼리
EXPLAIN (ANALYZE) SELECT * FROM film WHERE title = 'LOVE ACTUALLY' AND rental_rate = 4.99;
EXPLAIN (ANALYZE) SELECT * FROM film WHERE rental_rate = 4.99 AND title = 'LOVE ACTUALLY';

-- 정렬 성능 비교
EXPLAIN (ANALYZE) SELECT * FROM film WHERE rental_rate = 4.99 ORDER BY title;
EXPLAIN (ANALYZE) SELECT * FROM film WHERE title = 'LOVE ACTUALLY' ORDER BY rental_rate;
```

#### Explanation



