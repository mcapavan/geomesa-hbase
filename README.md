# geomesa-hbase
Use GeoMesa-Hbase to ingest and query geospatial dataset

Ref: http://www.geomesa.org/documentation/user/hbase/install.html

Download source code

*Ensure git, mvn installed and JAVA_HOME is configured*


```
$ git clone https://github.com/locationtech/geomesa.git$ 
$ cd geomesa
$ mvn clean install -DskipTests
$ cd geomesa/geomesa-hbase/target/
$ cp geomesa-hbase_2.11-2.0.0-SNAPSHOT-bin.tar.gz ~/
$ cd ~/
$ tar -xvf geomesa-hbase_2.11-2.0.0-SNAPSHOT-bin.tar.gz
$ cp geomesa-hbase_2.11-2.0.0-SNAPSHOT /opt/
$ ln -s /opt/geomesa-hbase_2.11-2.0.0-SNAPSHOT /opt/geomesa-hbase

```

Configure the environment to use an HDP install

Add the below into /etc/profile

```text
export HADOOP_HOME=/usr/hdp/current/hadoop-client/
export HBASE_HOME=/usr/hdp/current/hbase-client/
export GEOMESA_HBASE_HOME=/opt/geomesa-hbase
export PATH="${PATH}:${GEOMESA_HBASE_HOME}/bin"
```

Due to licensing restrictions, dependencies for shape file support must be separately installed. Do this with the following commands:

```bash
$ cd /opt/geomesa-hbase
$ bin/install-jai.sh
$ bin/install-jline.sh
```

**Deploying the GeoMesa HBase Distributed Runtime JAR**

Setting the hbase.dynamic.jars.dir property in hbase-site.xml as custom property. 

Make sure /apps/hbase/lib is already created in HDFS and change owner to hbase:hdfs

```bash
su hdfs
hdfs dfs -mkdir /apps/hbase/lib
hdfs dfs -chown hbase:hdfs /apps/hbase/lib
hdfs dfs -chmod 755 /apps/hbase/lib
```

Add proprety in Hbase-site.xml via Ambari

```text
hbase.dynamic.jars.dir=/apps/hbase/lib
geomesa.hbase.remote.filtering=false

```

Copy 

```bash
su hbase
hadoop fs -put ${GEOMESA_HBASE_HOME}/dist/hbase/geomesa-hbase-distributed-runtime_2.11-2.0.0-SNAPSHOT.jar /apps/hbase/lib
```

**Register the Coprocessors**

 Register the coprocessors is to specify the coprocessors in the hbase-site.xml
 
 ```text
hbase.coprocessor.user.region.classes=org.locationtech.geomesa.hbase.coprocessor.GeoMesaCoprocessor
```

Due to licensing restrictions, certain dependencies for shape file support and faster XML parsing must be separately installed. Do this with the following commands:

```bash
$ bin/install-jai.sh
$ bin/install-jline.sh
$ bin/install-saxon.sh
```

Test the command that invokes the GeoMesa Tools:

```bash
$ bin/geomesa-hbase
INFO  Usage: geomesa-hbase [command] [command options]
  Commands:
  ...
```

```text
export JAVA_TOOL_OPTIONS=-Dgeomesa.hbase.coprocessor.path=hdfs://pchalla0.field.hortonworks.com:8020/apps/hbase/lib/geomesa-hbase-distributed-runtime_2.11-2.0.0-SNAPSHOT.jar
```