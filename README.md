# spark-with-ec2-s3

이 프로젝트는 아마존 AWS 에 EC2 Spark 클러스터를 생성하는 방법을 설명합니다.
사용되는 툴은 [Flintrock](https://github.com/nchammas/flintrock) 입니다.

## 참조사이트

참조 : [https://alexioannides.com/2016/08/18/building-a-data-science-platform-for-rd-part-2-deploying-spark-on-aws-using-flintrock/](https://alexioannides.com/2016/08/18/building-a-data-science-platform-for-rd-part-2-deploying-spark-on-aws-using-flintrock/)

참조 : [https://linode.com/docs/databases/hadoop/how-to-install-and-set-up-hadoop-cluster/](https://linode.com/docs/databases/hadoop/how-to-install-and-set-up-hadoop-cluster/)

참조 : [https://linode.com/docs/databases/hadoop/install-configure-run-spark-on-top-of-hadoop-yarn-cluster/](https://linode.com/docs/databases/hadoop/install-configure-run-spark-on-top-of-hadoop-yarn-cluster/)

## 클러스터 생성하기

- [로컬서버 구성하기](./setup-local-server.md)

- [Flintrock 설치하기](./install-flintrock.md)

- [Flintrock 상태 확인하기](./03.check-status.txt)

## 파일시스템에 접근하기

- [Spark 클러스터에서 HDFS 접근하기](./04.connect-to-hdfs.txt)

- [Spark 에서 AWS S3 데이타 가져오기](./05.get-file-from-s3.txt)

- [Spark 에서 AWS S3 데이타 가져오기(With Scala)](./get-file-from-s3-with-scala.md)

- [Spark 에서 AWS S3 데이타 가져오기(With Python)](./get-file-from-s3-with-python.md)

## HADOOP YARN 구성하기

- [HADOOP YARN 위에서 스파크 실행하기](./06.run-spark-on-hadoop-yarn.txt)

## Jupyter Scala 설치하기

- [Jupyter Notebook 에서 Scala 실행하기](./run-scala-on-jupyter-notebook.md)