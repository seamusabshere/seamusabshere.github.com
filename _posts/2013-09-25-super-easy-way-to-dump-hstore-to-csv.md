---
layout: post
title: "Super easy way to dump HStore to CSV"
description: "Sometimes you just want to see your HStore table in Excel..."
category: 
tags: [postgres, database, sql, hstore, csv]
---
{% include JB/setup %}

<script src="https://gist.github.com/seamusabshere/6708941.js">
</script>

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

