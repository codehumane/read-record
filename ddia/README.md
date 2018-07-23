# Designing Data-Intensive Application

## 저장소와 검색

### 데이터베이스를 강력하게 만드는 데이터 구조

```bash
#!/bin/bash

db_set () {
    echo "$1,$2" >> database
}

db_get () {
    grep "^$1," database | sed -e "s/^$1,//" | tail -n 1
}
```

- 아주 간단한 데이터베이스.
- 매 라인마다 쉼표로 구성된 키-값 쌍의 텍스트 파일을 활용.
- 기존 값을 갱신하지 않고 계속 쌓기만 하는 append-only.
- 데이터 베이스의 로그가 바로 이런 형태.
- 로그가 너무 커지지 않게 디스크 공간 회수나 오류 처리, 동시성 제어 등의 문제 해결이 필요하긴 하지만, 기본적으로는 같은 원리.
- 하지만, 레코드가 많아지면 검색 성능이 나빠짐. 특정 키를 찾기 위해, 처음부터 끝까지 스캔해야 하기 때문. O(n).
- 키 찾기의 효율성을 위해 활용되는 것이 바로 색인.
- 하지만 쓰기 속도는 느려짐.
- 이런 트레이드 오프로 인해 데이터베이스는 모든 것을 색인하지 않음. 사용자의 선택.

#### 해시 색인

키-값 저장소를 해시 맵으로 색인하기.

- [해시 맵 인덱스 그림](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0301.png) 참고.
- 키를 데이터 파일의 바이트 오프셋에 매핑해 인메모리 해시 맵을 유지.
- 단순하지만 Bitcask 등 실제로 많이 사용됨.
- 고유키가 많지 않으면서도(메모리에 적재해야 하므로) 값의 갱신이 빈번한 경우에 유리.

만약, 디스크 상의 로그 구조화 파일에 계속 append만 되서 공간이 부족해 진다면?

- 로그를 특정 크기의 세그먼트<sup>segment</sup>로 나누기.
- 세그먼트가 특정 크기에 도달하면, 세그먼트를 닫고 이후의 쓰기는 새로운 세그먼트에 수행.
- 닫힌 세그먼트에 대해서는 일반적인 컴팩션<sup>compaction</sup> 수행.
- 당연히, 여러 개의 세그먼트에 대해서도 컴팩션 가능.

구현 시 주의 사항들

- CSV 형식은 로그에 부적합. 문자열 길이를 바이트 단위로 인코딩 후, 원시 문자열을 인코딩 하는 것이 빠르고 간편함. 이스케이핑도 필요 없어짐.
- 키 관련 값을 삭제하려면 특수한 삭제 레코드(tombstone이라고도 불림)을 추가. 세그먼트 병합 시 이 키의 이전 값들은 무시됨.

추가 전용 로그의 장단점

- 순차적 쓰기 작업으로, 무작위 쓰기에 비해 훨씬 빠름.
- 동시성과 고장 복구에 유리. 고장의 경우 별도의 파일에 추가한 뒤 병합하면 됨.
- 반면, 해시맵은 메모리에 저장해야 빠르므로, 너무 큰 색인은 불가.
- 범위 질의가 비효율적. 해시 맵에서 일일이 모든 개별 키를 조회해야 함.

### B 트리

- SS(Sorted String)테이블과 같이 정렬된 키-값 쌍을 유지.
- 따라서, 키-값 검색과 범위 질의에 효율적.
- 가변적 단위인 세그먼트 아니라, (일반적으로) 4KB라는 고정 크기의 블록이나 페이지 단위로 나눔.
- 한 페이지에서 참조하는 하위 페이지의 수를 가리켜 분기 계수<sup>branching factor</sup>라고 함. 보통 수백 개에 달함.
- 키 값의 갱신은 [여기 그림](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0307.png) 참고.

#### 신뢰할 수 있는 B 트리 만들기

- 로그 구조화 색인과 대조적으로 새로운 데이터를 기존 페이지에 덮어 씀.
- 일부 동작은 여러 다양한 페이지의 덮어쓰기가 수반됨. 고아 페이지에 유의.
- 고장 복구를 위해 디스크 상의 쓰기 전 로그<sup>write-ahead log</sup>를 쌓음. 재실행 로그<sup>redo log</sup>라고도 불림.
- 다중 스레드가 동일한 페이지 갱신에 유의. 보통 레치<sup>latch</sup> 즉, 가벼운 잠금으로 이를 보호함.

#### B 트리 최적화

- 페이지를 오버라이트하거나 WAL을 유지하는 대신, 일부 데이터베이스들은 copy-on-write 스키마를 사용. 동시성 제어에 유리. 일종의 READ-COMMITTED.
- 키 전체 대신 축약 버전을 저장. B+ 트리를 일컫는 것.
- 리프 페이지들은 디스크 상에서 연속된 순서로 나타나게끔 배치를 시도.
- 트리에 포인터 추가. 리프 페이지가 양쪽 형제들에 대한 참조를 가지면, 상위 페이지로 가지 않아도 순서대로 키 스캔이 가능.


