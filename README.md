pg\_wait\_sampling – sampling based statistics of wait events 
=============================================================

Introduction
------------

PostgreSQL 9.6+ provides an information about current wait event of particular
process.  However, in order to gather descriptive statistics of server
behavior user have to sample current wait event multiple times.
pg\_wait\_sampling is an extension for collecting sampling statistics of wait
events.  It collects two kinds of statistics.

 * History of waits events.  It's implemented as in-memory ring buffer where
   samples of each process wait events are written with given (configurable)
   period.  Therefore, for each running process user can see some number of
   recent samples depending on history size (configurable).  Assuming there is
   a client who periodically read this history and dump it somewhere, user
   can have continuous history. 
 * Waits profile.  It's implemented as in-memory hash table where count
   of samples are accumulated per each process and each wait event.  This hash
   table can be reset by user request.  Assuming there is a client who
   periodically dumps profile and resets it, user can have statistics of
   intensivity of wait events among time. 

pg\_wait\_sampling launches special background worker for gathering the
statistics above.

Authors
-------

 * Alexander Korotkov <a.korotkov@postgrespro.ru>, Postgres Professional,
   Moscow, Russia

Availability
------------

pg\_wait\_sampling is realized as an extension and not available in default
PostgreSQL installation. It is available from
[github](https://github.com/postgrespro/pg_wait_sampling)
under the same license as
[PostgreSQL](http://www.postgresql.org/about/licence/)
and supports PostgreSQL 9.6+.

Installation
------------

pg\_wait\_sampling is PostgreSQL extension which requires PostgreSQL 9.6 or
higher.  Before build and install you should ensure following:
    
 * PostgreSQL version is 9.6 or higher.
 * You have development package of PostgreSQL installed or you built
   PostgreSQL from source.
 * Your PATH variable is configured so that pg\_config command available, or
   set PG_CONFIG variable.

Typical installation procedure may look like this:

    $ git clone https://github.com/postgrespro/pg_wait_sampling.git
    $ cd pg_wait_sampling
    $ make USE_PGXS=1
    $ sudo make USE_PGXS=1 install
    $ make USE_PGXS=1 installcheck
    $ psql DB -c "CREATE EXTENSION pg_wait_sampling;"

Usage
-----

pg\_wait\_sampling interacts with user by set of views and functions.

pg\_wait\_sampling\_current view – information about current wait events for
all processed including background workers.

| Column name | Column type |      Description        |
| ----------- | ----------- | ----------------------- |
| pid         | int4        | Id of process           |
| event_type  | text        | Name of wait event type |
| event       | text        | Name of wait event      |

pg_wait_sampling_get_current(pid int4) returns the same table for single given
process.

pg\_wait\_sampling\_history view – history of wait events obtained by sampling into
in-memory ring buffer.

| Column name | Column type |      Description        |
| ----------- | ----------- | ----------------------- |
| pid         | int4        | Id of process           |
| ts          | timestamptz | Sample timestamp        |
| event_type  | text        | Name of wait event type |
| event       | text        | Name of wait event      |

pg\_wait\_sampling\_profile view – profile of wait events obtained by sampling into
in-memory hash table.

| Column name | Column type |      Description        |
| ----------- | ----------- | ----------------------- |
| pid         | int4        | Id of process           |
| event_type  | text        | Name of wait event type |
| event       | text        | Name of wait event      |
| count       | text        | Count of samples        |

pg_wait_sampling_reset_profile() function resets the profile.

The work of wait event statistics collector worker is controlled by following
GUCs.

|         Parameter name          | Data type |                  Description                | Default value |
| ------------------------------- | --------- | ------------------------------------------- | ------------: |
| pg_wait_sampling.history_size   | int4      | Size of history in-memory ring buffer       |          5000 |
| pg_wait_sampling.history_period | int4      | Period for history sampling in milliseconds |            10 |
| pg_wait_sampling.profile_period | int4      | Period for profile sampling in milliseconds |            10 |

These GUCs are allowed to be changed by superuser. Also, they are placed into
shared memory. Thus, they could be changed from any backend and affects worker
runtime.

See
[PostgreSQL documentation](http://www.postgresql.org/docs/devel/static/monitoring-stats.html#WAIT-EVENT-TABLE)
for list of possible wait events.

Contribution
------------

Please, notice, that pg\_wait\_sampling is still under development and while
it's stable and tested, it may contains some bugs. Don't hesitate to raise
[issues at github](https://github.com/postgrespro/pg_wait_sampling/issues) with
your bug reports.

If you're lacking of some functionality in pg\_wait\_sampling and feeling power
to implement it then you're welcome to make pull requests.

