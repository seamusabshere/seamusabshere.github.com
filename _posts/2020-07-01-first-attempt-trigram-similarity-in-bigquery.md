---
layout: post
title: "First attempt at trigram similarity function in BigQuery"
description: "pg_trgm is so useful, I wish it was native in BigQuery."
category:
tags: [sql, bigquery]
---
{% include JB/setup %}

Here's my first attempt at trigram matching in [Google BigQuery](https://cloud.google.com/bigquery):

```
CREATE TEMP FUNCTION trigram_similarity(a STRING, b STRING) AS (
  (
    WITH a_trigrams AS (
      SELECT
        DISTINCT tri_a
      FROM
        unnest(ML.NGRAMS(SPLIT(LOWER(a), ''), [3,3])) AS tri_a
    ),
    b_trigrams AS (
      SELECT
        DISTINCT tri_b
      FROM
        unnest(ML.NGRAMS(SPLIT(LOWER(b), ''), [3,3])) AS tri_b
    )
    SELECT
      COUNTIF(tri_b IS NOT NULL) / COUNT(*)
    FROM
      a_trigrams
      LEFT JOIN b_trigrams ON tri_a = tri_b
  )
);
```

I compared it to [Postgres's pg_trgm](https://www.postgresql.org/docs/12/pgtrgm.html) and it got _similar_ results, but I must be missing something:


```
select trigram_similarity('saemus', 'seamus');
-- 0.25 vs. pg_trgm 0.272727

select trigram_similarity('shamus', 'seamus');
-- 0.5 vs. pg_trgm 0.4
```

I'll update this if/when I figure out how to make it match `pg_trgm`, which is the gold standard. I already did that in [Ruby pg_trgm gem](https://github.com/seamusabshere/pg_trgm).