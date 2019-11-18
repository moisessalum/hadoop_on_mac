# Hadoop Ecosystem on Mac

## Introduction
Thank you for your time and interest in this repository.

I'm a Data Engineer and I was interested in testing and installing the Hadoop ecosystem on my personal Mac to play around.
I share this installation steps because I haven't found enough documentation to install Hadoop on Mac.

## Installation

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

Before we move on, set your HADOOP_HOME and the deamons to start and stop Hadoop on your configuration file for your bash shell. In my case I use [Zsh](http://www.zsh.org/) so I have to modify my ~/.zshrc file:
```bash
alias start_hadoop="/usr/local/Cellar/hadoop/3.2.1/sbin/start-all.sh" # start Hadoop daemons
alias stop_hadoop="/usr/local/Cellar/hadoop/3.2.1/sbin/stop-all.sh" # stop Hadoop daemons
export HADOOP_HOME=/usr/local/Cellar/hadoop/3.2.1/libexec
```

Remember to source your configuration file so the changes take effect.
```bash
$ source ~/.zshrc
```

First we search for Hadoop on brew and install the latest version.
```bash
$ brew search hadoop

$ brew install hadoop
```

The installation is performed on /usr/local/Cellar/hadoop/ .

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

You can verify your Hadoop cluster on http://localhost:8088/cluster .
