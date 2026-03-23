---
title: "Amazon Athena"
description: "Amazon Athena 정리"
date: 2026-03-23
update: 2026-03-23
tags:
  - AWS
series: "AWS SAA-C03"
---

<br>

#### 한줄 요약
- S3 데이터에 쿼리하는 서버리스 서비스 

<br>

#### What is it?

- 표준 SQL을 사용해서 Amazon S3에 있는 데이터를 쉽게 분석할 수 있는 대화형 쿼리 서비스
- 서버리스 서비스로, 관리할 인프라가 없고 실행한 쿼리에 대해서 비용 지불.
- 자동으로 확장되며, 쿼리를 병렬 실행하여 빠르게 결과를 반환
- S3 데이터를 in-place 조회하므로, 별도 적재 없이 ad hoc 쿼리에 적합
- AWS Glue Data Catalog, 외부 Hive metastore, federated query와 연계해 메타데이터를 관리할 수 있음

<br>


#### When use it?

- S3에 저장된 정형/반정형/비정형 데이터 분석 (CSV, JSON, Parquet 등)
- S3 데이터에 대해 간단한 ad hoc SQL 쿼리를 빠르게 실행하고 싶을 때
- 인프라나 클러스터를 직접 구축/운영하고 싶지 않을 때
- 로그 분석, 리포팅, 탐색적 주문처럼 주문형 조회가 필요할 때
- S3 Inventory, AWS Cost and Usage Report, 각종 AWS 서비스 로그를 SQL로 조회하고 싶을 때

<br>

#### Key points for SAA

- S3에 있는 데이터를 최소 변경, 최소 운영으로 SQL 조회 → Athena
- S3 + SQL + 서버리스 + ad hoc 쿼리 + 운영 오버헤드 최소화 → Athena
- 이미 데이터가 S3에 저장되어 있다면 Athena를 가장 먼저 떠올리기
- 보통 Glue Data Catalog와 함께 출제됨

<br>

#### 참고자료

https://docs.aws.amazon.com/athena/latest/ug/what-is.html
https://docs.aws.amazon.com/athena/latest/ug/when-should-i-use-ate.html
https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-inventory-athena-query.html