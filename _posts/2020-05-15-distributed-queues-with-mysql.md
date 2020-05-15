---
title: Distributed queues with MySQL 8
date: 2020-05-15 02:00:00
lastmod: 2020-05-15 02:00:00
description: >-
  Writing a distributed queue doesn't have to involve specialist software or writing reams of code. This post covers using MySQL to create a scalable distributed queue.
layout: post
---

Writing a distributed queue doesn't have involve specialist software or writing reams of code. This post covers using MySQL to create a scalable distributed queue. All you will need is MySQL and two terminals.
{: .article-headline}

## Setup

First of all, we need to create a table for job queue and add some jobs:

```mysql
CREATE TABLE job (
  id INT(11) unsigned auto_increment PRIMARY KEY,
  state ENUM("NEW", "DONE")
);
INSERT INTO job VALUES
  (null, "NEW"),
  (null, "NEW"),
  (null, "NEW"),
  (null, "NEW"),
  (null, "NEW");
```

## Locking

In a distributed system with multiple instances of your application there is always the risk that two instances will attempt to take the same jobs from the queue. Luckily, almost all relational databases come with some form of locking built in. In MySQL's case we can use `SELECT ... FOR UPDATE`

Let's simulate multiple instances of our application trying to pick up the jobs at the same time using multiple terminals.

Terminal 1:

```mysql
SELECT * FROM job WHERE state = "NEW" LIMIT 1;
+----+-------+
| id | state |
+----+-------+
|  1 | NEW   |
+----+-------+
```

Terminal 2:

```mysql
SELECT * FROM job WHERE state = "NEW" LIMIT 1;
+----+-------+
| id | state |
+----+-------+
|  1 | NEW   |
+----+-------+
```

As you would expect both instances have now started processing the same job. Not what we want.

However, using `SELECT ... FOR UPDATE` inside a transaction will lock the table until it's been updated by the instance that locked it.

Terminal 1:

```mysql
START TRANSACTION;
SELECT * FROM job WHERE state = "NEW" LIMIT 1 FOR UPDATE;
+----+-------+
| id | state |
+----+-------+
|  1 | NEW   |
+----+-------+
```

How switch over to Terminal 2 and try the same thing:
```mysql
START TRANSACTION;
SELECT * FROM job WHERE state = "NEW" LIMIT 1 FOR UPDATE;
```

At this moment Terminal 2 is locked until Terminal 1 commits the transaction and the query will not return a result.

Terminal 1:

```mysql
UPDATE job SET state = "DONE" WHERE id = 1;
COMMIT;
```

Now Terminal 2 has been unlocked and returns:

```mysql
+----+-------+
| id | state |
+----+-------+
|  2 | NEW   |
+----+-------+
1 row in set (37.304 sec)
```

Not only did it wait for the lock to be free but it also waited for the update to be applied and selected the next job in the list.

## Scaling

You may have noticed that this approach locked the whole table, meaning that only one instance can process jobs at a time. This is also not what we want.

Luckily, the second instance can ask MySQL to skip over any locked rows when it tries to select a job.

Lock a row in Terminal 1:

```mysql
START TRANSACTION;
SELECT * FROM job WHERE state = "NEW" LIMIT 1 FOR UPDATE SKIP LOCKED;
+----+-------+
| id | state |
+----+-------+
|  2 | NEW   |
+----+-------+
```

And switch over to Terminal 2:

```mysql
START TRANSACTION;
SELECT * FROM job WHERE state = "NEW" LIMIT 1 FOR UPDATE SKIP LOCKED;
+----+-------+
| id | state |
+----+-------+
|  3 | NEW   |
+----+-------+
```

And now our second instance selects the row that is not locked by the first instance. It's now safe for each instance to start polling for jobs without fear of stepping on each others toes. You can add as many instances as you want to start processing jobs to horizontally scale your service.

Postgres 9.5 and MySQL 8.0 both have support for the `SKIP LOCKED` feature, unfortunately MariaDB does not currently support it.

There's no doubt that queues can grow more complex with features like TTL and dead letter queues, but if you want something simple and don't need all the bells and whistles of something like RabbitMQ then this approach should prove simple and scalable.  
