# [Hive Documentation](https://cwiki.apache.org/confluence/display/Hive)

## [General Information about Hive](https://cwiki.apache.org/confluence/display/Hive#Home-GeneralInformationaboutHive)

### [Getting Started](https://cwiki.apache.org/confluence/display/Hive/GettingStarted)

#### [Installation and Configuration](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-InstallationandConfiguration)

##### [Requirements](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-Requirements)

```
cd ~/soft/hadoop/hadoop_2_10_0
export HADOOP_HOME=`pwd`
export PATH=$HADOOP_HOME/bin:$PATH
export PATH=$HADOOP_HOME/sbin:$PATH
```

```
vi ~/.bashrc
```

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_HOME=/home/alvin/soft/hadoop/hadoop_2_10_0
export PATH=$HADOOP_HOME/bin:$PATH
export PATH=$HADOOP_HOME/sbin:$PATH
```

```
source ~/.bashrc
```

##### [Installing Hive from a Stable Release](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-InstallingHivefromaStableRelease)

```
mkdir -p ~/soft/hive/
tar -zxvf apache-hive-2.3.7-bin.tar.gz -C ~/soft/hive/
cd ~/soft/hive/apache-hive-2.3.7-bin/
./bin/start-hbase.sh
./bin/hbase shell
netstat -t4npl
jps
./bin/stop-hbase.sh

export HIVE_HOME=`pwd`
export PATH=$HIVE_HOME/bin:$PATH
```

```
vi ~/.bashrc
```

```
export HIVE_HOME=/home/alvin/soft/hive/apache-hive-2.3.7-bin
export PATH=$HIVE_HOME/bin:$PATH
```

```
source ~/.bashrc
```

##### [Running Hive](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-RunningHive)

```
hadoop fs -mkdir /tmp
hadoop fs -mkdir -p /user/hive/warehouse
hadoop fs -chmod g+w /tmp
hadoop fs -chmod g+w /user/hive/warehouse
```

###### [Running Hive CLI](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-RunningHiveCLI)

```
$HIVE_HOME/bin/hive
```

###### [Running HiveServer2 and Beeline](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-RunningHiveServer2andBeeline.1)

Starting from Hive 2.1, we need to run the schematool command below as an initialization step.

```
cd $HIVE_HOME
schematool -dbType derby -initSchema
#schematool -dbType mysql -initSchema
ls -l|grep metastore_db
```

[HiveServer2](https://cwiki.apache.org/confluence/display/Hive/Setting+Up+HiveServer2) (introduced in Hive 0.11) has its own CLI called [Beeline](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients).  HiveCLI is now **deprecated** in favor of Beeline, as it lacks the multi-user, security, and other capabilities of HiveServer2.  To run HiveServer2 and Beeline from shell:

```
$HIVE_HOME/bin/hiveserver2
$HIVE_HOME/bin/beeline -u jdbc:hive2://localhost:10000
```

Or to start Beeline and HiveServer2 in the same process for testing purpose, for a similar user experience to HiveCLI:

```
$HIVE_HOME/bin/beeline -u jdbc:hive2://
```

##### [Configuration Management Overview](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-ConfigurationManagementOverview)

```
cd $HIVE_HOME/conf
cp hive-default.xml.template hive-site.xml
vi hive-site.xml
```

```
  <property>
    <name>system:java.io.tmpdir</name>
    <value>/tmp/java</value>
  </property>
  <property>
    <name>system:user.name</name>
    <value>${user.name}</value>
  </property>
```

Hive configuration can be manipulated by:

```
vi hive-site.xml
SET mapred.job.tracker=myhost.mycompany.com:50030;
bin/hive --hiveconf x1=y1 --hiveconf x2=y
bin/hiveserver2 --hiveconf x1=y1 --hiveconf x2=y2
bin/beeline --hiveconf x1=y1 --hiveconf x2=y2
export HIVE_OPTS="--hiveconf x1=y1 --hiveconf x2=y2"
```

##### [Runtime Configuration](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-RuntimeConfiguration)

```
SET mapred.job.tracker=myhost.mycompany.com:50030;
SET -v;
```

#### [DDL Operations](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-DDLOperations)

##### [Creating Hive Tables](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-CreatingHiveTables)

```
CREATE TABLE pokes (foo INT, bar STRING);
CREATE TABLE invites (foo INT, bar STRING) PARTITIONED BY (ds STRING);
```

##### [Browsing through Tables](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-BrowsingthroughTables)

```
SHOW TABLES;
SHOW TABLES '.*s';
SHOW CREATE TABLE pokes;
SHOW CREATE TABLE invites;
DESCRIBE invites;
DESC invites;
```

##### [Altering and Dropping Tables](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-AlteringandDroppingTables)

```
ALTER TABLE events RENAME TO 3koobecaf;
ALTER TABLE pokes ADD COLUMNS (new_col INT);
ALTER TABLE invites ADD COLUMNS (new_col2 INT COMMENT 'a comment');
ALTER TABLE invites REPLACE COLUMNS (foo INT, bar STRING, baz INT COMMENT 'baz replaces new_col2');
```

##### [Metadata Store](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-MetadataStore)

Metadata is in an embedded Derby database whose disk storage location is determined by the Hive configuration variable named `javax.jdo.option.ConnectionURL`. 

**By default this location is** `./metastore_db`

```
  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:derby:;databaseName=metastore_db;create=true</value>
    <description>
      JDBC connect string for a JDBC metastore.
      To use SSL to encrypt/authenticate the connection, provide database-specific SSL flag in the connection URL.
      For example, jdbc:postgresql://myhost/db?ssl=true for postgres database.
    </description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>org.apache.derby.jdbc.EmbeddedDriver</value>
    <description>Driver class name for a JDBC metastore</description>
  </property>
```

#### [DML Operations](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-DMLOperations)

The Hive DML operations are documented in [Hive Data Manipulation Language](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML).

```
LOAD DATA LOCAL INPATH './examples/files/kv1.txt' OVERWRITE INTO TABLE pokes;
LOAD DATA LOCAL INPATH './examples/files/kv2.txt' OVERWRITE INTO TABLE invites PARTITION (ds='2008-08-15');
LOAD DATA LOCAL INPATH './examples/files/kv3.txt' OVERWRITE INTO TABLE invites PARTITION (ds='2008-08-08');
```

```
hdfs dfs -ls /user/hive/warehouse/pokes
hdfs dfs -ls /user/hive/warehouse/invites
```

#### [SQL Operations](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-SQLOperations)

The Hive query operations are documented in [Select](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Select).

##### [Example Queries](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-ExampleQueries)

###### [SELECTS and FILTERS](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-SELECTSandFILTERS)

```
SELECT a.foo FROM invites a WHERE a.ds='2008-08-15';
INSERT OVERWRITE DIRECTORY '/tmp/hdfs_out' SELECT a.* FROM invites a WHERE a.ds='2008-08-15';
INSERT OVERWRITE LOCAL DIRECTORY '/tmp/local_out' SELECT a.* FROM pokes a;
INSERT OVERWRITE TABLE events SELECT a.* FROM profiles a;
INSERT OVERWRITE TABLE events SELECT a.* FROM profiles a WHERE a.key < 100;
INSERT OVERWRITE LOCAL DIRECTORY '/tmp/reg_3' SELECT a.* FROM events a;
INSERT OVERWRITE DIRECTORY '/tmp/reg_4' select a.invites, a.pokes FROM profiles a;
INSERT OVERWRITE DIRECTORY '/tmp/reg_5' SELECT COUNT(*) FROM invites a WHERE a.ds='2008-08-15';
INSERT OVERWRITE DIRECTORY '/tmp/reg_5' SELECT a.foo, a.bar FROM invites a;
INSERT OVERWRITE LOCAL DIRECTORY '/tmp/sum' SELECT SUM(a.pc) FROM pc1 a;
```

###### [GROUP BY](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-GROUPBY)

```
FROM invites a INSERT OVERWRITE TABLE events SELECT a.bar, count(*) WHERE a.foo > 0 GROUP BY a.bar;
INSERT OVERWRITE TABLE events SELECT a.bar, count(*) FROM invites a WHERE a.foo > 0 GROUP BY a.bar;
```

###### [JOIN](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-JOIN)

```
FROM pokes t1 JOIN invites t2 ON (t1.bar = t2.bar) INSERT OVERWRITE TABLE events SELECT t1.bar, t1.foo, t2.foo;
```

###### [MULTITABLE INSERT](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-MULTITABLEINSERT)

```
FROM src
INSERT OVERWRITE TABLE dest1 SELECT src.* WHERE src.key < 100
INSERT OVERWRITE TABLE dest2 SELECT src.key, src.value WHERE src.key >= 100 and src.key < 200
INSERT OVERWRITE TABLE dest3 PARTITION(ds='2008-04-08', hr='12') SELECT src.key WHERE src.key >= 200 and src.key < 300
INSERT OVERWRITE LOCAL DIRECTORY '/tmp/dest4.out' SELECT src.value WHERE src.key >= 300;
```

###### [STREAMING](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-STREAMING)

```
FROM invites a INSERT OVERWRITE TABLE events SELECT TRANSFORM(a.foo, a.bar) AS (oof, rab) USING '/bin/cat' WHERE a.ds > '2008-08-09';
```

This streams the data in the map phase through the script `/bin/cat` (like Hadoop streaming).
Similarly – streaming can be used on the reduce side (please see the [Hive Tutorial](https://cwiki.apache.org/confluence/display/Hive/Tutorial#Tutorial-Custommap%2Freducescripts) for examples).

#### [Simple Example Use Cases](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-SimpleExampleUseCases)

##### [MovieLens User Ratings](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-MovieLensUserRatings)

```
CREATE TABLE u_data (
  userid INT,
  movieid INT,
  rating INT,
  unixtime STRING)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE;
LOAD DATA LOCAL INPATH '<path>/u.data'
OVERWRITE INTO TABLE u_data;
SELECT COUNT(*) FROM u_data;
```

##### [Apache Weblog Data](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-ApacheWeblogData)

```
CREATE TABLE apachelog (
  host STRING,
  identity STRING,
  user STRING,
  time STRING,
  request STRING,
  status STRING,
  size STRING,
  referer STRING,
  agent STRING)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'
WITH SERDEPROPERTIES (
  "input.regex" = "([^]*) ([^]*) ([^]*) (-|\\[^\\]*\\]) ([^ \"]*|\"[^\"]*\") (-|[0-9]*) (-|[0-9]*)(?: ([^ \"]*|\".*\") ([^ \"]*|\".*\"))?"
)
STORED AS TEXTFILE;
```

## [User Documentation](https://cwiki.apache.org/confluence/display/Hive#Home-UserDocumentation)

## [Administrator Documentation](https://cwiki.apache.org/confluence/display/Hive#Home-AdministratorDocumentation)



