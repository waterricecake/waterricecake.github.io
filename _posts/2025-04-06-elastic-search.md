---
layout: post
title: "[CS] Elastic search 개념"
categories:
  - 학습
tags:
  - Elasticsearch
comments: true
---

# 0. 서론

ElasticSearch에 대한 공부를 시작하기로 하였습니다.

해당 내용은 [# Elasticsearch: EP1 - Elasticsearch (미쿡엔지니어)](https://www.youtube.com/watch?v=XZQP7fMFHMM)의 내용을 따라갔습니다.

# 1. ElasticSearch란

> Elasticsearch는 Apache Lucene 라이브러리를 기반으로 하는 오픈 소스 검색 및 분석 엔진입니다.
> [ibm-Elasticsearch란 무엇인가요](https://www.ibm.com/kr-ko/topics/elasticsearch)

몇년 전까지는 **ElasticSearch, Logstash, Kibana**를 합쳐서 **ELK 스택**을 많이 사용하였습니다. 최근에는 **Beats**등 여러 기술 스택을 합치며 **Elastic Stack**이라고 부르게 되었습니다.

# 2. 특징

- Distributed Nature
- Full-Text Search
- Scalability
- Flexibility

## 2.1 Distributed Nature

> Elasticsearch automatically distributes data and query load across all available nodes in a cluster. It can handle large volumes of data in near real-time.

Elasticsearch는 자동적으로 클러스터 내의 가능한 노드들의 데이터와 쿼리를 분산시킵니다. 이는 대규모 데이터를 거의 실시간으로 다룰 수 있게 해줍니다.

## 2.2. Full-Text Search

> Elasticsearch is built to perform advanced full-text search capablitited with an HTTP web interface and schema-free JSON documents

 Elascticsearch는 Full-Text search 가능하게 합니다.
## 2.3. Scalability

> It scales out to hundreds (or even thousands) of servers and handles petabytes of stuctured and unstructured data.

몇백 - 몇천개의 서버를 수평적으로 늘릴 수 있어 대용량 데이터를 다룰 수 있게 해줍니다.
## 2.4. Flexibility

> Elasticsearch allows for the indexing of heterogeneous data types from various sources with complex search

다양한 종류의 데이타 타입을 여러개의 소스에서 사용할 수 있습니다.

# 3. Document & Field

 ElasticSearch를 이루는 단위 (unit)들이 있는데 그중 가장 작은 단위는 Document와 Field입니다.

```json
{
	"name" : "dongyuen kim", // Filed
	"age" : 31,
	"email": "123@123.com"
} // Document
```

## 3.1. Document

> A document in Elasticsearch is a basic unit of information that can be indexed. Each document is expressed in JSON, which is a lightweight data-interchange format.

index이 가능한 가장 기본적인 단위입니다. 각 document는 JSON 타입으로 이루어져 있습니다.

## 3.2. Field

> Fields are the smallest units of data in Elasticsearch and reperesent the key-value pairs in a document.

# 4. Use Cases

## 4.1. Enterprise Search

> Used by businesses to index their entire digital content and enable sophisticated search capabilities across their websites or internal networks.

회사 내에서 비지니스 용도로 사용됩니다

## 4.2 Logging and Log Analysis

> In conjunction with Logstash and Kibana, Elasticsearch is used for log analysis, providing insights into log data for IT operations, security, and performance monitoring.

Logstash (데이타 파이프라인의 일종)과 Kibana(UI  대쉬보드) 를 통해 거의 실시간으로 로그를 확인할 수 있습니다.

## 4.3. Security Information and Event Management

> Organizations use Elasticsearch to analyze and visualize their security data in real-time, helping in threat detection and compliance mangement.

Log를 사용하서 Security form을 실시간으로 확인할 수 있습니다.

## 4.4. Data Analysis

> Elasticsearch is utilized for big data analytic, allowing users to explore large amounts of data quickly and in various complex ways.

 Elasticsearch를 통해 빠르게 Data 분석으 할 수 있습니다.

## 4.5. Personalization and Recommendations

> E-commerse websites use Elasticsearch to analyze user behaviors and interactions to provide personalized product recommendations and dynaic content.

간단한 Personalization이나 Recommendation은 Elasticsearch를 사용할 수 있습니다.

Personalization란 개인의 특성에 맞게 서비스나 제품을 노출하는 행위를 말합니다.

Recommendation 또한 추천을 하는 행위를 뜻합니다.

# 5. How does Elasticsearch work?


## 5.1 Data Distribution and Storage

### 5.1.1 Indexing

> Data in Elasticsearch is organized into indices, and each index can be thoughts of as a database. The documents stored in these indices are expressed in JSON format. When a document is added to an index, Elasticsearch indexes it by converting the data into a format that is optimized for search.

Elasticsearch의 데이터드는 인덱싱되며, 각 인덱스들은 데이터베이스로 생각해도 무방합니다. 이 인덱스에 저장된 documents들은 JSON 포멧으로 표현되어 집니다.  documents가 index에 저장되어 질때 Elasticsearch는 검색에 최적화된 데이터 포멧으로 바꾸어 줍니다.

### 5.1.2. Sharding

> Each index can be divided into multiple shards, allowing the index to be distributed across several nodes in a cluster, facilitating high availability and performance scalability. Sharding is critical for handling large datasets and high query volumes.

각 인덱스는 여러개의 shards로 나뉠수 있으며, 한 클러스터에서 여러개의 노드로 인덱스를 나눌 수 있도록 해줍니다. 이는 고가용성과 확장성을 보장해 줍니다. 이러한 특징때문에 대용량 데이터와 쿼리에서 Sharding은 필수입니다.

### 5.1.3. Replicas

> Elasticsearch can also create replica shards, which are copies of primary shards. Replicas provide data redundancy, which protects against hardware failure and increases query capacity as searches can be performed on any replica.

Elasticsearch는 shards의 replica를 만들어 하드웨어적인 실패를 방지하며, 쿼리 수행 능력을 상향시킵니다.

## 5.2. Search Mechanisms

### 5.2.1. Query Processing

> When a query is submitted, it is parsed and transformed into a format suitable for searching the Lucene index. The query is then executed against all relevant shards (primary or replica) in parallel.

쿼리가 요청되면, 해당 쿼리를 Lucene index(서치에 사용되는 index)로 변환됩니다. 그 이후 쿼리는 관련된 샤드를 통해서 수행됩니다.

### 5.2.2. Relevance Scoring

>  Elasticsearch uses a variey of relevance scoring algorithms to determine how well each document matches a query. This scoring helps in ranking the search results accroding to their relevance.

Elastsicsearch는 받은 쿼리를 scoring algorithm을 통해 가장 잘 맞는 documents를 점수를 내어 찾아냅니다.

해당 알고리즘은 TF-IDF(Term Frequency-Inverce Frequency)나 BM26(Best Matching 25)등을 사용하는데 이러한 알고리즘은 나중에 직접 알아보는것이 좋을것 같습니다.

### 5.3. Real-Time Operations

### 5.3.1. Near Real-Time(NRT) Search

> Elasticsearch is known for its NRT capabilites, which means that it can retreve and index data almost simutaneously. This is achieved through the use of an in-memory buffer to collect incoming documnets and periodially flushing this buffer to create new segments of the index.

실시간으로 검색을 할 수 있습니다.

## 5.4.  Cluster Management

### 5.4.1. Nodes and Clusters

> An Elasticsearch cluster is a collection of one or more nodes that together hold the entire data and provide indexing and search capabilities across all nodes. A cluster is identified by a unique name which by default is "elasticsearch".

Elastaticsearch cluster는 모든 데이터와 인덱싱 그리고 노드간의 검색을 가지고 있는 모든 Nodes의 집합입니다. cluster는 고유한 이름을 가지고 있습니다.

### 5.4.2. Master Node

> The Master node manages their cluster's health and metadata. It tracks the addition of new documents and the allocation of shards to nodes.

Master node는 클러스터의 메타데이터와 상태를 관리합니다.
### 5.4.3. Node Communication

> Nodes communicate with each other via a RESTful API using JSON over HTTP or using internal Elasticsearch transport protocol.

노드간의 통신은 JSON데이터를 통한 HTTP RESTful API로 통신하거나 Elaststicsearch 내부 전송 프로토콜을 사용합니다.

## 5.5 Analysis and Aggregation

### 5.5.1. Analysis

> Elasticsearch perfoms text analysis during indexing(breaking down text into tokens or terms). It's highly customizable with features such as custom tokenizaers, filters, and analyzers.

Elasticsearch는 텍스트를 토큰 단위로 나누어 인덱싱해 분석합니다. 이 행위는 커스텀 할 수 있습니다.

### 5.5.2. Aggregation

> This feature enables users to obtain complex summaries and insights from their data. Aggregtions can be metrics (calculationg sums, averages, etc.), bucketing (grouping data), or even nested aggregations to explore data hierarchically.

이 기능은 사용자에게 데이터에 대한 요약과 인사이트를 제공할 수 있습니다.

# 6. Elastic Stack

- Elasticsearch : Store, Searching, Analyzing
- Logstash : Server-side data processing
- Kibana : Data visualization dashboard
- Beats : Data shipper
- X-Pack : Security, Alerting, Monitoring, Reporting, Machine Learning(Anomalies), Graph Analytics, SQL

| Logs     | Data Colllection | Data buffer  | Data Processing | Storage       | Visualize |
| -------- | ---------------- | ------------ | --------------- | ------------- | --------- |
| Log data | Beats            | Redis, Kafka | Logstash        | Elasticsearch | Kibana    |

이러한 데이터가 점점 많아질 경우 Data Collection과 Data Processing 사이에 Redis나 Kafka를 Buffer로서 사용하기도 합니다.

# 참고

[elastic](https://www.elastic.co/elasticsearch)

[ibm](https://www.ibm.com/kr-ko/topics/elasticsearch)

 [# Elasticsearch: EP1 - Elasticsearch (미쿡엔지니어)](https://www.youtube.com/watch?v=XZQP7fMFHMM)