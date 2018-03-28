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


```bash
git clone https://github.com/locationtech/geomesa.git
cd geomesa
# Build geomesa-hbase from source code
mvn clean install -pl geomesa-hbase/geomesa-hbase-dist -am
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


**Issues reference links:**

https://dev.locationtech.org/mhonarc/lists/geomesa-users/msg02028.html

https://viztales.wordpress.com/my-geomesa-experience/

**Install GeoServer**

```bash
wget http://sourceforge.net/projects/geoserver/files/GeoServer/2.12.2/geoserver-2.12.2-bin.zip
unzip geoserver-2.12.2-bin.zip
mv geoserver-2.12.2 /usr/share
cd /usr/share
ln -s geoserver-2.12.2 geoserver
echo "export GEOSERVER_HOME=/usr/share/geoserver" >> /etc/bashrc
source /etc/bashrc
# Check if GeoServer can be started successfully
cd geoserver/bin
sh startup.sh
```
open the GeoServer from post 8080/geoserver like below
http://pchalla2.field.hortonworks.com:8080/geoserver

**Installing GeoMesa HBase in GeoServer**

The following JARs should be copied from the lib directory of your HBase or Hadoop installations into GeoServer’s WEB-INF/lib
 
```bash
cd /usr/share/geoserver/webapps/geoserver/WEB-INF/lib/
cp /opt/geomesa-hbase/dist/gs-plugins/geomesa-hbase-gs-plugin_2.11-2.0.0-SNAPSHOT-install.tar.gz .
tar -xvf geomesa-hbase-gs-plugin_2.11-2.0.0-SNAPSHOT-install.tar.gz
rm -f geomesa-hbase-gs-plugin_2.11-2.0.0-SNAPSHOT-install.tar.gz
cp /usr/hdp/2.6.4.0-91/hadoop//hadoop-annotations-2.7.3.2.6.4.0-91.jar .
cp /usr/hdp/2.6.4.0-91/hadoop//hadoop-auth-2.7.3.2.6.4.0-91.jar .
cp /usr/hdp/2.6.4.0-91/hadoop//hadoop-common-2.7.3.2.6.4.0-91.jar .
cp /usr/hdp/2.6.4.0-91/hadoop/lib/protobuf-java-2.5.0.jar .
cp /usr/hdp/2.6.4.0-91/hadoop/lib/commons-io-2.4.jar .
cp /usr/hdp/2.6.4.0-91/hbase/lib/hbase-server-1.1.2.2.6.4.0-91.jar .
cp /usr/hdp/2.6.4.0-91/hadoop/lib/zookeeper-3.4.6.2.6.4.0-91.jar .
cp /usr/hdp/2.6.4.0-91/hadoop/lib/commons-configuration-1.6.jar .
ln -s /usr/hdp/current/hbase-client/conf/hbase-site.xml ../classes/hbase-site.xml
```

*Stop and Start the GeoServer to reflect the GeoMesa HBase plugin*

Register the GeoMesa Store with GeoServer
Log into GeoServer using your user and password credentials. The default administration credentials are: User name: admin; Password: geoserver 

Click “Stores” and “Add new Store”. Select the HBase (GeoMesa) vector data source, and fill in the required parameters.

Basic store info:

workspace this is dependent upon your GeoServer installation
data source name pick a sensible name, such as geomesa_quick_start
description this is strictly decorative; GeoMesa quick start
Connection parameters:

these are the same parameter values that you supplied on the command line when you ran the tutorial; they describe how to connect to the HBase instance where your data reside
Click “Save”, and GeoServer will search your HBase table for any GeoMesa-managed feature types.


**Publish the Layer**
GeoServer should recognize the gdelt-quickstart feature type, and should present that as a layer that can be published. Click on the “Publish” link.

You will be taken to the “Edit Layer” screen. You will need to enter values for the data bounding boxes. In this case, you can click on the link to compute these values from the data.
*Bounding Boxes can be updated by clicking on "compute from data" or "compute from native bounds"*

Click on the “Save” button when you are done.

**Take a Look**
Click on the “Layer Preview” link in the left-hand gutter. If you don’t see the quick-start layer on the first page of results, enter the name of the layer you just created into the search box, and press <Enter>.

Once you see your layer, click on the “OpenLayers” link, which will open a new tab. You should see a collection of red dots similar to the following image:

![alt text](https://github.com/mcapavan/geomesa-hbase/images/map.png)


refer http://www.geomesa.org/documentation/tutorials/geomesa-quickstart-hbase.html 

ref:

http://docs.geoserver.org/2.12.x/en/user/installation/linux.html
https://github.com/geoserver/geoserver/tree/2.12.2
