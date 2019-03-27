# 6.1.2 특정한 조건의 데이터만 조회하는 SELECT ... FROM ... WHERE

## 데이터 추가
```
INSERT INTO userTbl values('KNG', '김남길', 1981, '서울', '012', '2222222', 184, '2003-03-01');
INSERT INTO userTbl values('KSG', '김성균', 1980, '서울', '013', '3333333', 180, '2004-04-02');
INSERT INTO buyTbl VALUES(NULL, 'KNG', '운동화', NULL, 30, 3);
INSERT INTO buyTbl VALUES(NULL, 'KNG', '모니터', '전자', 200, 6);
INSERT INTO buyTbl VALUES(NULL, 'KSG', '메모리', '전자', 80, 4);
```

## 기본적인 WHERE절(p.198)
```
SELECT 필드이름 FROM 테이블이름 WHERE 조건식;
```

## 비교 연산자/함수
| 이름 | 설명 |
|:--|:--|
| = | 동등 비교 |
| <=> | NULL-safe 동등 비교 |
| !=, <> | 비동등 비교 |
| >, >=, <, <= | 대소 비교 |
| IS | TRUE, FALSE 확인 |
| IS NOT | TRUE, FALSE 아닌지 확인 |
| IS NULL | NULL 인지 확인 |    
| IS NOT NULL | NULL 아닌지 확인 |
| ISNULL() | 인자가 NULL 인지 확인 |
| BETWEEN ... AND ... | 범위 내의 값인지 확인 |
| NOT BETWEEN ... AND | 범위 내의 값이 아닌지 확인 |
| COALESCE()  | NULL이 아닌 첫번째 인자 반환 |
| GREATEST() | 가장 큰 인자 반환 |
| LEAST() | 가장 작은 인자 반환 |
| INTERVAL() | 첫번째 인자보다 큰 인자의 index 반환 |
| IN() | 괄호 안의 인자 중에 있는지 확인 |
| NOT IN() | 괄호 안의 인자 중에 없는지 확인 |
| LIKE | 단순한 패턴 매칭 |
| NOT LIKE | 단순한 패턴 매칭의 반대 |
| STRCMP() | 두 문자열 비교(같으면 0, 앞이 크면 1, 뒤가 크면 -1 반환) |

## 관계 연산자의 사용(p.199)
```
SELECT userID, name 
FROM userTbl 
WHERE birthYear >= 1970 AND height >= 182;
-- 1970년 이후에 출생하고 키가 182 이상인 사람의 아이디와 이름 조회

SELECT userID, name
FROM userTbl 
WHERE birthYear >= 1970 OR height >= 182;
-- 1970년 이후에 출생했거나 키가 182 이상인 사람의 아이디와 이름 조회
```

## BETWEEN ... AND와 IN() 그리고 LIKE(p.199)
```
SELECT name, height 
FROM userTbl 
WHERE height >= 180 AND height <= 183;

SELECT name, height
FROM userTbl
WHERE height BETWEEN 180 AND 183;
-- 키가 180~183인 사람

SELECT name, addr
FROM userTbl
WHERE addr = '경남' OR addr = '전남' OR addr = '경북';

SELECT name, addr
FROM userTbl
WHERE addr IN ('경남', '전남', '경북');
-- 지역이 '경남', '전남', '경북' 인 사람

SELECT name, height
FROM userTbl
WHERE name LIKE '김%';
-- 성이 '김'씨이고 그 뒤는 무엇이든(%) 허용, '김'이 제일 앞 글자인 것들을 추출

SELECT name, height 
FROM userTbl 
WHERE name LIKE '_종신';
-- 한 글자와 매치하기 위해서 '_' 사용
-- '_용%' -> '조용필', '사용한 사람', '이용해 줘서 감사합니다'
```
- 와일드 카드
    - %, _ 문자열의 제일 앞에 들어가면 성능 악영향
    - 인덱스가 있더라도 사용하지 않고 전체 데이터 검색

### Q1. 결과 예상해보기
```
1) =, <=>
SELECT 1 = 1, NULL = NULL, 1 = NULL;
SELECT 1 <=> 1, NULL <=> NULL, 1 <=> NULL;
SELECT (1 = 0), ('0' = 0), ('0.0' = 0), ('0.01' = 0), ('.01' = 0.01);

SELECT * from userTbl 
WHERE (userID = 'BBK') AND (name = '바비킴');

SELECT * FROM userTbl 
WHERE (userID, name) = ('BBK', '바비킴');

2) <>
SELECT * FROM userTbl 
WHERE addr <> '서울' OR mobile1 <> 010;

SELECT * FROM userTbl 
WHERE (addr, mobile1) <> ('서울', 010);

3) IS, IS NOT, IS NULL
SELECT 1 IS TRUE, 0 IS FALSE, NULL IS UNKNOWN;
SELECT 1 IS NOT UNKNOWN, 0 IS NOT UNKNOWN, NULL IS NOT UNKNOWN;
SELECT 1 IS NULL, 0 IS NULL, NULL IS NULL;

4) COALESCE 
SELECT COALESCE(NULL, 1), COALESCE(NULL, NULL, NULL);

5) GREATEST
SELECT GREATEST(34.0, 3.0, 5.0, 767.0), GREATEST('B','A','C');

6) LEAST
SELECT LEAST(34.0, 3.0, 5.0, 767.0), LEAST('B','A','C');

7) INTERVAL
SELECT INTERVAL(23, 1, 15, 17, 30, 44, 200), INTERVAL(22, 23, 30, 44, 200);

8) IN
SELECT (3, 4) IN ((1, 2), (3, 4));
SELECT (3, 4) IN ((1, 2), (3, 5));

9) STRCMP
SELECT STRCMP('kick', 'kick'), STRCMP('kick', 'going'), STRCMP('kickgoing', 'z');
SELECT STRCMP('킥', '킥'), STRCMP('킥', '고잉'), STRCMP('킥고잉', 'ㅎ');
```

## ANY/ALL/SOME 그리고 서브쿼리(SubQuery, 하위쿼리)(p.200)
- 서브 쿼리: 쿼리문 안에 또 쿼리문이 들어 있는 것
- operand(=  >  <  >=  <=  <>  !=) ANY/ALL/SOME (서브쿼리)
- 외부 쿼리는 SELECT, INSERT, UPDATE, DELETE, SET, DO 중 하나이어야 함

```
SELECT name, height 
FROM userTbl
WHERE height > 177;

SELECT name, height
FROM userTbl
WHERE height > (SELECT height FROM userTbl WHERE name = '김경호');
-- 김경호보다 키(177)가 크거나 같은 사람

SELECT name, height
FROM userTbl
WHERE height >= (SELECT height FROM userTbl WHERE addr = '경남');
-- 지역이 '경남'인 사람의 키보다 크거나 같은 사람
-- ERROR 1242 (21000): Subquery returns more than 1 row

SELECT name, height
FROM userTbl
WHERE height >= ANY(SELECT height FROM userTbl WHERE addr = '경남');
-- ANY(173, 170) 
-- height >= 173 OR height >= 170 
-- height >= 170

SELECT name, height
FROM userTbl
WHERE height >= ALL(SELECT height FROM userTbl WHERE addr = '경남');
-- ALL(173, 170)
-- height >= 173 AND height >= 170
-- height >= 173

SELECT name, height 
FROM userTbl
WHERE height = ANY(SELECT height FROM userTbl WHERE addr = '경남');
-- = ANY(173, 170)

SELECT name, height 
FROM userTbl
WHERE height IN (SELECT height FROM userTbl WHERE addr = '경남');
-- IN (173, 170)
```

### Q2. 김씨인 고객이 구매한 물건의 prodName, price 와 userID 를 조회하기

## 원하는 순서대로 정렬하여 출력 : ORDER BY(p.203)
```
SELECT name, mDate
FROM userTbl
ORDER BY mDate;
-- 기본적으로 오름차순(ASC) 정렬

SELECT name, mDate
FROM userTbl
ORDER BY mDate DESC;

SELECT name, height
FROM userTbl
ORDER BY height DESC, name ASC;
-- 키가 큰 순서대로 정렬하되 같다면 이름 순 정렬 

SELECT userID 
FROM userTbl 
ORDER BY height;
-- ORDER BY에 나온 열이 SELECT 에 있을 필요 없음
```

## 논리 연산자

| 이름 | 설명 |
|:--|:--|
| AND, && | 논리 AND |
| NOT, ! | 값의 부정 |
| OR | 논리 OR |
| XOR | 논리 eXculsive OR, (a AND (NOT b)) OR ((NOT a) AND b) |

### Q3. 결과 예상해보기
```
SELECT (NOT 10), (NOT 0), (NOT NULL), (!(1+1)), (!1+1);
SELECT (1 AND 1), (1 AND 0), (1 AND NULL), (0 AND NULL), (NULL AND 0);
SELECT (1 OR 1), (1 OR 0), (0 OR 0), (0 OR NULL), (1 OR NULL);
```

## 할당 연산자

| 이름 | 설명 |
|:--|:--|
| = | SET 과 함께 사용할 경우 할당, 그 외 비교 연산자 |
| := | 값 할당, 비교 연산으로 처리되지 않고 어떤 유효한 SQL 문에서든 할당 연산으로 사용 |

```
SELECT @var1 := 1, @var2;
SELECT @userCount:=COUNT(*) FROM userTbl;
SELECT @userCount;

UPDATE userTbl SET mobile1 = '011' WHERE userID = 'KNG';
-- 앞의 = 은 할당 연산자, 뒤의 = 은 비교 연산자 
```

## 중복된 것은 하나만 남기는 DISTINCT(p.204)
```
SELECT DISTINCT addr
FROM userTbl;
```

## 출력하는 개수를 제한하는 LIMIT(p.205)
```
USE employees;

SELECT emp_no, hire_date
FROM employees
ORDER BY hire_date ASC
LIMIT 5;

SELECT emp_no, hire_date
FROM employees
ORDER BY hire_date ASC
LIMIT 0, 5;
-- LIMIT 개수 OFFSET 시작
-- LIMIT 5 OFFSET 0과 동일
```
- 악성 쿼리문
    - 서버 처리량 많아서 결국 서버 전반적인 성능 나쁘게 하는 쿼리문
    - 예) 많은 사람이 표를 끊기 위해 줄 서 있는데 어떤 사람이 계속 판매원에게 필요치 않은 질문을 던져서 뒤에 서 있는 다른 많은 사람이 표를 끊는데 시간이 오래 걸리는 것과 같음

### Q4. 금액(price * mount)이 가장 큰 5건의 주문을 조회하기

## 테이블을 복사하는 CREATE TABLE ... SELECT(p.207)
```
CREATE TABLE 새로운테이블 (SELECT 복사할열 FROM 기존테이블)

USE sqlDB;

CREATE TABLE buyTbl2 (SELECT * FROM buyTbl);
SELECT * FROM buyTbl2;

CREATE TABLE buyTbl3 (SELECT userID, prodName FROM buyTbl);
SELECT * FROM buyTbl3;

DESC buyTbl;
DESC buyTbl2;
DESC buyTbl3;
-- PK(num), FK(userID) 등의 제약 조건은 복사되지 않음
```

## 참고
- [MySQL 5.7 Reference Manual > Operator](https://dev.mysql.com/doc/refman/5.7/en/non-typed-operators.html)