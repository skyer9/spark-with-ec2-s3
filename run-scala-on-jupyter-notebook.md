# Jupyter Notebook 에서 Scala 실행하기

[스칼라 노트북](https://github.com/jupyter-scala/jupyter-scala) 을 설치한다.

## 1. AMI 인스턴스 생성하기

Amazon Linux AMI 를 생성하고, 인스턴스 유형은 t2.medium, SSD 용량은 20G 로 합니다.

인스턴스가 생성되면 아래 명령을 실행해준다.

```sh
sudo yum -y update
```

## 2. 자바 업그레이드하기

```sh
sudo yum -y install java-1.8.0
sudo yum -y remove java-1.7.0-openjdk
```

## 2. 파이선 3.x 을 설치하기

```sh
sudo yum list | grep python3
sudo yum -y install python36
python3 -V
```

## 3. jupyter nootbook 설치하기

```sh
$ sudo pip-3.6 install jupyter

$ jupyter --version
4.4.0
```

## 4. jupyter-scala 설치하기

jupyter-scala 를 설치한다.

```sh
$ cd ~/
$ wget https://raw.githubusercontent.com/alexarchambault/jupyter-scala/master/jupyter-scala

$ vi jupyter-scala
......
SCALA_VERSION=2.11.8
......

$ sh jupyter-scala

$ jupyter kernelspec list
Available kernels:
  scala      /home/ec2-user/.local/share/jupyter/kernels/scala
  python3    /usr/local/share/jupyter/kernels/python3
```

노트북을 실행한다.

```sh
jupyter notebook
```

`Control-C` 를 두번 입력해서 노트북을 종료한다.

## 5. 노트북 설정 변경하기

노트북을 외부에서 접속 가능하도록 수정한다.

```sh
$ jupyter notebook --generate-config
Writing default config to: /home/ec2-user/.jupyter/jupyter_notebook_config.py
```

아래 명령으로 외부에서 접속 가능하도록 수정한다.
(!!!!!! 아래 명령은 모든 아이피에서 접속가능하도록 설정한다. 방화벽 등 기타 다른 방법으로 접속을 제한해야 한다. !!!!!!)

```sh
$ vi /home/ec2-user/.jupyter/jupyter_notebook_config.py
......
c.NotebookApp.ip = '*'
......
c.NotebookApp.port = 8888
......
```

AWS 콘솔에서 로컬(테스트용) 서버 인스턴스 보안그룹에 접속할 아이피(PC 아이피) 및 포트(8888) 를 열어준다.

아래 명령을 실행한다.

```sh
jupyter notebook
```

브라우저에서 확인한다.

`http://<로컬서버아이피>:8888/?token=91fb1aa750ade184a9c04335f20694cfb37a2a971bd3a`

## 6. 노트북을 백그라운드에서 실행하기

```sh
cd ~/
nohup jupyter notebook &
cat nohup.out
```

## 7. 노트북에서 Spark Scala 실행하기

스파크 라이브러리를 별도로 설치하므로 Spark 를 추가로 설치할 필요가 없다.

```scala
// import $exclude.`org.slf4j:slf4j-log4j12`, $ivy.`org.slf4j:slf4j-nop:1.7.21` // for cleaner logs
import $profile.`hadoop-2.7`
import $ivy.`org.apache.spark::spark-sql:2.1.0` // adjust spark version - spark >= 2.0
import $ivy.`org.apache.hadoop:hadoop-aws:2.6.4`
import $ivy.`org.jupyter-scala::spark:0.4.2` // for JupyterSparkSession (SparkSession aware of the jupyter-scala kernel)

import org.apache.spark._
import org.apache.spark.sql._
import jupyter.spark._
import jupyter.spark.session._

def main() {
    val conf = new SparkConf()
        .setAppName("ViewTogether")
        .setMaster("local")
    val sc = new JupyterSparkContext(conf)

    val sparkSession = JupyterSparkSession.builder() // important - call this rather than SparkSession.builder()
        .jupyter() // this method must be called straightaway after builder()
        // .yarn("/etc/hadoop/conf") // optional, for Spark on YARN - argument is the Hadoop conf directory
        // .emr("2.6.4") // on AWS ElasticMapReduce, this adds aws-related to the spark jar list
        .master("local") // change to "yarn-client" on YARN
        // .config("spark.executor.instances", "10")
        // .config("spark.executor.memory", "3g")
        // .config("spark.hadoop.fs.s3a.access.key", awsCredentials._1)
        // .config("spark.hadoop.fs.s3a.secret.key", awsCredentials._2)
        .appName("notebook")
        .getOrCreate()

    println("Started")

    sc.stop()
}

main()
```

`sparkSession` 은 `JupyterSparkSession` 을 통해서만 생성해야 한다. `SparkContext` 는 `JupyterSparkContext` 를 통해 생성되어야 한다.
