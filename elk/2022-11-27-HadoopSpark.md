---
layout: single
title: "(따라하면 된다.)하둡 멀티 클러스터 설치하기"

---

- 완벽하게 따라함. https://www.youtube.com/watch?v=qiEQ7gnYRfk&t=1351s
- 추후에 빠른 설치를 위한 기록

## 1. 다양한 인스턴스에 접속하기 편하게 설계하자

```bash

# 키가 있는 위치에서 

vim config

HOST master
        HostName {외부 ip}
        User jwjinn
        IdentityFile /home/joo/.ssh/es-rsa.pem

chmod 440 config

ssh master # master로 접속한다.

```

- 하는 이유는 많은 인스턴스에 접속을 해야하다보니, 호스트 명으로 쉽게 접속을 하기 위해서이다.


## 2. update

```bash

jwjinn@master:~$ sudo apt-get -y update

jwjinn@master:~$ sudo apt-get -y upgrade

jwjinn@master:~$ sudo apt-get -y dist-upgrade


jwjinn@master:~$ sudo apt-get install -y vim wget unzip ssh openssh-* net-tools

```

## 자바 설치

```bash
sudo apt-get install -y openjdk-11-jdk

sudo vim ~/.bashrc #환경등록

export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH=${PATH}:${JAVA_HOME}/bin

```
- 기본적으로 원하는 방식으로 자바를 다운 받는 것은 괜찮지만, 이 방법의 장점은 root 계정에서 터미널에 jps를 입력했을 때, jps 목록들이 뜬다.

## 파이선 설치

```bash

sudo apt install python3-pip -y

jwjinn@master:~$ sudo vim ~/.bashrc #환경등록

# python
alias python=python3 # python으로 python3을 호출할 수 있도록 별칭 설정.

jwjinn@master:~$ source ~/.bashrc
jwjinn@master:~$ pip -V
pip 20.0.2 from /usr/lib/python3/dist-packages/pip (python 3.8)
jwjinn@master:~$ python -V
Python 3.8.10
jwjinn@master:~$ pip3 -V
pip 20.0.2 from /usr/lib/python3/dist-packages/pip (python 3.8

```

### pyspark

```
sudo pip3 install pyspark

```

## Hadoop 다운

```bash
jwjinn@master:~$ wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.4/hadoop-3.3.4.tar.gz

sudo tar -xvzf hadoop-3.3.4.tar.gz -C /usr/local/
# /usr/local/에 압축을 해제했다.


sudo chown root:root -R /usr/local/hadoop-3.3.4
# 나중에 루트로 통신을 할 것이기에 미리 설정을 했다.

```

## Spark 다운

```bash

wget https://dlcdn.apache.org/spark/spark-3.2.2/spark-3.2.2-bin-without-hadoop.tgz

wget https://dlcdn.apache.org/spark/spark-3.2.3/spark-3.2.3-bin-without-hadoop.tgz

jwjinn@master:~$ sudo tar xvzf spark-3.2.2-bin-without-hadoop.tgz -C /usr/local/

```

## 환경설정 등록

```bash

# JAVA
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH=${PATH}:${JAVA_HOME}/bin

#python
alias python=python3


#Hadoop
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export YARN_CONF_DIR=$HADOOP_HOME/etc/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin


#Spark
export SPARK_HOME=/usr/local/spark
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
export SPARK_DIST_CLASSPATH=$($HADOOP_HOME/bin/hadoop classpath)

```

## HADOOP 설정

### sudo vim core-site.xml

```xml

<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://master:9000</value>
    </property>
</configuration>

```
- 모든 데이터노드들이 master에 9000번 포트로 통신을 한다.

### sudo vim hdfs-site.xml


```xml

<configuration>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>

    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///hdfs_dir/namenode</value>
    </property>

    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///hdfs_dir/datanode</value>
    </property>

    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>worker01:50090</value>
    </property>
</configuration>

```

### sudo vim yarn-site.xml

```xml

<configuration>
    <property>
        <name>yarn.nodemanager.local-dirs</name>
        <value>file:///hdfs_dir/yarn/local</value>
    </property>

    <property>
        <name>yarn.nodemanager.log-dirs</name>
        <value>file:///hdfs_dir/yarn/logs</value>
    </property>

    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>master</value>
    </property>
</configuration>

```

### sudo vim mapred-site.xml

```xml

<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>

```

### sudo vim hadoop-env.sh

```bash

export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64

#hadoop user
export HDFS_NAMENODE_USER="root"
export HDFS_DATANODE_USER="root"
export HDFS_SECONDARYNAMENODE_USER="root"
export YARN_RESOURCEMANAGER_USER="root"
export YARN_NODEMANAGER_USER="root"

```

## spark 설정


### spark-defaults.conf
```bash

jwjinn@master:/usr/local/spark/conf$ 

jwjinn@master:/usr/local/spark/conf$ sudo cp spark-defaults.conf.template spark-defaults.conf

jwjinn@master:/usr/local/spark/conf$ sudo vim spark-defaults.conf


```

```conf

spark.master                            yarn
; spark.eventLog.enabled                  true
; spark.eventLog.dir                      hdfs://namenode:8021/spark_enginelog

; 위에서 주석처리하니 된다.
```

### spark-env.sh

```bash

jwjinn@master:/usr/local/spark/conf$ sudo cp spark-env.sh.template spark-env.sh
jwjinn@master:/usr/local/spark/conf$ sudo vim spark-env.sh


export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64

export HADOOP_HOME=/usr/local/hadoop

export SPARK_MASTER_HOST=master
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export YARN_CONF_DIR=$HADOOP_HOME/etc/hadoop
export SPARK_DIST_CLASSPATH=$(/usr/local/hadoop/bin/hadoop classpath)

export PYSPARK_PYTHON=/usr/bin/python3
export PYSPARK_DRIVER_PYTHON=/usr/bin/python3


```


## 이미지 만들기
- 머신 이미지를 만들어서 접속한다.

## 접속하기 쉽게

```bash

HOST master
        HostName {외부ip}
        User jwjinn
        IdentityFile /home/joo/.ssh/es-rsa.pem

HOST worker01
        HostName {외부ip}
        User jwjinn
        IdentityFile /home/joo/.ssh/es-rsa.pem


HOST worker02
        HostName {외부ip}
        User jwjinn
        IdentityFile /home/joo/.ssh/es-rsa.pem


HOST worker03
        HostName {외부ip}
        User jwjinn
        IdentityFile /home/joo/.ssh/es-rsa.pem

```
# 각 인스턴스 호스트 네임 변경
- 구글 클라우드는 인스턴트를 생성을 할때, 호스트 네임도 설정이 되므로 할 필요는 없으나, 알우두면 좋다.
- 호스트 네임으로 통신을 할 것이기에 중요하다.

### 확인 및 변경
```bash

hostname # hostname확인

sudo hostnamectl set-hostname master # 각 인스턴스마다
sudo hostnamectl set-hostname worker01
sudo hostnamectl set-hostname worker02
sudo hostnamectl set-hostname worker03

```
## 호스트이름으로 통신이 가능하게 하자.

- 각 인스턴스 마다 master, worker01 이라는 이름으로 통신이 가능하도록 설정을 한다.

```bash

sudo vim /etc/hosts

10.178.0.5 master
10.178.0.6 worker01
10.178.0.7 worker02
10.178.0.8 worker03


```

## 서로 간의 인스턴스끼리 묻지 말고 통신하도록 하자

- 모든 인스턴스들에 적용
```bash

sudo vim /etc/ssh/sshd_config

PermitRootLogin yes

PasswordAuthentication yes

systemctl restart sshd

passwd

```

## key를 만들자

```bash

ssh-keygen # 모든 인스턴스들에 키를 만든다.

cd ~/.ssh # 위치

# 키를 만들면 pub키가 나오는데, 이 pub키들을 인스턴스들에게 나눠주면서 인증없이 되게 한다.

cd ~/.ssh/known_hosts # pub key들의 정보들을 모아둔다.

ssh-copy-id root@master # pub키들을 전송한다.
ssh-copy-id root@worker01
ssh-copy-id root@worker02
ssh-copy-id root@worker03


ssh master # 위 방식으로 통신이 되어야 한다.
ssh worker01
ssh worker02
ssh worker03
```

## namenode와 secondary_nameNode에서 실행

- master와 secondary_nameNode에서
```bash

/usr/local/hadoop-3.3.4/bin/hdfs namenode -format /hdfs_dir

# 심볼릭한 경우
/usr/local/hadoop/bin/hdfs namenode -format /hdfs_dir

```

## workerNode, DataNode에서

```bash

/usr/local/hadoop-3.3.4/bin/hdfs datanode -format /hdfs_dir/

# 심볼릭한 경우
/usr/local/hadoop/bin/hdfs datanode -format /hdfs_dir

```

## master서버에서 워커들을 등록하자. (스파크)

```bash
root@master:/usr/local/spark/conf# cp workers.template workers
vi /usr/local/hadoop-3.3.4/etc/hadoop/workers

#localhost

worker01
worker02
worker03

```

## master 서버에서 워커들을 등록하자(하둡)

```bash

root@master:/usr/local/hadoop/etc/hadoop# pwd
/usr/local/hadoop/etc/hadoop
root@master:/usr/local/hadoop/etc/hadoop# vi workers

```
- 해당 경로에 워커들을 등록해야 한다. 기본은 localhost, 등록해야 데이터노드가 slave에도 올라온다.


## 실행하자 master에서

```bash

#하둡
/usr/local/hadoop-3.3.4/sbin/start-all.sh #실행
/usr/local/hadoop-3.3.4/sbin/stop-all.sh # 정지


#spark
/usr/local/spark/sbin/start-all.sh # 스파크 실행
/usr/local/spark/sbin/stop-all.sh
```