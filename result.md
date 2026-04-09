# SQL 실습 문제 답안

---

## 1. 테이블 생성하기 (CREATE TABLE)

### 1. 중복된 컬럼은 무엇인가?

`attendance` 테이블에서 중복되는 컬럼은 **`nickname`** 이다.  
같은 크루가 여러 날 출석할 때마다 `crew_id`와 `nickname`이 반복해서 저장된다.

---

### 2. crew 테이블 구성

크루의 고유 정보를 별도로 관리하기 위해 `crew_id`와 `nickname`으로 구성한다.

---

### 3. 크루 정보 추출 (DISTINCT)

```sql
SELECT DISTINCT crew_id, nickname
FROM attendance;
```

---

### 4. crew 테이블 생성 (CREATE)

```sql
CREATE TABLE crew (
    crew_id  INT PRIMARY KEY,
    nickname VARCHAR(50) NOT NULL
);
```

---

### 5. crew 테이블에 데이터 삽입 (INSERT)

```sql
INSERT INTO crew (crew_id, nickname)
SELECT DISTINCT crew_id, nickname
FROM attendance;
```

---

## 2. 테이블 컬럼 삭제하기 (ALTER TABLE)

### 1. 불필요해지는 컬럼은?

`crew` 테이블을 별도로 만들었으므로 `attendance` 테이블에서 **`nickname`** 컬럼이 불필요해진다.

---

### 2. 컬럼 삭제 (ALTER)

```sql
ALTER TABLE attendance
DROP COLUMN nickname;
```

---

## 3. 외래키 설정하기

`attendance` 테이블의 `crew_id`가 `crew` 테이블의 `crew_id`를 참조하도록 외래키를 설정한다.  
이렇게 하면 `crew` 테이블에 존재하지 않는 `crew_id`가 `attendance`에 들어오는 것을 방지할 수 있다.

```sql
ALTER TABLE attendance
ADD CONSTRAINT fk_crew
FOREIGN KEY (crew_id) REFERENCES crew(crew_id);
```

---

## 4. 유니크 키 설정

`crew` 테이블의 `nickname` 컬럼에 UNIQUE 제약을 추가하여 중복 닉네임을 방지한다.

```sql
ALTER TABLE crew
ADD CONSTRAINT uq_nickname UNIQUE (nickname);
```

---

## 5. 크루 닉네임 검색하기 (LIKE)

닉네임 첫 글자가 '디'인 크루를 검색한다.

```sql
SELECT *
FROM crew
WHERE nickname LIKE '디%';
```

---

## 6. 출석 기록 확인하기 (SELECT + WHERE)

어셔의 3월 6일 출석 기록이 존재하는지 확인한다.

```sql
SELECT *
FROM attendance
WHERE crew_id = (SELECT crew_id FROM crew WHERE nickname = '어셔')
  AND attendance_date = '2023-03-06';
```

---

## 7. 누락된 출석 기록 추가 (INSERT)

어셔의 3월 6일 출석 기록을 수동으로 추가한다.

```sql
INSERT INTO attendance (crew_id, attendance_date, start_time, end_time)
VALUES (
    (SELECT crew_id FROM crew WHERE nickname = '어셔'),
    '2023-03-06',
    '09:31:00',
    '18:01:00'
);
```

---

## 8. 잘못된 출석 기록 수정 (UPDATE)

주니의 3월 12일 등교 시각을 10시 5분에서 10시 정각으로 수정한다.

```sql
UPDATE attendance
SET start_time = '10:00:00'
WHERE crew_id = (SELECT crew_id FROM crew WHERE nickname = '주니')
  AND attendance_date = '2023-03-12';
```

---

## 9. 허위 출석 기록 삭제 (DELETE)

아론의 3월 12일 출석 기록을 삭제한다.

```sql
DELETE FROM attendance
WHERE crew_id = (SELECT crew_id FROM crew WHERE nickname = '아론')
  AND attendance_date = '2023-03-12';
```

---

## 10. 출석 정보 조회하기 (JOIN)

`crew` 테이블과 `attendance` 테이블을 JOIN하여 닉네임과 함께 출석 기록을 조회한다.

```sql
SELECT c.nickname, a.attendance_date, a.start_time, a.end_time
FROM attendance a
JOIN crew c ON a.crew_id = c.crew_id;
```

---

## 11. nickname으로 쿼리 처리하기 (서브 쿼리)

닉네임을 직접 입력하면 해당 크루의 출석 기록을 조회할 수 있도록 서브 쿼리를 사용한다.

```sql
SELECT *
FROM attendance
WHERE crew_id = (
    SELECT crew_id
    FROM crew
    WHERE nickname = '어셔'
);
```

---

## 12. 가장 늦게 하교한 크루 찾기

3월 5일에 가장 늦게 하교한 크루의 닉네임과 하교 시각을 조회한다.

```sql
SELECT c.nickname, a.end_time
FROM attendance a
JOIN crew c ON a.crew_id = c.crew_id
WHERE a.attendance_date = '2023-03-05'
ORDER BY a.end_time DESC
LIMIT 1;
```

---

## 13. 크루별로 '기록된' 날짜 수 조회

```sql
SELECT crew_id, COUNT(*) AS record_count
FROM attendance
GROUP BY crew_id;
```

---

## 14. 크루별로 등교 기록이 있는 날짜 수 조회

```sql
SELECT crew_id, COUNT(start_time) AS attend_count
FROM attendance
WHERE start_time IS NOT NULL
GROUP BY crew_id;
```

---

## 15. 날짜별로 등교한 크루 수 조회

```sql
SELECT attendance_date, COUNT(*) AS crew_count
FROM attendance
WHERE start_time IS NOT NULL
GROUP BY attendance_date;
```

---

## 16. 크루별 가장 빠른 등교 시각(MIN)과 가장 늦은 등교 시각(MAX)

```sql
SELECT crew_id,
       MIN(start_time) AS earliest_start,
       MAX(start_time) AS latest_start
FROM attendance
WHERE start_time IS NOT NULL
GROUP BY crew_id;
```
