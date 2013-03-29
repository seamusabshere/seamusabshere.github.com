---
layout: post
title: "Hinting Postgres and MySQL with OFFSET and LIMIT"
description: "Prevent SQL inlining in Postgres 9.x; make sure temporary tables are created in memory in MySQL 5.5+"
category: 
tags: [postgres, mysql, database, sql]
---
{% include JB/setup %}

If your database is behaving irrationally, try white diamonds.

<p><a href="https://www.youtube.com/watch?v=vjVfu8-Wp6s" title="Elizabeth Taylor White Diamonds"><img src="/images/2013-03-29-hinting-postgres-and-mysql-with-offset-and-limit/whitediamonds.png" /></a></p>

"These have always brought me luck". No, JK, but maybe `LIMIT` and `OFFSET` can be used creatively to solve your problem. Here are two examples.

## tl;dr

Tell Postgres not to inline with `OFFSET 0`:

{% highlight postgres %}
SELECT COUNT(*) FROM (
  SELECT * FROM tbl WHERE id IN ('6d48fc431d21', 'd9e659e756ad') OFFSET 0
) AS t WHERE data ? 'building_floorspace'
{% endhighlight %}

In MySQL 5.5, create an in-memory temporary table with `LIMIT 0`:

{% highlight mysql %}
CREATE TEMPORARY TABLE flight_segment_cohort_78990172264
  ENGINE=MEMORY
  AS (SELECT * FROM `flight_segments` LIMIT 0)
{% endhighlight %}

## Prevent SQL inlining in Postgres 9.x with `OFFSET 0`

Sometimes the Postgres query optimizer does silly things like applying a more complex condition before paying attention to primary keys: (4.5 seconds, even though I've explicitly provided the primary keys)

{% highlight postgres %}
EXPLAIN ANALYZE SELECT COUNT(*) FROM tbl WHERE id IN ('6d48fc431d21', 'd9e659e756ad') AND data ? 'building_floorspace' AND data ?| ARRAY['elec_mean_monthly_use', 'gas_mean_monthly_use'];
                                                                     QUERY PLAN                                                                     
----------------------------------------------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=4.09..4.09 rows=1 width=0) (actual time=4457.886..4457.887 rows=1 loops=1)
   ->  Index Scan using idx_tbl_on_data_gist on tbl  (cost=0.00..4.09 rows=1 width=0) (actual time=4457.880..4457.880 rows=0 loops=1)
         Index Cond: ((data ? 'building_floorspace'::text) AND (data ?| '{elec_mean_monthly_use,gas_mean_monthly_use}'::text[]))
         Filter: ((id)::text = ANY ('{6d48fc431d21,d9e659e756ad}'::text[]))
 Total runtime: 4457.948 ms
(5 rows)
{% endhighlight %}{.wide}

If you try a subselect and it doesn't help, the problem is inlining: (still 4.5 seconds because the subselect is inlined and therefore the query is exactly the same as above)

{% highlight postgres %}
EXPLAIN ANALYZE SELECT COUNT(*) FROM (  SELECT * FROM tbl WHERE id IN ('6d48fc431d21', 'd9e659e756ad')  ) AS t WHERE data ? 'building_floorspace' AND data ?| ARRAY['elec_mean_monthly_use', 'gas_mean_monthly_use'];
                                                                     QUERY PLAN                                                                     
----------------------------------------------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=4.09..4.09 rows=1 width=0) (actual time=4854.170..4854.171 rows=1 loops=1)
   ->  Index Scan using idx_tbl_on_data_gist on tbl  (cost=0.00..4.09 rows=1 width=0) (actual time=4854.165..4854.165 rows=0 loops=1)
         Index Cond: ((data ? 'building_floorspace'::text) AND (data ?| '{elec_mean_monthly_use,gas_mean_monthly_use}'::text[]))
         Filter: ((id)::text = ANY ('{6d48fc431d21,d9e659e756ad}'::text[]))
 Total runtime: 4854.220 ms
(5 rows)
{% endhighlight %}

Even though you're not supposed to need hinting, there is a [way to tell Postgres not to inline](http://www.postgresql.org/message-id/E1RfAwz-0006Us-7B@wrigleys.postgresql.org): (much faster - 0.223ms!)

{% highlight postgres %}
EXPLAIN ANALYZE SELECT COUNT(*) FROM (  SELECT * FROM tbl WHERE id IN ('6d48fc431d21', 'd9e659e756ad') OFFSET 0 ) AS t WHERE data ? 'building_floorspace' AND data ?| ARRAY['elec_mean_monthly_use', 'gas_mean_monthly_use'];
                                                                QUERY PLAN                                                                
------------------------------------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=8.14..8.15 rows=1 width=0) (actual time=0.165..0.166 rows=1 loops=1)
   ->  Subquery Scan on t  (cost=4.14..8.14 rows=1 width=0) (actual time=0.160..0.160 rows=0 loops=1)
         Filter: ((t.data ? 'building_floorspace'::text) AND (t.data ?| '{elec_mean_monthly_use,gas_mean_monthly_use}'::text[]))
         ->  Limit  (cost=4.14..8.13 rows=2 width=496) (actual time=0.086..0.092 rows=2 loops=1)
               ->  Bitmap Heap Scan on tbl  (cost=4.14..8.13 rows=2 width=496) (actual time=0.083..0.086 rows=2 loops=1)
                     Recheck Cond: ((id)::text = ANY ('{6d48fc431d21,d9e659e756ad}'::text[]))
                     ->  Bitmap Index Scan on tbl_pkey  (cost=0.00..4.14 rows=2 width=0) (actual time=0.068..0.068 rows=2 loops=1)
                           Index Cond: ((id)::text = ANY ('{6d48fc431d21,d9e659e756ad}'::text[]))
 Total runtime: 0.223 ms
(9 rows)
{% endhighlight %}

The trick is `OFFSET 0` in the subquery.

Background: I have a Postgres 9.1 table using [hstore](http://www.postgresql.org/docs/9.1/static/hstore.html) and with a [GiST index](http://www.postgresql.org/docs/9.1/static/textsearch-indexes.html) on it.

## Make sure temporary tables are created in memory in MySQL 5.5+ with `LIMIT 0`

I upgraded to MySQL 5.5 (from 5.1) and suddenly my server started thrashing like crazy. In the logs:

{% highlight mysql %}
(3278.3ms)   CREATE TEMPORARY TABLE flight_segment_cohort_70219108888 LIKE `flight_segments` 
{% endhighlight %}

It turns out that [in MySQL 5.5+ it's easy to accidentally create InnoDB temp tables](http://stackoverflow.com/questions/5859391/create-temporary-table-from-select-statement-without-using-create-table). What's more, you can't use `ENGINE=MEMORY` with `LIKE`.

{% highlight mysql %}
(3.1ms)   CREATE TEMPORARY TABLE flight_segment_cohort_23148488864 ENGINE=MEMORY AS (SELECT * FROM `flight_segments` LIMIT 0)
{% endhighlight %}

The trick is replacing `LIKE` with `AS (SELECT [...] LIMIT 0)`.

Background: the [flight impact model](http://impact.brighterplanet.com/models/flight) [(source code)](https://github.com/brighterplanet/flight) uses temp tables to perform aggregations over a subset of rows that changes for almost every API call.
