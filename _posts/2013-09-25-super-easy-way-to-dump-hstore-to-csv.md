---
layout: post
title: "[out of date] Super easy way to dump HStore to CSV"
description: "Sometimes you just want to see your HStore table in Excel..."
category: 
tags: [postgres, database, sql, hstore, csv]
---
{% include JB/setup %}

_WARNING: This post is ancient and probably wrong._

As of today, neither [pgAdmin](http://www.pgadmin.org/) nor [PG Commander](http://eggerapps.at/pgcommander/) display [hstore](http://www.postgresql.org/docs/9.1/static/hstore.html) data nicely. Here's a 5-line Ruby script to stick in `~/bin/hscv` that dumps a hstore column to a CSV:

{% highlight ruby %}
#!/usr/bin/env ruby
# Usage: hcsv DBNAME TBLNAME HSTORECOL 
# Output columns will be id + all the hstore keys

dbname, tblname, hstorecol = ARGV[0..2]

# Get hstore keys
out = `psql #{dbname} --tuples --command "SELECT DISTINCT k FROM (SELECT skeys(#{hstorecol}) AS k FROM #{tblname}) AS dt ORDER BY k"`
headers = out.split(/\n/).map(&:strip)

# Dump CSV of id + all hstore keys
hstore_headers_sql = headers.map { |k| %{#{hstorecol}->'#{k}' AS "#{k}"} }.join(', ')
system 'psql', dbname, '--tuples', '--command', "COPY (SELECT id, #{hstore_headers_sql} FROM #{tblname}) TO STDOUT (FORMAT 'csv', HEADER)"
{% endhighlight %}{.wide}

So in your `showoff` database you have a `pets` table with an hstore column called `d`:

{% highlight console %}
$ createdb showoff
$ psql showoff
psql (9.1.9)
Type "help" for help.

showoff=# CREATE EXTENSION hstore;
CREATE EXTENSION
showoff=# CREATE TABLE pets (id SERIAL, d HSTORE);
CREATE TABLE
showoff=# INSERT INTO pets (d) VALUES ('"name"=>"Jerry","breed"=>"beagle","age"=>"6"');
INSERT 0 1
showoff=# INSERT INTO pets (d) VALUES ('"name"=>"Amigo","breed"=>"lizard","age"=>"15"');
INSERT 0 1
showoff=# select * from pets;
 id |                        d                        
----+-------------------------------------------------
  2 | "age"=>"6", "name"=>"Jerry", "breed"=>"beagle"
  3 | "age"=>"15", "name"=>"Amigo", "breed"=>"lizard"
(2 rows)
{% endhighlight %}{.wide}

Then you can do:

{% highlight console %}
$ hcsv showoff pets d
id,age,breed,name
2,6,beagle,Jerry
3,15,lizard,Amigo
{% endhighlight %}{.wide}

