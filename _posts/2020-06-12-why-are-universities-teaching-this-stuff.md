---
layout: post
title: "Why are universities teaching this stuff? Stata, Java, no SQL..."
description: "It would be so straightforward for universities to give students better preparation for industry. Why don't they?"
category:
tags: [sql]
---
{% include JB/setup %}

I'm the CTO at [Faraday](https://faraday.io), where we do a lot of hard programming. I've been looking resumes of current and recently graduated students from really good universities like [Middlebury](https://www.middlebury.edu) and also scanning the current syllabus of [COS 333 Advanced Programming Techniques](https://www.cs.princeton.edu/courses/archive/spr20/cos333/topics.html), the advanced programming class at Princeton. Here's what I'm finding.

## Not enough SQL

How do you graduate somebody in a STEM degree without teaching them SQL? It should be an absolute requirement for CS majors, and arguably for many sister disciplines. Here I quote from an [interview with the late great Jim Gray](https://conservancy.umn.edu/bitstream/handle/11299/107339/oh353jg.pdf?sequence=1):

> Frana: So who were the great populizers and the proselytizers for the relational model? Ted? Chris Date? Yourself?

> Gray: [...] What happened is that the academic community found DBTG and IMS pretty complicated [...] And along came the relational model with query optimization, and transactions, and security. The data model was simple enough that you could state it and then start reasoning about it. [...] There began to be academic departments, and those departments started producing students. Some of those students went off to form other departments, and some went to industry. It was a self-reinforcing system.

The reason we have excellent systems like [Google BigQuery](https://cloud.google.com/bigquery) and [Postgres](https://www.postgresql.org/) is because SQL is _theoretically interesting_. OK, maybe all the necessary theorems have been proved, so it can't be an advanced course for the go-getters, but maybe that's a good thing. *Just please graduate people who know SQL.*

## Too much Java

Why are universities still teaching Java? It forms the core of [COS 333 Advanced Programming Techniques](https://www.cs.princeton.edu/courses/archive/spr20/cos333/topics.html). Java teaches you

* no memory management
* little functional programming

As a student, if you want to pick up a hot new language like [Rust](https://www.rust-lang.org/), this is the double whammy - you just won't understand. *Java should be an elective for people who want to work in Java shops.*

How about just follow MIT's example and have students start in Python like [6.0001 Introduction to Computer Science and Programming in Python](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-0001-introduction-to-computer-science-and-programming-in-python-fall-2016/) and then teach Lisp, again just like MIT's [6.009 Fundementals of Programming](https://py.mit.edu/spring20)?

## Why would you ever teach proprietary software like Stata?

Often I see resumes that list [Stata](https://www.stata.com/) or [ESRI GIS products](https://www.esri.com/en-us/home) front and center. [Matlab](https://www.mathworks.com/products/matlab.html) appears a lot too. That's fine, I think government statisticians and city planners and professional engineers really use these proprietary products in their places of work. But that is a terrible reason to start students on a proprietary system when such excellent open alternatives exist and in some cases are superseding their proprietary peers. Some examples:

* Why teach Stata when you could teach [R](https://www.r-project.org/)? I would be very impressed if an intern could talk to me about [tidyverse](https://www.tidyverse.org/).
* Why teach ArcGIS when you could use [Postgis](https://postgis.net/) and [QGIS](https://www.qgis.org/en/site/)? Postgis is deployed across all of the major cloud databases ([AWS RDS Postgres](https://aws.amazon.com/rds/postgresql/), [AWS Aurora Postgres](https://aws.amazon.com/rds/aurora/postgresql-features/), [Azure Postgres](https://azure.microsoft.com/en-us/services/postgresql/), [GCP Cloud SQL Postgres](https://cloud.google.com/sql/), [Azure Hyperscale (formerly Citus Data)](https://docs.microsoft.com/en-us/azure/azure-sql/database/service-tier-hyperscale)) and you can use very similar `ST_` method calls in [Google BigQuery GIS](https://cloud.google.com/bigquery/docs/gis-intro).

Matlab gets a pass because [Octave](https://www.gnu.org/software/octave/) is really just free Matlab... in any work environment, I'm guessing you would get the real thing.

## Conclusion

Universities pick a bizarre set of technologies that don't reflect the modern world, or even respect the academic traditions that brought us wonderful things like SQL. As an employer, I wish I saw more candidates with these three basic qualities:

1. SQL
2. Experience in a language other than Java
3. R / Postgis / choose any one speciality in open software

Thank you! Omg.
