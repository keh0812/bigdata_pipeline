# bigdata_pipeline

centos7 에서 VM 3대 구성하여 분석/저장/수집 pipeline 만들기

- NIFI
- KAFKA
- SPARK
- HDFS
- Elasticsearch 
- KIBANA


![image](https://user-images.githubusercontent.com/80734989/149238648-32f3f56d-822d-48c0-8438-34aca4a4301a.png)

![image](https://user-images.githubusercontent.com/80734989/149238855-49119e4f-ceb8-4e14-ac36-ed579c654736.png)

![image](https://user-images.githubusercontent.com/80734989/149238888-5405d5d1-ffd6-4ef5-96f7-303dc7e2ca46.png)

![image](https://user-images.githubusercontent.com/80734989/149238914-3f05eacf-d5ed-49e9-b982-eb67af9c31d5.png)

![image](https://user-images.githubusercontent.com/80734989/149238935-f80b63c3-6452-49eb-9dc4-a5f9376e326f.png)

![image](https://user-images.githubusercontent.com/80734989/149238946-e03049b8-afaa-44cc-be75-fdeb59b9286a.png)

![image](https://user-images.githubusercontent.com/80734989/149238964-9e156069-5a77-45de-88b6-abfaa758fe0e.png)

![image](https://user-images.githubusercontent.com/80734989/149238982-a9ef7bfc-7b6c-463a-a216-e9ae011c6fa9.png)

![image](https://user-images.githubusercontent.com/80734989/149239002-47726910-78e8-46c3-9c2f-582c210d049e.png)

![image](https://user-images.githubusercontent.com/80734989/149239020-2f17388a-b943-4f38-9eed-b23e881ad178.png)

![image](https://user-images.githubusercontent.com/80734989/149239041-ea387f95-cfeb-433f-b515-b23e5670e7ec.png)

![image](https://user-images.githubusercontent.com/80734989/149239059-9438f8c5-3dda-4e36-9fa1-8955d4a24853.png)

![image](https://user-images.githubusercontent.com/80734989/149239082-3d233e44-bfdb-4a5f-9fc0-8e5fedbe98ea.png)

![image](https://user-images.githubusercontent.com/80734989/149239105-4593e1b4-5657-4e83-9ead-550c6b01925c.png)

![image](https://user-images.githubusercontent.com/80734989/149239120-79c52b36-4269-448c-8b00-b6ff9b1210d6.png)



# 세미나 자료

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

## zookeeper

- on : /usr/local/zookeeper/bin/zkServer.sh start
- /usr/local/zookeeper/bin/zkServer.sh status로 follow랑 reader 확인 가능
- port : 2888, 3888
 ZooKeeper는 분산 시스템을 위한 코디네이터이다. 
주키퍼의 역할은 잠금제어(Locking), 공급/구독 (Publisher/ Subscriber), 리더선정, 동기화 등이 될 수 있다.
주키퍼는 클러스터로 구성할 경우 과반수 이상의 서버가 정상일 때만 지속적인 서비스가 가능하기 때문에 홀수로 구성을 해줘야 한다. 그렇기 때문에 서버 2대가 아닌 3대에 설치를 할 것이다.

- 지노드 : 주키퍼 내에 분산 애플리케이션 상태 정보가 저장되는 곳. 분산 애플리케이션들은 각각 클라이언트가 되어 주키퍼 서버들과 연결을 맺은 후 상태 정보를 주고받게 된다. 상태정보는 주키퍼의 지노드(znode)에 Key-Value 형태로 저장되며, 지노드에 저장된 것을 이용하여 분산 애플리케이션들은 서로 데이터를 주고받게 된다.

## 2. NiFi  8090

- NiFi는 데이터를 수집하고 workflow를 관리하는 시스템이다. flowfile, processor, connection으로 이루어져 있고 장점은 실시간 처리에 매우 적합하다는 것이다.
단점은 현재 실행되는 내용을 확인할 수 없는 점이다. 또한 간단한 데이터 조작만 가능하다는 단점이 있지만 spark나 storm과 연동하여 사용해서 보완할 수 있다고 한다.
- 대용량 분산 시스템에서 서로 다른 여러 시스템들 사이를 연결하여 데이터가 직관적이고 신속하며 어떠한 유실 없이 전달되게 하는 DataFlow 엔진
- 데이터 전송에 주목적이 있지만, 다양한 역할 수행 가능
- Zero-Master Clustering : 단일 마스터 노드가 없는 환경으로 Zookeeper가 자동으로 활성화된 노드들 중 하나를 선정하여 Cluster Coordinator라는 이름을 주고 마스터 노드의 역할을 하게하여 장애대응에서 안정성을 가진다.
- 장애가 발생해도 데이터의 손실이 없다 : nifi는 지속적으로 노드끼리 데이터를 공유한다.

NiFi todolist는 데이터에 파일 이름을 추가하여 kafka에 보내는 것이다.
그래서 ListFile과 FetchFile을 사용하여 /sftp/data 안의 모든 txt파일을 가져와서,
ReplaceText로 데이터 첫 줄에 filename을 추가해주었고
publichKafka를 사용하여 kafka topic t3q에 메시지를 보냈다.

##  Kafka : 분산 메시징 시스템
- Broker : kafka 서버이고 zookeeper는 kafka cluster를 구성할 수 있도록 분산 코디네이션 시스템 역할을 한다.
- Kafka cluster는 zookeeper와 kafka broker로 이루어져있다.
- Partition : 병렬처리가 가능하도록 토픽을 나눌 수 있고, 많은 양의 메시지 처리를 위해 파티션의 수를 늘려줄 수 있다.
- Zookeeper : 분산 애플리케이션을 위한 코디네이션 시스템. 분산 애플리케이션이 안정적인 서비스를 할 수 있도록 분산되어 있는 각 애플리케이션의 정보를 중앙에 집중한다. 컨슈머 혹은 카프카와 직접 통신하면서 구성 관리, 그룹 관리 네이밍, 동기화 등의 서비스를 제공한다.
- Log : producer가 생성한 메시지


##  E/S : 검색 엔진
- Master Node : 전체 Cluster의 상태에 대한 Meta 정보를 관리하는 Node, 기존 Master Node가 종료되면 새로운 Master Node가 선출
- Data Node : 색인된 데이터를 실제로 저장하는 Node
- Master Node도 아니고 Data Node도 아닌 Node 존재. 색인과 검색을 위한 명령과 결과를 전달하는 역할로만 존재
- indexing : 데이터를 검색될 수 있는 구조로 변경하기 위해 원본 문서를 검색어 토큰들로 변환하여 저장하는 일련의 과정 = 색인
- index : 색인 과정을 거친 결과물, 색인된 데이터가 저장되는 저장소
- Kibana : E/s와 연동되는 시각화 도구
- 
5. HDFS : Hadoop 내부에 구성되어 있는 분산 파일 시스템
- 분산 저장으로 복제가 가능 -> 특정 노드 장애에 무정지 대응
- HDFS는 하나의 네임 노드와 다수의 데이터 노드로 구성
- Name Node와 Data Node는 마스터-슬레이브 구조를 이룬다.
- HA cluster란 두 개의 여분의 name node를 실행하는 옵션 -> Active-Standby Name 
- (1) NameNode, (2) Secondary NameNode(보조 네임노드), (3) DataNode

- (1) NameNode : 메타데이터 관리, 
- (2) Secondary NameNode : NameNode 장애시 데이터 복구를 위한 노드
- (3) DataNode : slave server, 주기적으로 NameNode에게 Heartbeat과 블록의 목록 리포트 보냄
- (4) JournalNode : 


## Spark
- 빅데이터 분산 처리 엔진, 인메모리 기법
- 하둡 기반 맵리듀스 작업이 가진 단점을 보완하기 위해서 만들어 진 프레임워크
- 하둡과 달리 인메모리 기법을 활용한 데이터 저장 방식을 제공함으로써 반복적인 데이터 처리가 필요한 분야에서 높은 성능을 보여준다
- 반복적인 처리가 필요한 작업에서 속도가 하둡보다 최소 1000배 이상 빠르다.이를 통해 데이터 실시간 스트리밍 처리라는 니즈를 충족함으로써, 
- 빅데이터 프레임워크 시장을 빠르게 잠식해가고 있다.
- 이점
(1) 속도
(2) 편의성
(3) 보편성
(4) 오픈소스


1. Elasticsearch Index

- index 생성

curl –XPUT '192.168.81.101:9200/index명?pretty’

- index 목록 확인

curl –XGET '192.168.81.101:9200/_cat/indices?v&pretty’

- 클러스터 상태 확인

curl –XGET '192.168.81.101:9200/_cat/health?v&pretty’

: 상태확인 메시지를 통해서 status 항목에 green, yellow, red라는 정보를 볼 수 있다.
  . green은 모든 기능이 정상적으로 동작
  . yellow는 전체적인 기능은 수행하고 있으나 일부 복제본이 아직 할당되지 않은 상태
  . red는 어떤 이유로 인하여 데이터를 사용할 수 없는 상태
  
샤드의 개수는 인덱스를 처음 생성할 때 지정할 수 있다. 
프라이머리 샤드 수는 인덱스를 처음 생성할 때 지정하며, 인덱스를 재색인 하지 않는 이상 바꿀 수 없다. 복제본의 개수는 나중에 변경이 가능하다. 

- shard : 인덱스는 기본적으로 샤드라는 단위로 분리 / 분산 저장소
 primary shard : 원본 shard / 기본 shard
- replica : 복제본 / Shard의 복제본
- index : 도큐먼트의 집합 단위 / Table
- 데이터를 Elasticsearch에 저장하는 행위는 색인, 그리고 도큐먼트의 집합 단위는 인덱스


1. NiFi에서 kafka data를 json 형식으로 변환

- replaceText 1 에서,
search Value(검색 값) : “
replace Value(교체 값) : Empty string set 

- replaceText 2 에서,
search Value(검색 값) : (?s)(^.*$)
replace Value(교체 값) : {"${filename}":"$1"}

2. E/S index 생성 (replica 3개)
kibana에서 생성 :　http://192.168.81.101:5601/
(1) http://192.168.81.101:5601/ 접속 후
(2) Index Management 클릭
(3) id, body, subject, writedatetime, fullpathname, filename 생성
- 인덱스 다시 생성해야하면 다 삭제하고 kibana에서 처음부터 생성

3. Spark에서 HDFS 원본 저장
(1) http://192.168.81.101:8890/ 접속 후 jupyter notebook 사용

kafka data 꺼내기
hdfs 파일 저장 방법







