# Hadoop Installation Guide - Single Node (Pseudo-Distributed) Mode

**For Ubuntu Systems**
This guide walks you through installing **Apache Hadoop 3.3.6** in **single-node (pseudo-distributed) mode** on a fresh Ubuntu system.

---

## Prerequisites

* Ubuntu Linux (18.04 or later recommended)
* User account with `sudo` privileges
* Internet connection for downloading packages

---

## Step 1: Update System Packages

Open a terminal and run:

```bash
sudo apt-get update -y
sudo apt-get upgrade -y
```

---

## Step 2: Install Java (OpenJDK 11)

Hadoop requires Java to run.

```bash
sudo apt-get install -y openjdk-11-jdk
```

Verify installation:

```bash
java -version
javac -version
```

You should see Java 11 version information.

---

## Step 3: Install Required Utilities

Install `curl` and `tar` (if not already present):

```bash
sudo apt-get install -y curl tar
```

---

## Step 4: Install and Configure SSH

Hadoop uses SSH for node communication.

```bash
sudo apt-get install -y openssh-client openssh-server
```

Generate SSH key pair:

```bash
ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa
```

Add the public key to `authorized_keys` for passwordless SSH:

```bash
mkdir -p ~/.ssh
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

Test passwordless SSH to localhost:

```bash
ssh -o StrictHostKeyChecking=accept-new localhost
```

Type `exit` to close the SSH session.

---

## Step 5: Set JAVA_HOME Environment Variable

Find your Java path:

```bash
readlink -f $(which javac)
```

This typically returns:

```
/usr/lib/jvm/java-11-openjdk-amd64/bin/javac
```

Set `JAVA_HOME` (temporary):

```bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```

---

## Step 6: Download Hadoop

```bash
cd /tmp
curl -L https://dlcdn.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz -o hadoop-3.3.6.tar.gz
```

> If the above URL doesn't work, visit [Hadoop Releases](https://hadoop.apache.org/releases.html) to find the latest mirror link.

---

## Step 7: Extract and Install Hadoop

```bash
sudo mkdir -p /opt/hadoop-3.3.6
sudo tar -xzf hadoop-3.3.6.tar.gz -C /opt
sudo ln -sfn /opt/hadoop-3.3.6 /opt/hadoop
sudo chown -R $USER:$USER /opt/hadoop-3.3.6
rm /tmp/hadoop-3.3.6.tar.gz
```

---

## Step 8: Configure Hadoop Environment Variables

Create configuration directory:

```bash
mkdir -p ~/.hadoop
```

Create environment file:

```bash
cat > ~/.hadoop/env.sh <<'EOF'
# Hadoop environment
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export HADOOP_HOME=/opt/hadoop
export HADOOP_INSTALL=/opt/hadoop
export HADOOP_COMMON_HOME=/opt/hadoop
export HADOOP_HDFS_HOME=/opt/hadoop
export HADOOP_MAPRED_HOME=/opt/hadoop
export HADOOP_YARN_HOME=/opt/hadoop
export HADOOP_CONF_DIR=/opt/hadoop/etc/hadoop
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
EOF
```

Load environment variables:

```bash
source ~/.hadoop/env.sh
```

Add to `~/.bashrc` for automatic loading:

```bash
echo "source ~/.hadoop/env.sh" >> ~/.bashrc
```

Verify Hadoop installation:

```bash
hadoop version
```

You should see **Hadoop 3.3.6**.

---

## Step 9: Configure Hadoop for Single-Node Mode

Create data directories:

```bash
mkdir -p ~/hadoop_data/nn ~/hadoop_data/dn ~/hadoop_data/tmp
```

### 9.1 Edit `hadoop-env.sh`

```bash
nano /opt/hadoop/etc/hadoop/hadoop-env.sh
```

Set:

```bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```

---

### 9.2 Edit `core-site.xml`

```bash
nano /opt/hadoop/etc/hadoop/core-site.xml
```

Replace with:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>
  </property>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>file:///home/YOUR_USERNAME/hadoop_data/tmp</value>
  </property>
</configuration>
```

> Replace `YOUR_USERNAME` with your actual username.

---

### 9.3 Edit `hdfs-site.xml`

```bash
nano /opt/hadoop/etc/hadoop/hdfs-site.xml
```

Replace with:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:///home/YOUR_USERNAME/hadoop_data/nn</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///home/YOUR_USERNAME/hadoop_data/dn</value>
  </property>
</configuration>
```

---

### 9.4 Edit `mapred-site.xml`

```bash
nano /opt/hadoop/etc/hadoop/mapred-site.xml
```

Replace with:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
  <property>
    <name>mapreduce.application.classpath</name>
    <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
  </property>
</configuration>
```

---

### 9.5 Edit `yarn-site.xml`

```bash
nano /opt/hadoop/etc/hadoop/yarn-site.xml
```

Replace with:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.env-whitelist</name>
    <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_HOME,PATH,LANG,TZ,HADOOP_MAPRED_HOME</value>
  </property>
</configuration>
```

---

## Step 10: Format the NameNode

> **Important:** Only do this on first-time installation.

```bash
hdfs namenode -format
```

You should see:

```
Storage directory ... has been successfully formatted.
```

---

## Step 11: Start Hadoop Services

### 11.1 Start HDFS

```bash
start-dfs.sh
```

### 11.2 Start YARN

```bash
start-yarn.sh
```

### 11.3 Verify Services

```bash
jps
```

Expected Java processes:

* NameNode
* DataNode
* SecondaryNameNode
* ResourceManager
* NodeManager
* Jps

---

## Step 12: Verify Hadoop Web Interfaces

* **HDFS NameNode UI:** [http://localhost:9870](http://localhost:9870)
* **YARN ResourceManager UI:** [http://localhost:8088](http://localhost:8088)

Both should display Hadoop dashboards.

---

## Step 13: Test Hadoop with HDFS Commands

```bash
# Create directories in HDFS
hdfs dfs -mkdir -p /user/$USER
hdfs dfs -mkdir -p /user/$USER/input

# List HDFS root directory
hdfs dfs -ls /

# Create a test file and upload
echo "Hello Hadoop!" > /tmp/test.txt
hdfs dfs -put /tmp/test.txt /user/$USER/input/

# List uploaded files
hdfs dfs -ls /user/$USER/input/

# Read the file from HDFS
hdfs dfs -cat /user/$USER/input/test.txt
```

---

## Step 14: Stopping Hadoop Services

```bash
stop-yarn.sh
stop-dfs.sh
```

Verify:

```bash
jps
```

Only `Jps` should remain.

---

## Step 15: Starting Hadoop After Reboot

```bash
source ~/.hadoop/env.sh
start-dfs.sh
start-yarn.sh
```

> Environment variables load automatically if added to `~/.bashrc`.

---

## Troubleshooting Tips

| Issue                              | Solution                                                                                                 |
| ---------------------------------- | -------------------------------------------------------------------------------------------------------- |
| "Permission denied" when using SSH | Ensure `~/.ssh/authorized_keys` has correct permissions: `chmod 600 ~/.ssh/authorized_keys`              |
| Java not found                     | Verify `JAVA_HOME` is set correctly in `hadoop-env.sh` and `~/.hadoop/env.sh`                            |
| NameNode won't start               | Check logs: `/opt/hadoop/logs/hadoop-*-namenode-*.log`                                                   |
| Port already in use                | Stop any existing Hadoop processes: `stop-yarn.sh && stop-dfs.sh` and manually kill remaining processes  |
| Cannot format NameNode again       | WARNING: destroys all data. Run: `rm -rf ~/hadoop_data/nn/* ~/hadoop_data/dn/* && hdfs namenode -format` |

---

## Useful Hadoop Commands

```bash
hadoop version                # Check Hadoop version
jps                            # View running Java processes
hdfs dfsadmin -report          # Check HDFS health
hdfs dfs -mkdir /path           # Create directory in HDFS
hdfs dfs -put localfile /hdfs/path  # Upload file to HDFS
hdfs dfs -get /hdfs/path localfile  # Download file from HDFS
hdfs dfs -ls /path              # List HDFS contents
hdfs dfs -rm -r /path           # Delete file/directory from HDFS
ls /opt/hadoop/logs/            # View Hadoop logs
```

---

## Congratulations! 🎉

You have successfully installed and configured **Apache Hadoop** in single-node mode.

Next steps:

* Explore MapReduce programming
* Run example jobs:

  ```bash
  hadoop jar /opt/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar
  ```
* Learn Hive, HBase, Pig, and other Hadoop ecosystem tools
* Practice HDFS commands and operations

Official documentation: [https://hadoop.apache.org/docs/stable/](https://hadoop.apache.org/docs/stable/)
