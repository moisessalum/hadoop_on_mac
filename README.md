# Hadoop Ecosystem on Mac

## Introduction
Thank you for your time and interest in this repository.

I'm a Data Engineer and I was interested in testing and installing the Hadoop ecosystem on my personal Mac to play around.
I share this installation steps because I haven't found enough documentation to install Hadoop on Mac.

## Installation

### Prerequisites
PostgreSQL 9.6.15 already installed.

### Homebrew
Most of the Hadoop ecosystem can be directly installed from [Homebrew](https://brew.sh/) so this is where you start.

In your terminal, run:
```bash
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### Java
It's important that you verify which version of Java you have installed on your Mac to avoid compatibility issues.

The [Hadoop Java Versions](https://cwiki.apache.org/confluence/display/HADOOP/Hadoop+Java+Versions) information is helpful to know which Java version you must have installed for Hadoop to work.

We're installing the latest version Hadoop version (3.2.1), so we need Java 8 installed.

To verify which version of Java you have installed run:
```bash
$ /usr/libexec/java_home -V
```

My output shows something like this:
```bash
Matching Java Virtual Machines (4):
    13.0.1, x86_64:     "OpenJDK 13.0.1"        /Library/Java/JavaVirtualMachines/openjdk-13.0.1.jdk/Contents/Home
    11.0.1, x86_64:     "OpenJDK 11.0.1"        /Library/Java/JavaVirtualMachines/openjdk-11.0.1.jdk/Contents/Home
    1.8.0_191, x86_64:  "Java SE 8"     /Library/Java/JavaVirtualMachines/jdk1.8.0_191.jdk/Contents/Home
    1.8.0_172, x86_64:  "Java SE 8"     /Library/Java/JavaVirtualMachines/jdk1.8.0_172.jdk/Contents/Home
```

I will use the 1.8.0_172 Java version on this installation.

If you need to install Java 8 on your computer follow this [link](https://www.oracle.com/technetwork/java/javase/downloads/index.html).

### Hadoop
I've found this very helpful answer by [Arefe](https://stackoverflow.com/users/2746110/arefe) on [Stackoverflow](https://stackoverflow.com/questions/51808588/run-hadoop-in-the-mac-os?answertab=votes#tab-top), so also give it a try if you have any questions.

First we search for Hadoop on brew and install the latest version.
```bash
$ brew update

$ brew search hadoop

$ brew install hadoop
```

The installation is performed on /usr/local/Cellar/hadoop/

Go to /usr/local/Cellar/hadoop/3.2.1/libexec/etc/hadoop/ and modify the following files:

#### hadoop-env.sh
```bash
export JAVA_HOME="/Library/Java/JavaVirtualMachines/jdk1.8.0_172.jdk/Contents/Home"
export HADOOP_OPTS="$HADOOP_OPTS -Djava.net.preferIPv4Stack=true -Djava.security.krb5.realm= -Djava.security.krb5.kdc="
```

As you can see, JAVA_HOME matches the Java 1.8.0_172 version I was planning to use.

#### core-site.xml
```bash
<configuration>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/usr/local/Cellar/hadoop/hdfs/tmp</value>
    <description>A base for other temporary directories.</description>
  </property>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:8020</value>
  </property>
</configuration>
```

#### mapred-site.xml
```bash
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
  <property>
    <name>yarn.app.mapreduce.am.env</name>
    <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
  </property>
  <property>
    <name>mapreduce.map.env</name>
    <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
  </property>
  <property>
    <name>mapreduce.reduce.env</name>
    <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
  </property>
</configuration>
```

#### hdfs-site.xml
```bash
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
</configuration>
```

#### yarn-site.xml
```bash
<configuration>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
  </property>
  <property>
    <name>yarn.nodemanager.disk-health-checker.max-disk-utilization-per-disk-percentage</name>
    <value>98.5</value>
  </property>
</configuration>
```

Once this changes are done, generate an SSH key.
```bash
$ ssh-keygen -t rsa -P ''
```

Copy your newly generated SSH public key into your SSH authorized keys.
```bash
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

Test your SSH connection
```bash
$ ssh localhost

Last login: Sun Nov 17 23:46:23 2019
```

Before we move on, set your HADOOP_HOME and the daemons to start and stop Hadoop on your configuration file for your bash shell. In my case I use [Zsh](http://www.zsh.org/) so I have to modify my ~/.zshrc file:
```bash
alias start_hadoop="/usr/local/Cellar/hadoop/3.2.1/sbin/start-all.sh" # start Hadoop daemons
alias stop_hadoop="/usr/local/Cellar/hadoop/3.2.1/sbin/stop-all.sh" # stop Hadoop daemons
export HADOOP_HOME=/usr/local/Cellar/hadoop/3.2.1/libexec
```

Remember to source your configuration file so the changes take effect.
```bash
$ source ~/.zshrc
```

Format the Hadoop file system
```bash
$ hdfs namenode -format
```

Now we can start Hadoop on our Mac.
```bash
$ start_hadoop

WARNING: Attempting to start all Apache Hadoop daemons as Macbook.local in 10 seconds.
WARNING: This is not a recommended production deployment configuration.
WARNING: Use CTRL-C to abort.
Starting namenodes on [localhost]
Starting datanodes
Starting secondary namenodes [MacBook.local]
2019-11-18 00:23:50,553 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Starting resourcemanager
Starting nodemanagers
```

You can verify your Hadoop cluster on http://localhost:8088/cluster

### Hive
Now we'll focus on installing [Hive](https://hive.apache.org/). The Hive version that will be installed is 3.1.2

Once again, we install Hive through Homebrew.

```bash
$ brew update

$ brew search hive

$ brew install hive
```

The installation is performed on /usr/local/Cellar/hive/

Set the HIVE_HOME on your configuration file for your bash shell. Remember I use Zsh, so I have to modify my ~/.zshrc file:

```bash
export HIVE_HOME=/usr/local/Cellar/hive/3.1.2/libexec
```

Don't forget to source your configuration file:
```bash
source ~/.zshrc
```

On the /usr/local/Cellar/hive/3.1.2/libexec/conf copy the hive-default.xml.template to hive-site.xml

```bash
$ cp hive-default.xml.template hive-site.xml
```

On the hive-site.xml file you will have to modify or add the following lines.
```bash
  <property>
    <name>hive.exec.local.scratchdir</name>
    <value>/tmp/hive</value>
    <description>Local scratch space for Hive jobs</description>
  </property>
  <property>
    <name>hive.downloaded.resources.dir</name>
    <value>/tmp/hive</value>
    <description>Temporary local directory for added resources in the remote file system.</description>
  </property>
  <property>
    <name>hive.metastore.schema.verification</name>
    <value>false</value>
    <description>
      Enforce metastore schema version consistency.
      True: Verify that version information stored in is compatible with one from Hive jars.  Also disable automatic
            schema migration attempt. Users are required to manually migrate schema after Hive upgrade which ensures
            proper metastore schema migration. (Default)
      False: Warn if the version information stored in metastore doesn't match with one from in Hive jars.
    </description>
  </property>
  <property>
    <name>hive.querylog.location</name>
    <value>/tmp/hive</value>
    <description>Location of Hive run time structured log file</description>
  </property>
  <property>
    <name>hive.druid.metadata.db.type</name>
    <value>postgresql</value>
    <description>
      Expects one of the pattern in [mysql, postgresql, derby].
      Type of the metadata database.
    </description>
  </property>
  <property>
    <name>hive.druid.metadata.uri</name>
    <value>jdbc:postgresql://127.0.0.1:5432/</value>
    <description>URI to connect to the database (for example jdbc:mysql://hostname:port/DBName).</description>
  </property>
  <property>
    <name>datanucleus.fixedDatastore</name>
    <value>false</value>
  </property>
```

Remember I'm using PostgreSQL 9.6.15, so my hive.driud.metadata.db.type is postgresql and the URI to connect to the database (hive.druid.metadata.uri) is my local database. You must change this settings accordingly.

Download the JDBC to connect to the database and place it on /usr/local/Cellar/hive/3.1.2/libexec/lib
The JDBC I downloaded for PostgreSQL can be found [here](https://jdbc.postgresql.org/download.html).

```bash
$ wget -P ~/Desktop https://jdbc.postgresql.org/download/postgresql-42.2.8.jar

$ cp ~/Desktop/postgresql-42.2.8.jar /usr/local/Cellar/hive/3.1.2/libexec/lib
```

As a sidenote, when I try to run Hive for the first time I get the following error:
```bash
Exception in thread "main" java.lang.NoSuchMethodError: com.google.common.base.Preconditions.checkArgument(ZLjava/lang/String;Ljava/lang/Object;)V
	at org.apache.hadoop.conf.Configuration.set(Configuration.java:1357)
	at org.apache.hadoop.conf.Configuration.set(Configuration.java:1338)
	at org.apache.hadoop.mapred.JobConf.setJar(JobConf.java:536)
	at org.apache.hadoop.mapred.JobConf.setJarByClass(JobConf.java:554)
	at org.apache.hadoop.mapred.JobConf.<init>(JobConf.java:448)
	at org.apache.hadoop.hive.conf.HiveConf.initialize(HiveConf.java:5141)
	at org.apache.hadoop.hive.conf.HiveConf.<init>(HiveConf.java:5099)
	at org.apache.hadoop.hive.common.LogUtils.initHiveLog4jCommon(LogUtils.java:97)
	at org.apache.hadoop.hive.common.LogUtils.initHiveLog4j(LogUtils.java:81)
	at org.apache.hadoop.hive.cli.CliDriver.run(CliDriver.java:699)
	at org.apache.hadoop.hive.cli.CliDriver.main(CliDriver.java:683)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.hadoop.util.RunJar.run(RunJar.java:323)
	at org.apache.hadoop.util.RunJar.main(RunJar.java:236)
```

To fix this issue, I had to update the guava Jar file on Hive.
```bash
$ ls /usr/local/Cellar/hadoop/3.2.1/libexec/share/hadoop/common/lib

guava-27.0-jre.jar

$ ls /usr/local/Cellar/hive/3.1.2/libexec/lib

guava-19.0.jar

$ cp /usr/local/Cellar/hadoop/3.2.1/libexec/share/hadoop/common/lib/guava-27.0-jre.jar /usr/local/Cellar/hive/3.1.2/libexec/lib

$ rm /usr/local/Cellar/hive/3.1.2/libexec/lib/guava-19.0.jar
```

Once this issue is fixed, run Hive on your terminal to ensure everything is working:
```bash
$ hive

hive > show databases;
```
