---
title: "Amazon S3 Transfer Acceleration"
description: "Amazon S3 Transfer Acceleration 정리"
date: 2026-03-23
update: 2026-03-23
tags:
  - AWS
series: "AWS SAA-C03"
---

<br>

#### 한줄요약
- 전 세계 클라이언트가 먼 S3 버킷에 대용량 데이터를 빠르게 업로드하도록 돕는 S3 기능

<br>


#### What is it?

- **전 세계 어디서든** 멀리 떨어진 **중앙 S3 버킷**으로 더 **빠르게 업로드**할 수 있게 해주는 **S3 버킷 기능**
- 사용자는 가까운 **CloudFront 엣지 로케이션**에 먼저 업로드하고, 이후 **AWS 글로벌 백본 네트워크**를 통해 대상 S3 버킷으로 전달됨
- **장거리 인터넷 구간**의 비효율을 줄여 업로드 성능을 개선함

<br>

#### When use it?

- **여러 국가·대륙**의 사용자 또는 지사에서 **하나의 S3 버킷**으로 데이터를 **빠르게** 모아야 할 때
- **GB~TB 규모의 대용량 데이터**를 장거리로 업로드할 때
- **공용 인터넷**만으로는 장거리 전송 속도와 안정성이 떨어질 수 있을 때
- **운영 복잡도 증가 없이** S3 업로드 성능을 개선하고 싶을 때

<br>

#### Key points for SAA

- **글로벌 업로드 가속**이 필요하면 가장 먼저 떠올릴 것
- **단일 S3 버킷으로 직접 업로드**하는 구조에 적합
- 별도 서버나 복잡한 중계 아키텍처 없이 사용 가능
- **멀티파트 업로드와 함께** 쓰면 대용량 파일 업로드에 더 유리함

<br>

#### 참고자료

[https://docs.aws.amazon.com/AmazonS3/latest/userguide/transfer-acceleration.html](https://docs.aws.amazon.com/AmazonS3/latest/userguide/transfer-acceleration.html?utm_source=chatgpt.com)