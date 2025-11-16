## SQL 튜닝
> SQL 전문가 가이드를 참고해 작성하였습니다. 

### 인덱스 튜닝
####  CASE 1: 인덱스 칼럼을 가공하지 않는다. (p563)
[예시]
```sql
select * from employees 
         where substr(registeration_number, 1, 6) = '990101';
```
[튜닝 방안]
```sql
select * from employees 
         where registeration_number like '990101%';
```
#### CASE 2: 부정형 비교를 사용하지 않는다. (p563)
!=, <>, not 연산은 아닌 것을 찾기 때문에 인덱스 풀스캔 또는 테이블 풀 스캔이 발생한다. 

[예시]
```sql
select * from employees 
         where status <> 'A';
```
[튜닝 방안] 가능하면 비교로 바꾼다. 
```sql
select * from employees 
         where status in ('B', 'C', 'D');
```
#### CASE 3: 테이블 랜덤 액세스를 줄인다. (p572)
[예시]

인덱스: employees_index_01(department_id, job)
```sql
select * from employees 
         where department_id = 10 and salary > 3000;
```
[튜닝 방안] salary 칼럼을 인덱스에 추가한다. (department_id, job, salary)
```sql
alter index employees_index_01 add salary;
```
#### CASE 4: 인덱스 선행 칼럼이 범위조건이면 In List 또는 Union All을 사용한다. (p578)
인덱스 선행 칼럼이 범위 조건이면, 그 다음 조건을 만족하지 않는 칼럼까지 스캔하는 비효율이 생긴다. 

**범위 조건이 1개일 때**

[예시] 인덱스 employees_index_02(department_id, job)
```sql
select * from employees 
         where department_id between 10 and 30 
           and job = 'CLERK';
```
[튜닝 방안] 범위 조건을 In List로 바꾼다. 
```sql
select * from employees 
         where department_id in (10, 20, 30) 
           and job = 'CLERK';
```

**범위 조건이 2개 이상일 때**

[예시] 인덱스 employees_index_03(department_id, job, name), job 파라미터는 없을 수 있다.
```sql
select * from employees 
         where department_id = :department_id
           and job like :job + '%'
           and name like :name + '%';
```
[튜닝 방안] union all로 분리해 상단 쿼리의 인덱스 스캔 범위를 줄인다. 하단 쿼리를 위해 deparetment_id, name으로 구성된 인덱스를 추가한다. 
```sql
select * from employees
         where department_id = :department_id
           and :job is not null
           and job = :job
           and name like :name + '%'
union all
select * from employees
         where department_id = :department_id
           and :job is null
           and name like :name + '%'
 ```

### 조인 튜닝
#### CASE 5: 1:M 조인 결과를 1에 맞춰 그룹핑해야 된다면, 인라인 뷰를 활용한다. (p.610)
#### CASE 6: 배타적 관계에 있는 테이블이면 union all 대신 outer join을 활용한다. (p613)
#### CASE 7:  조건절 이행을 이용해 조인 데이터 양을 줄인다. (p663)
#### CASE 8: OR 조건절 대신 union all을 활용한다. (p665)
### 기타 튜닝
#### CASE 9: UNION 대신 UNION ALL을 활용한다. (p684)
#### CASE 10: Distinct 대신 Exists를 활용한다. (p686)
#### CASE 11: 데이터 존재여부만 확인한다면 count 대신 rownum을 활용한다. (p688)
#### CASE 12: 소트 연산은 인덱스를 활용하게 한다. (p690)
#### CASE 13: 소트 영역을 최대한 줄인다. TOP N 쿼리 & 정렬 후 데이터 붙이기 (p692)
#### CASE 14: 서브쿼리 대신 CASE 문을 사용한다. (p754)
#### CASE 15: With 구문을 사용해, 테이블 접근 횟수를 줄인다. (p766)
