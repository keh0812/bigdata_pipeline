## centos7 에서 VM 3대 구성하여 분석/저장/수집 pipeline 만들기

- NIFI
- KAFKA
- SPARK
- HDFS
- Elasticsearch 
- KIBANA


![image](https://user-images.githubusercontent.com/80734989/149238648-32f3f56d-822d-48c0-8438-34aca4a4301a.png)

![image](https://user-images.githubusercontent.com/80734989/149238855-49119e4f-ceb8-4e14-ac36-ed579c654736.png)

![image](https://user-images.githubusercontent.com/80734989/149238888-5405d5d1-ffd6-4ef5-96f7-303dc7e2ca46.png)

###  port 정보
- kibana 5601
- e/s 9200
- zoo 2181
- kafka 9092
- nifi 8090
- spark 8081
- 2888, 3888은 zookeeper server간 통신
- 2181은 zookeeper client

###  아키텍처 설명
t3qai01에 sftp서버에서 IP 192.168.0.174 특정 경로의 모든 txt 파일을 가지고 왔습니다. 그 데이터를 NiFi에서 가공하고 Kafka topic에 메시지로 전달합니다.
spark 분산 처리 엔진을 통해 kafka의 데이터를 꺼내, 파일이름을 할당하고 데이터를 파싱 후 HDFS에 원본파일이름으로 파일 저장,
E/S 인덱스 생성 id, fileName, fullPathName, subject, body, writeDatetime 으로 필드 구성하는 것이 전체 아키텍처입니다.

### vm구성
vm은 3개로 구성하였습니다.
