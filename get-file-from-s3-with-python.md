# Spark 에서 AWS S3 데이타 가져오기(With Python3.x)

Python3.x 를 이용해 S3 에 저장되어 있는 데이타를 가져온다.

## 1. 스파크 클러스터 생성하기

```sh
flintrock launch spark
```

## 2. 테스트 프로그램 생성하기

```sh
$ vi test.py
# -*- coding: utf-8 -*-
import pyspark

sc = pyspark.SparkContext.getOrCreate()
sc.setSystemProperty("com.amazonaws.services.s3.enableV4", "true")

hadoopConf = sc._jsc.hadoopConfiguration()
hadoopConf.set("fs.s3a.impl", "org.apache.hadoop.fs.s3a.S3AFileSystem")
hadoopConf.set("fs.s3a.endpoint", 's3.ap-northeast-2.amazonaws.com')

# 확장자는 csv, gz 등이 가능하다.
df = sc.textFile("s3a://버킷이름/파일이름")

print('\n\n------------------------------------------------\n\n')
print(df.take(10))
print('\n\n------------------------------------------------\n\n')

sc.stop()
```

## 3. 스파크 클러스터에서 실행하기

```sh
# 모든 스파크 클러스터 노드에 실행파일 복사
$ flintrock copy-file spark test.py ./

# 모든 스파크 클러스터 노드에 pyspark 설치
$ flintrock run-command spark 'sudo yum -y install python36'
$ flintrock run-command spark 'sudo yum -y install python36-setuptools'
$ flintrock run-command spark 'sudo easy_install-3.6 pip'
$ flintrock run-command spark 'sudo pip-3.6 install pyspark'

# 마스터 노드에 로그인
$ flintrock login spark

# 클라이언트 모드로 실행하기
<SparkMaster> $ spark-submit --master spark://172.31.31.3:7077 \
               --conf spark.hadoop.fs.s3a.endpoint=s3.ap-northeast-2.amazonaws.com \
               --conf spark.hadoop.fs.s3a.impl=org.apache.hadoop.fs.s3a.S3AFileSystem \
               --conf spark.executor.extraJavaOptions=-Dcom.amazonaws.services.s3.enableV4=true \
               --conf spark.driver.extraJavaOptions=-Dcom.amazonaws.services.s3.enableV4=true \
               --conf spark.hadoop.fs.s3a.access.key=XXXXXXXXXXXXXXXXXXXXXX \
               --conf spark.hadoop.fs.s3a.secret.key=XXXXXXXXXXXXXXXXXXXXXXXXXXXXX \
               test.py
```

## 4. yarn 설치 및 실행하기

Python 은 `Spark Standalone` 모드에서는 `cluster` 모드를 작동시킬 수 없다. 따라서, `Spark yarn` 모드를 이용해 `cluster` 모드를 작동시킨다.

```sh
<SparkMaster> $ cd ~/

<SparkMaster> $ cp hadoop/etc/hadoop/yarn-site.xml hadoop/conf/yarn-site.xml
<SparkMaster> $ vi hadoop/conf/yarn-site.xml
----------------------------------------------------------------------------
<configuration>

  <property>
    <name>yarn.acl.enable</name>
    <value>0</value>
  </property>

  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value><마스터노드 내부 아이피></value>
  </property>

  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>

  <property>
    <name>mapreduce.reduce.java.opts</name>
    <value>-Xmx400m</value>
  </property>

  <property>
    <name>yarn.scheduler.minimum-allocation-mb</name>
    <value>256</value>
  </property>

  <property>
    <!-- 서버 전체 메모리 중 1-2기가 제외하고 전부할당 -->
    <name>yarn.scheduler.maximum-allocation-mb</name>
    <value>3372</value>
  </property>

  <property>
    <name>yarn.app.mapreduce.am.resource.mb</name>
    <value>512</value>
  </property>

  <property>
    <name>yarn.app.mapreduce.am.command-opts</name>
    <value>-Xmx400m</value>
  </property>

  <property>
    <name>yarn.nodemanager.vmem-check-enabled</name>
    <value>false</value>
    <description>Whether virtual memory limits will be enforced for containers</description>
  </property>

  <property>
    <name>yarn.nodemanager.vmem-pmem-ratio</name>
    <value>4</value>
    <description>Ratio between virtual memory to physical memory when setting memory limits for containers</description>
  </property>

</configuration>
----------------------------------------------------------------------------

<SparkMaster> $ cp hadoop/etc/hadoop/hadoop-metrics2.properties hadoop/conf/hadoop-metrics2.properties
<SparkMaster> $ cp hadoop/etc/hadoop/capacity-scheduler.xml hadoop/conf/capacity-scheduler.xml
```

설정파일을 slave 노드에 복사한다.

```sh
scp hadoop/conf/* ec2-user@172.31.18.200:hadoop/conf/
```

## 5. 하둡 yarn 실행하기

```sh
<SparkMaster> $ hadoop/sbin/start-yarn.sh
```

```sh
spark-submit --master yarn \
               --deploy-mode cluster \
               --conf spark.hadoop.fs.s3a.endpoint=s3.ap-northeast-2.amazonaws.com \
               --conf spark.hadoop.fs.s3a.impl=org.apache.hadoop.fs.s3a.S3AFileSystem \
               --conf spark.executor.extraJavaOptions=-Dcom.amazonaws.services.s3.enableV4=true \
               --conf spark.driver.extraJavaOptions=-Dcom.amazonaws.services.s3.enableV4=true \
               --conf spark.hadoop.fs.s3a.access.key=XXXXXXXXXXXXXXXXXXXXXX \
               --conf spark.hadoop.fs.s3a.secret.key=XXXXXXXXXXXXXXXXXXXXXXXXXXXXX \
               test.py
```