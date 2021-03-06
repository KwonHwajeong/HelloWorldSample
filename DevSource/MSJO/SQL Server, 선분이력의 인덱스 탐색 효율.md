### SQL Server, 선분이력의 인덱스 탐색 효율

아래의 내용은 ‘SQL Server 튜닝 원리와 해법’에서 발췌 요약한 것이다. 인덱스 탐색 효율과 더불어 대부분 업무에서 복합인덱스 설정이 시작일시보다 종료일시가 대부분 앞쪽에 있어야 한다는 것을 알 수 있습니다.
* 정재우, SQL Server 튜닝 원리와 해법, 비투엔컨설팅, 2010, p113-123

##### 점이력과 선분이력
1. 회원 정보에 변경이 발생했을 때 변경이 발생한 시점(時點)을 기준으로 이력을 관리하는 방식을 ‘점이력’ 모델이라고 한다. (회원번호+변경일시)
2. 점이력에 비해 선분이력 모델은 변경이 발생한 시점뿐만 아니라 이력이 종료된 시점까지 관리하는 방식이다. (회원번호+유효시작일시+유효종료일시)

##### 인덱스(복합) 생성 조건
1. (회원번호 + 유효시작일시 + 유효종료일시) : 과거 데이터 조회 시
2. (회원번호 + 유효종료일시 + 유효시작일시) : 최근 데이터 조회 시 : 대부분 적용

##### 쿼리 예
```sql
select top 1 *
from 회원변경이력
where 회원번호 = '001' and
      '20141010' between 적용일자 and 만료일자
order by  만료일자, 적용일자
```

##### 선분이력의 인덱스 탐색 효율
| 검색유형            | 인덱스 컬럼 순서 적용일자+만료일자                                    | 인덱스 컬럼 순서 만료일자+적용일자                                  |
|---------------------|-----------------------------------------------------------------------|---------------------------------------------------------------------|
| 과거 데이터 조회 시 | 유리                                                                  | 불리  단, (Top 1 + order by 만료일자 asc, 적용일자 asc)로 개선 가능 |
| 중간 데이터 조회 시 | 불리  단, (Top 1 + order by 적용일자 desc, 만료일자 desc)로 개선 가능 | 불리  단, (Top 1 + order by 만료일자 asc, 적용일자 asc)로 개선 가능 |
| 최근 데이터 조회 시 | 불리  단, (Top 1 + order by 적용일자 desc, 만료일자 desc)로 개선 가능 | 유리 일반적으로, 이 경우에 맞추는 게 성능상 유리함                  |

##### 인덱스 컬럼 순서
인덱스 컬럼 순서가 (적용일자 + 만료일자)일 때는 ‘order by desc’ 조건을 사용하여 해당 인덱스를 내림차순으로 액세스해야 한다.
