python-mysql-replication
========================

<a href="https://travis-ci.org/noplay/python-mysql-replication"><img src="https://travis-ci.org/noplay/python-mysql-replication.svg?branch=master"></a>&nbsp;
<a href="https://pypi.python.org/pypi/mysql-replication"><img src="http://img.shields.io/pypi/dm/mysql-replication.svg"></a>

Pure Python Implementation of MySQL replication protocol build on top of PyMYSQL. This allow you to receive event like insert, update, delete with their datas and raw SQL queries.

Use cases
===========

* MySQL to NoSQL database replication
* MySQL to search engine replication
* Invalidate cache when something change in database
* Audit
* Real time analytics

Documentation
==============

A work in progress documentation is available here: https://python-mysql-replication.readthedocs.org/en/latest/

Instruction about building documentation is available here:
https://python-mysql-replication.readthedocs.org/en/latest/developement.html


Installation
=============

```
pip install mysql-replication
```

Mailing List
==============

You can get support and discuss about new features on:
https://groups.google.com/d/forum/python-mysql-replication



Project status
================

The project is test with:
* MySQL 5.5, 5.6 and 5.7
* Python >= 2.6
* Python 3.3, 3.4, 3.5 and 3.6 (3.2 is not supported)
* PyPy (really faster than the standard Python interpreter)

The project is used in production for critical stuff in some
medium internet corporations. But all use case as not
been perfectly test in the real world.

Limitations
=============

https://python-mysql-replication.readthedocs.org/en/latest/limitations.html

Projects using this library
===========================

* MySQL River Plugin for ElasticSearch: https://github.com/scharron/elasticsearch-river-mysql
* Ditto: MySQL to MemSQL replicator https://github.com/memsql/ditto
* ElasticMage: Full Magento integration with ElasticSearch https://github.com/ElasticMage/elasticmage
* Cache buster: an automatic cache invalidation system https://github.com/rackerlabs/cache-busters
* Zabbix collector for OpenTSDB https://github.com/OpenTSDB/tcollector/blob/master/collectors/0/zabbix_bridge.py
* Meepo: Event sourcing and event broadcasting for datebases. https://github.com/eleme/meepo
* Python MySQL Replication Blinker: This package read events from MySQL binlog and send to blinker's signal. https://github.com/tarzanjw/python-mysql-replication-blinker
* aiomysql_replication: Fork supporting asyncio https://github.com/jettify/aiomysql_replication
* python-mysql-eventprocessor: Daemon interface for handling MySQL binary log events. https://github.com/jffifa/python-mysql-eventprocessor
* mymongo: MySQL to mongo replication https://github.com/njordr/mymongo
* pg_chameleon: Migration and replica from MySQL to PostgreSQL https://github.com/the4thdoctor/pg_chameleon
* MySQLStreamer: MySQLStreamer is a database change data capture and publish system https://github.com/Yelp/mysql_streamer
* Yelp Data Pipeline: https://engineeringblog.yelp.com/2016/11/open-sourcing-yelps-data-pipeline.html

MySQL server settings
=========================

In your MySQL server configuration file you need to enable replication:

    [mysqld]
    server-id		 = 1
    log_bin			 = /var/log/mysql/mysql-bin.log
    expire_logs_days = 10
    max_binlog_size  = 100M
    binlog-format    = row #Very important if you want to receive write, update and delete row events

Examples
=========

All examples are available in the [examples directory](https://github.com/noplay/python-mysql-replication/tree/master/examples)


This example will dump all replication events to the console:

```python
from pymysqlreplication import BinLogStreamReader

mysql_settings = {'host': '127.0.0.1', 'port': 3306, 'user': 'root', 'passwd': ''}

stream = BinLogStreamReader(connection_settings = mysql_settings, server_id=100)

for binlogevent in stream:
    binlogevent.dump()

stream.close()
```

For this SQL sessions:

```sql
CREATE DATABASE test;
use test;
CREATE TABLE test4 (id int NOT NULL AUTO_INCREMENT, data VARCHAR(255), data2 VARCHAR(255), PRIMARY KEY(id));
INSERT INTO test4 (data,data2) VALUES ("Hello", "World");
UPDATE test4 SET data = "World", data2="Hello" WHERE id = 1;
DELETE FROM test4 WHERE id = 1;
```

Output will be:

    === RotateEvent ===
    Date: 1970-01-01T01:00:00
    Event size: 24
    Read bytes: 0

    === FormatDescriptionEvent ===
    Date: 2012-10-07T15:03:06
    Event size: 84
    Read bytes: 0

    === QueryEvent ===
    Date: 2012-10-07T15:03:16
    Event size: 64
    Read bytes: 64
    Schema: test
    Execution time: 0
    Query: CREATE DATABASE test

    === QueryEvent ===
    Date: 2012-10-07T15:03:16
    Event size: 151
    Read bytes: 151
    Schema: test
    Execution time: 0
    Query: CREATE TABLE test4 (id int NOT NULL AUTO_INCREMENT, data VARCHAR(255), data2 VARCHAR(255), PRIMARY KEY(id))

    === QueryEvent ===
    Date: 2012-10-07T15:03:16
    Event size: 49
    Read bytes: 49
    Schema: test
    Execution time: 0
    Query: BEGIN

    === TableMapEvent ===
    Date: 2012-10-07T15:03:16
    Event size: 31
    Read bytes: 30
    Table id: 781
    Schema: test
    Table: test4
    Columns: 3

    === WriteRowsEvent ===
    Date: 2012-10-07T15:03:16
    Event size: 27
    Read bytes: 10
    Table: test.test4
    Affected columns: 3
    Changed rows: 1
    Values:
    --
    * data : Hello
    * id : 1
    * data2 : World

    === XidEvent ===
    Date: 2012-10-07T15:03:16
    Event size: 8
    Read bytes: 8
    Transaction ID: 14097

    === QueryEvent ===
    Date: 2012-10-07T15:03:17
    Event size: 49
    Read bytes: 49
    Schema: test
    Execution time: 0
    Query: BEGIN

    === TableMapEvent ===
    Date: 2012-10-07T15:03:17
    Event size: 31
    Read bytes: 30
    Table id: 781
    Schema: test
    Table: test4
    Columns: 3

    === UpdateRowsEvent ===
    Date: 2012-10-07T15:03:17
    Event size: 45
    Read bytes: 11
    Table: test.test4
    Affected columns: 3
    Changed rows: 1
    Affected columns: 3
    Values:
    --
    * data : Hello => World
    * id : 1 => 1
    * data2 : World => Hello

    === XidEvent ===
    Date: 2012-10-07T15:03:17
    Event size: 8
    Read bytes: 8
    Transaction ID: 14098

    === QueryEvent ===
    Date: 2012-10-07T15:03:17
    Event size: 49
    Read bytes: 49
    Schema: test
    Execution time: 1
    Query: BEGIN

    === TableMapEvent ===
    Date: 2012-10-07T15:03:17
    Event size: 31
    Read bytes: 30
    Table id: 781
    Schema: test
    Table: test4
    Columns: 3

    === DeleteRowsEvent ===
    Date: 2012-10-07T15:03:17
    Event size: 27
    Read bytes: 10
    Table: test.test4
    Affected columns: 3
    Changed rows: 1
    Values:
    --
    * data : World
    * id : 1
    * data2 : Hello

    === XidEvent ===
    Date: 2012-10-07T15:03:17
    Event size: 8
    Read bytes: 8
    Transaction ID: 14099


Tests
========
When it's possible we have a unit test.

More information is available here:
https://python-mysql-replication.readthedocs.org/en/latest/developement.html

Changelog
==========
https://github.com/noplay/python-mysql-replication/blob/master/CHANGELOG

Similar projects
==================
* Kodoma: Ruby-binlog based MySQL replication listener https://github.com/y310/kodama
* MySQL Hadoop Applier: C++ version http://dev.mysql.com/tech-resources/articles/mysql-hadoop-applier.html
* Java: https://github.com/shyiko/mysql-binlog-connector-java
* GO: https://github.com/siddontang/go-mysql
* PHP: Based on this this project https://github.com/krowinski/php-mysql-replication and https://github.com/fengxiangyun/mysql-replication 

Special thanks
================
* MySQL binlog from Jeremy Cole was a great source of knowledge about MySQL replication protocol https://github.com/jeremycole/mysql_binlog
* Samuel Charron for his help https://github.com/scharron

Contributors
==============

Major contributor:
* Julien Duponchelle Original author https://github.com/noplay
* bjoernhaeuser for his bugs fixing, improvements and community support https://github.com/bjoernhaeuser
* Arthur Gautier gtid, slave report...  https://github.com/baloo

Other contributors:
* Dvir Volk for bug fix https://github.com/dvirsky
* Lior Sion code cleanup and improvements https://github.com/liorsion
* Lx Yu code improvements, primary keys detections https://github.com/lxyu
* Young King for pymysql 0.6 support https://github.com/youngking
* David Reid checksum checking fix https://github.com/dreid
* Alex Gaynor fix smallint24 https://github.com/alex
* lifei NotImplementedEvent https://github.com/lifei
* Maralla Python 3.4 fix https://github.com/maralla
* Daniel Gavrila more MySQL error codes https://github.com/danielduduta
* Bernardo Sulzbach code cleanup https://github.com/mafagafogigante
* Darioush Jalali Python 2.6 backport https://github.com/darioush
* Jasonz bug fixes https://github.com/jasonzzz
* Bartek Ogryczak cleanup and improvements https://github.com/vartec
* Wang, Xiaozhe cleanup https://github.com/chaoslawful
* siddontang improvements https://github.com/siddontang
* Cheng Chen Python 2.6 compatibility https://github.com/cccc1999
* Jffifa utf8mb4 compatibility https://github.com/jffifa
* Romuald Brunet bug fixes https://github.com/romuald
* Cédric Hourcade Don't fail on incomplete dates https://github.com/hc
* Giacomo Lozito Explicit close stream connection on exception https://github.com/giacomolozito
* Giovanni F. MySQL 5.7 support https://github.com/26fe
* Igor Mastak intvar event https://github.com/mastak
* Xie Zhenye fix missing update _next_seq_no https://github.com/xiezhenye
* Abrar Sheikh: Multiple contributions https://github.com/abrarsheikh
* Keegan Parker: secondary database for reference schema https://github.com/kdparker
* Troy J. Farrell Clear table_map if RotateEvent has timestamp of 0 https://github.com/troyjfarrell
* Zhanwei Wang Fail to get table informations https://github.com/wangzw

Thanks to GetResponse for their support

Licence
=======
Copyright 2012-2017 Julien Duponchelle

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.


