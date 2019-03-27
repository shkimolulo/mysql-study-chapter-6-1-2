# '_' 가 들어가는 특정 문자열 검색
```
INSERT INTO userTbl VALUES('U_B', '언더스코어', 1999, '서울', '010', 5555555, 180, '2019-03-28');

SELECT * FROM userTbl
WHERE userID LIKE '%_%';

SELECT * FROM userTbl
WHERE userID LIKE '%\_%';

SELECT * FROM userTbl
WHERE userID LIKE '%$_%' ESCAPE '$';
```
- ESCAPE 이용해서 와일드 카드 문자(_, %) 를 문자로 해석할 수 있게 함
- 명시하지 않으면 '\\\(Backslash)'가 기본 ESCAPE 문자

# IS, =
```
SELECT 10 IS TRUE;
-- 1

SELECT 10 = TRUE;
-- 0, 10 != 1 이기 때문에
```
- MariaDB 에서 FALSE(False, false) 와 0, TRUE(True, true) 와 1 은 동의어
- IS
    - 양 쪽의 값을 boolean 연산
    - 수행 시 0과 1의 동의어가 없다고 가정
- = : 양 쪽의 값이 같은지 확인
- UNKNOWN
    - IS 연산에서 사용할 수 있는 TRUE, FALSE 외 세번째 상수
    - 항상 NULL 동의어
- 참고
    - [MariaDB Boolean Literals](https://mariadb.com/kb/en/library/sql-language-structure-boolean-literals/)
    - [MariaDB Boolean 예제](https://mariadb.com/kb/en/library/boolean/)

# STRCMP 비교 기준
- 사전식으로 앞의 글자부터 비교하고 같으면 뒤의 글자를 비교
- 문자열을 비교할 때 collation(character set 안에서 character 들을 비교하기 위한 rule) 을 사용하는데 이에 따라서 다른 결과를 조회
- 참고: [How to Use STRCMP() to Compare 2 Strings in MySQL](https://database.guide/how-to-use-strcmp-to-compare-2-strings-in-mysql/)

# SOME, ANY 비교
```
SELECT s1 FROM t1 WHERE s1 <> ANY  (SELECT s1 FROM t2);
SELECT s1 FROM t1 WHERE s1 <> SOME (SELECT s1 FROM t2);
```
- SOME 은 ANY 의 ailas 로, 같은 기능을 하지만 SOME 이 의미를 더 이해하기 쉬움
- "a is not equal to any b" = "there is no b which is equal to a"
- "there is some b to which a is not equal." 의 의미가 MySQL 쿼리와 더 비슷함
- 참고: [Subqueries with ANY, IN, or SOME](https://dev.mysql.com/doc/refman/5.7/en/any-in-some-subqueries.html)
