# geomesa-hbase
Use GeoMesa-Hbase to ingest and query geospatial dataset

Ref: http://www.geomesa.org/documentation/user/hbase/install.html

Download source code

*Ensure git, mvn installed and JAVA_HOME is configured below commands can be used*

```bash
yum install git -y

# configure JAVA_HOME
sed -i '$ a export JAVA_HOME=/usr/jdk64/jdk1.8.0_112' /etc/bashrc
sed -i '$ a export PATH=$PATH:$JAVA_HOME/bin' /etc/bashrc
source /etc/bashrc
java -version

#install Maven:
cd /tmp
wget http://mirror.jax.hugeserver.com/apache/maven/maven-3/3.5.3/binaries/apache-maven-3.5.3-bin.tar.gz
tar -zxvf apache-maven-3.5.3-bin.tar.gz

sudo mv /tmp/apache-maven-3.5.3 /opt
sudo chown -R root:root /opt/apache-maven-3.5.3
sudo ln -s /opt/apache-maven-3.5.3 /opt/apache-maven
echo 'export PATH=$PATH:/opt/apache-maven/bin' | sudo tee -a /etc/bashrc
source /etc/bashrc
mvn --version

``` 


```
git clone https://github.com/locationtech/geomesa.git
cd geomesa
# Build geomesa-hbase from source code
mvn clean install -pl geomesa-hbase -am -DskipTests
cd /tmp/geomesa/geomesa-hbase/geomesa-hbase-dist/target
cp geomesa-hbase_2.11-2.0.0-SNAPSHOT-bin.tar.gz ~/
cd ~/
tar -xvf geomesa-hbase_2.11-2.0.0-SNAPSHOT-bin.tar.gz
cp -R geomesa-hbase_2.11-2.0.0-SNAPSHOT /opt
ln -s /opt/geomesa-hbase_2.11-2.0.0-SNAPSHOT /opt/geomesa-hbase

```

Configure the environment to use an HDP install

Add the below into /etc/profile

```text
sed -i '$ a export HADOOP_HOME=/usr/hdp/current/hadoop-client/' /etc/bashrc
sed -i '$ a export HBASE_HOME=/usr/hdp/current/hbase-client/' /etc/bashrc
sed -i '$ a export GEOMESA_HBASE_HOME=/opt/geomesa-hbase' /etc/bashrc
sed -i '$ a export PATH=$PATH:$GEOMESA_HBASE_HOME/bin' /etc/bashrc
source /etc/bashrc
```

Due to licensing restrictions, dependencies for shape file support must be separately installed. Do this with the following commands:

```bash
cd /opt/geomesa-hbase
bin/install-jai.sh
bin/install-jline.sh
bin/install-saxon.sh
```

**Deploying the GeoMesa HBase Distributed Runtime JAR**

Setting the hbase.dynamic.jars.dir property in hbase-site.xml as custom property. 

Make sure /apps/hbase/lib is already created in HDFS and change owner to hbase:hdfs

Copy GeoMesa-Hbase jar to HBase all Nodes

```bash
cp ${GEOMESA_HBASE_HOME}/dist/hbase/geomesa-hbase-distributed-runtime_2.11-2.0.0-SNAPSHOT.jar /usr/hdp/current/hbase-master/lib
chmod 777 /usr/hdp/current/hbase-master/lib/geomesa-hbase-distributed-runtime_2.11-2.0.0-SNAPSHOT.jar
```

Add proprety in Hbase-site.xml via Ambari

```text
geomesa.hbase.remote.filtering=false
hbase.coprocessor.user.region.classes=org.locationtech.geomesa.hbase.coprocessor.GeoMesaCoprocessor
```

Modify zookeeper.znode.parent value in HBase-Site.xml (org: /hbase-unsecure --> New: /hbase)

```text
zookeeper.znode.parent=/hbase

```
*Note: Make sure the HBase services are restarted *

Test the command that invokes the GeoMesa Tools:

```bash
$ bin/geomesa-hbase
INFO  Usage: geomesa-hbase [command] [command options]
  Commands:
  ...
```

**Download and Build the Tutorial**

```bash
cd ~/
git clone https://github.com/geomesa/geomesa-tutorials.git
cd geomesa-tutorials
mvn clean install -pl geomesa-tutorials-hbase/geomesa-tutorials-hbase-quickstart -am

```
**Running the Tutorial**

```bash
java -cp geomesa-tutorials-hbase/geomesa-tutorials-hbase-quickstart/target/geomesa-tutorials-hbase-quickstart-$VERSION.jar \
    org.geomesa.example.hbase.HBaseQuickStart \
    --hbase.zookeepers <zookeepers>           \
    --hbase.catalog <table>
# example:

java -cp geomesa-tutorials-hbase/geomesa-tutorials-hbase-quickstart/target/geomesa-tutorials-hbase-quickstart-2.0.0-SNAPSHOT.jar \
    org.geomesa.example.hbase.HBaseQuickStart \
    --hbase.zookeepers pchalla2.field.hortonworks.com:2181,pchalla0.field.hortonworks.com:2181,pchalla1.field.hortonworks.com:2181 \
    --hbase.catalog qs1
    
```

**Register the Coprocessors**

```text
export JAVA_TOOL_OPTIONS=-Dgeomesa.hbase.coprocessor.path=hdfs://pchalla0.field.hortonworks.com:8020/apps/hbase/lib/geomesa-hbase-distributed-runtime_2.11-2.0.0-SNAPSHOT.jar
```



**Issues reference links:**

https://dev.locationtech.org/mhonarc/lists/geomesa-users/msg02028.html

https://viztales.wordpress.com/my-geomesa-experience/
