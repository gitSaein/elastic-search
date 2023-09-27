# elastic-search

---------------------------
- 설명
  - Elasticsearch는 문장을 작은 단위로 분할하고, 분할된 결과를 가지고 색인하고 검색하는 과정에서 테스트를 처리(필터를 총해 대소문자 통일 or 불필요한 결과 제거) 한다.
  - Elasticsearch는 토크나이저(tokenizer)를 이용하여 문서를 작은 단위로 분할하는데 이렇게 분할된 텍스트를 '토큰' 이라한다.
  - Elasticsearch는 분석기를 이용해서 토크나이저를 정하고 필터를 이용해서 분석된 데이터를 처리한다.
    - 한글 형태소 분석기: Nori 확장 플러그인

- ELK(Elasticsearch & Logstash & Kibana) 스택 
  - Filebeat: 파일에 저장된 로그 파일 수집
  - Logstash: 정제 및 전처리
  - ElasticSearch: 저장, 검색, 집계
      1. 활용: 
      2. 구성
         1. 포트: 노드-클라이언트 통신 (9200~9299)/ 노드-노드 데이터 교환 (9300~9399)
  - Kibana: 모니터링 시각화


----------------------------

1. docker-compose install
```
 brew install docker
 brew install docker-compose
```
2. elastic & kibana install
```
docker-compose -f ./docker-compose.yaml up
```
3. nori plugin install
```
docker exec -it {container name} /bin/sh
bin/elasticsearch-plugin install analysis-nori

docker-compose stop
docker-compose up
```
4. GET 형태소 분석 

```
GET _analyze
{
  "analyzer": "standard",
  "text": "오늘의 날씨는 맑음입니다."
}
-->
{
  "tokens" : [
    {
      "token" : "오늘의",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "<HANGUL>",
      "position" : 0
    },
    {
      "token" : "날씨는",
      "start_offset" : 4,
      "end_offset" : 7,
      "type" : "<HANGUL>",
      "position" : 1
    },
    {
      "token" : "맑음입니다",
      "start_offset" : 8,
      "end_offset" : 13,
      "type" : "<HANGUL>",
      "position" : 2
    }
  ]
}
----------------------------------------
GET _analyze
{
  "analyzer": "nori",
  "text": "오늘의 날씨는 맑음입니다."
}
-->
{
  "tokens" : [
    {
      "token" : "오늘",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "날씨",
      "start_offset" : 4,
      "end_offset" : 6,
      "type" : "word",
      "position" : 2
    },
    {
      "token" : "맑",
      "start_offset" : 8,
      "end_offset" : 9,
      "type" : "word",
      "position" : 4
    },
    {
      "token" : "이",
      "start_offset" : 10,
      "end_offset" : 13,
      "type" : "word",
      "position" : 6
    }
  ]
}
```

5. PUT 인덱스 생성 및 설정

```
PUT nori_sample
{
  "settings": {
    "index": {
      "analysis": {
        "tokenizer": {
          "my_nori_tokenizer": {
            "type": "nori_tokenizer",
            "decompoound_mode": "mixed",
            "discard_punctuation": "false"
          }
        },
        "analyzer": {
          "my_nori_analyzer":{
            "type": "custom",
            "tokenizer": "my_nori_tokenizer",
            "filter": ["lowercase", "stop"],
            "char_filter": ["html_strip"]
          }
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title":{
        "type": "text"
        , "analyzer": "my_nori_analyzer"
      }
    }
  }
}
```

6. POST 데이터 삽입
```
POST nori_sample/_doc
{
  "title": "안녕하세요. 2023-09-27 오늘의 날씨는 구름이 많습니다. 비가 오는 곳도 있겠습니다."
}


POST nori_sample/_doc
{
  "title": "안녕하세요. 2023-09-26 오늘의 날씨는 맑음입니다."
}
```
7. GET 데이터 조회 (nori tokenizer)
```
GET nori_sample/_search
{
  "query": {
    "term": {
      "title": "맑"
    }
  }
}
```
- 참조
  - [nori tokenizer](https://www.elastic.co/guide/en/elasticsearch/plugins/current/analysis-nori-tokenizer.html)
