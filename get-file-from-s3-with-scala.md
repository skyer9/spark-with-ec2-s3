# Spark 에서 AWS S3 데이타 가져오기(With Scala)

Scala 를 이용해 S3 에 저장되어 있는 데이타를 가져온다.

## 1. 스파크 클러스터 생성하기

```sh
flintrock launch spark
```

## 2. 샘플 프로젝트 생성하기

```sh
$ mkdir test_prj
$ cd test_prj/
$ sbt
> set name := "MyProject"
> set version := "0.1"
> set scalaVersion := "2.11.8"
> session save
> exit
```

Main.scala 에 아래 내용을 입력한다.

```sh
$ mkdir -p src/main/scala
$ vi src/main/scala/Main.scala
import org.apache.spark.SparkContext
import org.apache.spark.SparkContext._
import org.apache.spark.SparkConf

object Main {
    def main(args: Array[String]) {

        val conf = new SparkConf().setAppName("HelloWorld")
        val sc = new SparkContext(conf)

        // S3 접속을 위한 설정하기
        val region = "ap-northeast-2"
        System.setProperty("com.amazonaws.services.s3.enableV4", "true")
        sc.hadoopConfiguration.set("fs.s3a.impl", "org.apache.hadoop.fs.s3a.S3AFileSystem")
        sc.hadoopConfiguration.set("com.amazonaws.services.s3.enableV4", "true")
        sc.hadoopConfiguration.set("fs.s3a.endpoint", "s3." + region + ".amazonaws.com")

        val rdd = sc.textFile("s3a://tenbyten-weblog-seoul/20180430/www*-www-*.gz")
                    .take(10)
                    .foreach(_ => println _)

        sc.stop()
    }
}
```

```sh
$ vi project/plugins.sbt
addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "0.14.5")

$ vi build.sbt
......
val sparkVersion = "2.3.0"
libraryDependencies ++= Seq(
  "org.apache.spark" %% "spark-core" % sparkVersion % "provided",
  "org.apache.spark" %% "spark-mllib" % sparkVersion % "provided"
)
......
```

## 3. 컴파일하고 실행하기

```sh
sbt assembly
spark-submit --class Main --master local target/scala-2.11/MyProject-assembly-0.1.jar
```

## 4. 스파크 클러스터에서 실행하기

```sh
# 클러스터의 모든 노드에 jar 파일을 복사한다.
$ flintrock copy-file spark target/scala-2.11/MyProject-assembly-0.1.jar ./

클러스터 마스터 노드에서 실행한다.
$ flintrock login spark
<SparkMaster> $ spark-submit --deploy-mode cluster \
    --master spark://<스파크 클러스터 마스터 노드 내부아이피>:6066 \
    --jars /home/ec2-user/.ivy2/cache/com.amazonaws/aws-java-sdk/jars/aws-java-sdk-1.7.4.jar,/home/ec2-user/.ivy2/cache/org.apache.hadoop/hadoop-aws/jars/hadoop-aws-2.7.6.jar \
    --executor-memory 512M \
    --driver-memory 512M \
    --executor-cores 1 \
    --conf spark.hadoop.fs.s3a.endpoint=s3.ap-northeast-2.amazonaws.com \
    --conf spark.hadoop.fs.s3a.impl=org.apache.hadoop.fs.s3a.S3AFileSystem \
    --conf spark.executor.extraJavaOptions=-Dcom.amazonaws.services.s3.enableV4=true \
    --conf spark.driver.extraJavaOptions=-Dcom.amazonaws.services.s3.enableV4=true \
    --conf spark.hadoop.fs.s3a.access.key=<AWS 액세스키> \
    --conf spark.hadoop.fs.s3a.secret.key=<AWS 보안키> \
    --class Main \
    MyProject-assembly-0.1.jar
```
