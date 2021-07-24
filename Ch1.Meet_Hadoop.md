# Meet Hadooop

## Table of Contents

[Data](##Data)

[Data Storage](##Data-Storage)

[Querying All Your Data](##Querying-All-Your-Data)

[Beyond Batch](##Beyond-Batch)

<br>

## Data

기술이 발전하면서 많은 양의 데이터가 나날이 쌓이고 있다. 개인이 생성하는 데이터도 많지만, 사물인터넷의 발전에 따라, 기계에서 생산되는 데이터양이 압도적이다.

좋은 소식은 빅 데이터가 생성되면서 보다 정교한 추측이 가능해졌다.

나쁜 소식은 그것을 저장하고, 관리하고, 분석하는 일이 매우매우 어렵다는 점이다.

<br>

## Data Storage

나쁜 소식이 생긴 원인은 간단하다.

지난 수년간 저장소의 용량은 급속도로 증가한 반면, 데이터를 읽는 속도는 별로 증가하지 못했기 때문이다.

    - **병렬 처리**를 통해 한 번에 읽는 데이터 양을 늘릴 수 있다. 대신, 다음과 같은 단점들이 발생한다

        1. H/W 장애

            - H/W가 많을수록, 장애(데이터 손실 등)가 일어날 확률도 늘어난다

                - 데이터 손실을 막기 위해 다수의 백업을 만든다

                - RAID (Redundant Array of Independent Disk)

                - Hadoop의 파일시스템 HDFS (Hadoop Distributed File System)와는 약간 다르다

        2. 데이터 분석 작업을 위해서는 분할된 데이터를 다시 병합해야 한다

            - 맵리듀스를 통해 극복 가능(?)

                - 맵 리듀스 : 맵과 리듀스로 계산이 분리되어 있고, 이를 혼합해주는 인터페이스 존재


Hadoop은 안정적이고 확장성이 높은 저장 및 분석 플랫폼을 제공한다.

특히 Hadoop은 범용 H/W에서 실행되고 오픈 소스로 저렴하다.

<br>

## Querying All Your Data

맵리듀스는 한 번의 쿼리로 전체 혹은 큰 규모의 데이터 셋을 처리한다는 전제로 작동하기 때문에 brute-force방식처럼 보인다.

이것은 맵리듀스의 장점이다.

맵리듀스는 일괄 질의 처리기며, 전체 데이터셋을 대상으로 비정형 쿼리를 수행하고, 합리적인 시간 내에 그 결과를 보여준다.

<br>

## Beyond Batch

맵리듀스는 일괄 처리 시스템이라는 강점을 갖고 있기 때문에 대화형 분석에는 적합하지 않다.

Hadoop은 HDFS와 맵리듀스만을 지칭하지 않고, 수많은 에코시스템 프로젝트를 지칭하고 있다.

Hadoop 에코 시스템은 분산 컴퓨팅과 빅 데이터 처리를 위한 기반 시설이다.

YARN (Yet Another Resource Negotiator)은 Hadoop의 새로운 처리 모델을 위한 것으로, Hadoop2에 포함되어 있다.

YARN은 클러스터 지원 관리 시스템으로, Hadoop 클러스터에 저장된 데이터를 처리할 수 있게 해준다.