# Create a new folder and name it Flume in your data_ingest repository in github.

## 1. Create a new flume configuration file with the following:

```
vi $DEVSH/exercises/flume/Netcat.conf

# Netcat.conf:

# Name the components on this agent
agent1.sources = webserver-log-source
agent1.sinks = hdfs-sink
agent1.channels = memory-channel

# Describe/configure the source
agent1.sources.webserver-log-source.type = Netcat
agent1.sources.webserver-log-source.bind = localhost
agent1.sources.webserver-log-source.port = 44444

agent1.sources.webserver-log-source.spoolDir = /flume/weblogs_spooldir
agent1.sources.webserver-log-source.channels = memory-channel

# Describe the sink
agent1.sinks.hdfs-sink.type = logger
agent1.sinks.hdfs-sink.channel = memory-channel

agent1.sinks.hdfs-sink.hdfs.path = /loudacre/weblogs_flume/
agent1.sinks.hdfs-sink.hdfs.rollInterval = 0
agent1.sinks.hdfs-sink.hdfs.rollSize = 524288
agent1.sinks.hdfs-sink.hdfs.rollCount = 0
agent1.sinks.hdfs-sink.hdfs.fileType = DataStream

# Use a channel which buffers events in memory
agent1.channels.memory-channel.type = memory
agent1.channels.memory-channel.capacity = 1000
agent1.channels.memory-channel.transactionCapacity = 100

```

# Run the Flume Agent

```
flume-ng agent \
--conf /etc/flume-ng/conf \
--conf-file $DEVSH/exercises/flume/Netcat.conf \
--name agent1 -Dflume.root.logger=INFO,console


```

# [backup] Collecting Web Server Logs with Apache Flume
## 1. Create an HDFS Directory for Flume-Ingested Data

```

hdfs dfs -mkdir -p /loudacre/weblogs_flume

[training@localhost ~]$ hdfs dfs -mkdir -p /loudacre/weblogs_flume
[training@localhost ~]$ hdfs dfs -ls /loudacre
Found 5 items
drwxrwxrwx   - training supergroup          0 2019-04-07 22:51 /loudacre/accounts
-rw-rw-rw-   1 training supergroup      15191 2019-04-07 22:00 /loudacre/base_stations.tsv
drwxrwxrwx   - training supergroup          0 2019-04-07 22:04 /loudacre/basestations_import
drwxrwxrwx   - training supergroup          0 2019-04-07 22:00 /loudacre/kb
drwxrwxrwx   - training supergroup          0 2019-04-08 18:29 /loudacre/weblogs_flume


```


## Create a Local Directory for Web Server Log Output

```

sudo mkdir -p /flume/weblogs_spooldir


[training@localhost ~]$ sudo mkdir -p /flume/weblogs_spooldir
[training@localhost ~]$ ll /flume
total 4
drwxr-xr-x 2 root root 4096 Apr  8 18:32 weblogs_spooldir

[training@localhost ~]$ sudo chmod a+w -R /flume
[training@localhost ~]$ ll /flume
total 4
drwxrwxrwx 2 root root 4096 Apr  8 18:32 weblogs_spooldir


```


## Configure Flume

```

[training@localhost ~]$ more $DEVSH/exercises/flume/spooldir.conf
# spooldir.conf: A Spooling Directory Source

# Name the components on this agent
agent1.sources = webserver-log-source
agent1.sinks = hdfs-sink
agent1.channels = memory-channel

# Describe/configure the source
agent1.sources.webserver-log-source.type = spooldir
agent1.sources.webserver-log-source.spoolDir = /flume/weblogs_spooldir
agent1.sources.webserver-log-source.channels = memory-channel

# Describe the sink
agent1.sinks.hdfs-sink.type = hdfs
agent1.sinks.hdfs-sink.hdfs.path = /loudacre/weblogs_flume/
agent1.sinks.hdfs-sink.channel = memory-channel
agent1.sinks.hdfs-sink.hdfs.rollInterval = 0
agent1.sinks.hdfs-sink.hdfs.rollSize = 524288
agent1.sinks.hdfs-sink.hdfs.rollCount = 0
agent1.sinks.hdfs-sink.hdfs.fileType = DataStream

# Use a channel which buffers events in memory
agent1.channels.memory-channel.type = memory
agent1.channels.memory-channel.capacity = 100000
agent1.channels.memory-channel.transactionCapacity = 1000



```

# Run the Flume Agent

```
flume-ng agent \
--conf /etc/flume-ng/conf \
--conf-file $DEVSH/exercises/flume/spooldir.conf \
--name agent1 -Dflume.root.logger=INFO,console


```

# Simulate Apache Web Server Output
```
$DEVSH/exercises/flume/copy-move-weblogs.sh \
/flume/weblogs_spooldir



-rw-rw-rw-   1 training supergroup     527774 2019-04-08 18:42 /loudacre/weblogs_flume/FlumeData.1554774107829
-rw-rw-rw-   1 training supergroup     527803 2019-04-08 18:42 /loudacre/weblogs_flume/FlumeData.1554774107830
-rw-rw-rw-   1 training supergroup     527867 2019-04-08 18:42 /loudacre/weblogs_flume/FlumeData.1554774107831
-rw-rw-rw-   1 training supergroup     527803 2019-04-08 18:42 /loudacre/weblogs_flume/FlumeData.1554774107832
-rw-rw-rw-   1 training supergroup      20149 2019-04-08 18:42 /loudacre/weblogs_flume/FlumeData.1554774107833
[training@localhost ~]$ hdfs dfs -ls /loudacre/weblogs_flume


```