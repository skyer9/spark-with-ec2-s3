# 로컬(테스트용) 서버 생성하기

Spark 서버 클러스터를 생성하기 전에 로컬(테스트용) 서버를 생성합니다.

## 1. AMI 인스턴스 생성하기

Amazon Linux AMI 를 생성하고, 인스턴스 유형은 t2.medium, SSD 용량은 20G 로 합니다.

인스턴스가 생성되면 아래 명령을 실행해준다.

```sh
sudo yum -y update
```

## 2. 자바 업그레이드하기

```sh
java -version
sudo yum -y install java-1.8.0
sudo yum -y remove java-1.7.0-openjdk
```

## 2. 파이선 3.x 을 설치하기

```sh
sudo yum list installed | grep python3
sudo yum -y install python36
python3 -V
```

## 3. Spark 설치하기

[https://spark.apache.org/downloads.html](https://spark.apache.org/downloads.html) 에서 다운받아 설치한다.

```sh
cd ~/
mkdir spark
cd spark
wget http://mirror.navercorp.com/apache/spark/spark-2.3.0/spark-2.3.0-bin-hadoop2.7.tgz
tar xvfz spark-2.3.0-bin-hadoop2.7.tgz
rm spark-2.3.0-bin-hadoop2.7.tgz
```

환경설정을 수정한다.

```sh
vi ~/.bashrc
......
export SPARK_HOME=/home/ec2-user/spark/spark-2.3.0-bin-hadoop2.7
export PATH=$SPARK_HOME/bin:$PATH
......
source ~/.bashrc
```

정상 작동하는지 확인한다.

```sh
$ spark-shell
2018-05-01 00:14:27 WARN  NativeCodeLoader:62 - Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
Spark context Web UI available at http://ip-172-31-18-241.ap-northeast-2.compute.internal:4040
Spark context available as 'sc' (master = local[*], app id = local-1525133673971).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.3.0
      /_/

Using Scala version 2.11.8 (OpenJDK 64-Bit Server VM, Java 1.8.0_171)
Type in expressions to have them evaluated.
Type :help for more information.

scala>
```

`:q` 를 입력하면 종료할 수 있다.

## 4. pyspark 설치하기

pip3 를 설치한다.

```sh
sudo easy_install-3.6 pip
```

pyspark 를 설치한다.

```sh
sudo pip-3.6 install pyspark
```

## 5. jupyter nootbook 설치하기

```sh
sudo pip-3.6 install jupyter
```

```sh
$ vi ~/.bashrc
......
export PYSPARK_PYTHON=/usr/bin/python3
export PYSPARK_DRIVER_PYTHON=ipython3
export PYSPARK_DRIVER_PYTHON_OPTS="notebook"
......
$ source ~/.bashrc
```

정상적으로 실행되는지 확인한다.

```sh
$ jupyter notebook
[I 00:28:55.826 NotebookApp] Writing notebook server cookie secret to /home/ec2-user/.local/share/jupyter/runtime/notebook_cookie_secret
[I 00:28:56.041 NotebookApp] Serving notebooks from local directory: /home/ec2-user/spark
[I 00:28:56.041 NotebookApp] 0 active kernels
[I 00:28:56.041 NotebookApp] The Jupyter Notebook is running at:
[I 00:28:56.041 NotebookApp] http://localhost:8888/?token=dfe9e728a4ed920af1f0a9ddcc462752f8b6f747912caa38
[I 00:28:56.041 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[W 00:28:56.041 NotebookApp] No web browser found: could not locate runnable browser.
[C 00:28:56.042 NotebookApp]

    Copy/paste this URL into your browser when you connect for the first time,
    to login with a token:
        http://localhost:8888/?token=dfe9e728a4ed920af1f0a9ddcc462752f8b6f747912caa38
^C[I 00:29:07.489 NotebookApp] interrupted
Serving notebooks from local directory: /home/ec2-user/spark
0 active kernels
The Jupyter Notebook is running at:
http://localhost:8888/?token=dfe9e728a4ed920af1f0a9ddcc462752f8b6f747912caa38
```

`Control-C` 를 두번 입력해서 노트북을 종료한다.

## 6. 노트북 설정 변경하기

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

## 7. 노트북을 백그라운드에서 실행하기

```sh
cd ~/
nohup jupyter notebook &
cat nohup.out
```

## 8. 정상작동을 확인하기

노트북에 아래 코드를 입력하고 실행한다.
에러없이 실행되면, pyspark 가 정상작동함을 확인할 수 있다.

```python
import pyspark
sc = pyspark.SparkContext.getOrCreate()
sc.stop()
```

## 9. AWS 인증키 등록

```sh
$ vi ~/.bashrc
......
export AWS_ACCESS_KEY_ID=액세스키
export AWS_SECRET_ACCESS_KEY=보안액세스키
export AWS_DEFAULT_REGION=ap-northeast-2
......
$ source ~/.bashrc
```

## 10. hadoop 설치하기

```sh
cd ~/
mkdir hadoop
cd hadoop
wget http://mirror.navercorp.com/apache/hadoop/common/hadoop-2.7.6/hadoop-2.7.6.tar.gz
tar xvfz hadoop-2.7.6.tar.gz
rm hadoop-2.7.6.tar.gz
```

환경설정을 수정한다.

```sh
$ vi ~/.bashrc
......
export HADOOP_HOME=/home/ec2-user/hadoop/hadoop-2.7.6
export PATH=$HADOOP_HOME/bin:$PATH
export LD_LIBRARY_PATH=$HADOOP_HOME/lib/native:$LD_LIBRARY_PATH
......
$ source ~/.bashrc
```
