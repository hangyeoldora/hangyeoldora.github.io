---
title: "[PostgreSQL] 테스트 DB로 알아보는 PostgreSQL 필수 문법"
author: hangyeoldora
categories: [All, PostgreSQL]
tags: [postgreSQL]
---



# postgreSQL

## 정의

객체-관계형 데이터베이스 관리 시스템

- 현대적이고 오픈 소스
  

## DB 세팅

### 테스트 DB 및 명령어

- `\l` -> Database 목록

- `\c` "database name" -> 데이터베이스 변경

- `\i` "file path" => 외부 SQL을 가져올 경우 사용

- `\d "table_name` 으로 테이블, 뷰, 시퀀스 또는 인덱스 설명 확인 가능

- `\dt` 는 테이블 목록만 조회

- `\x` 컬럼 단위 보기(ON/OFF) => 테이블이 가로로 넓게 나올 경우, 보기 편하게 해줌

- 예시 데이터 생성은 <a href="https://www.mockaroo.com/">mockaroo</a> 에서 생성

  blank => null 값
  

# DB 명령어

### 테이블 생성 (relation)

```sql
CREATE TABLE table_name (
    column name + data type + constraints if any
);
CREATE TABLE person (
 id int,
 first_name VARCHAR(50),
 last_name VARCHAR(50),
 gender VARCHAR(6),
 data_of_birth TIMESTAMP,
);
CREATE TABLE person(
 id BIGSERIAL NOT NULL PRIMARY KEY,
 first_name VARCHAR(50) NOT NULL ,
 last_name VARCHAR(50) NOT NULL ,
 gender VARCHAR(7) NOT NULL ,
 date_of_birth DATE NOT NULL,
 email VARCHAR(150) // nullable
);
-- TIMESTAMP에는 전체 날짜, 분 초 등 다 포함되어 있어서 date 대신 사용

-- constraints 추가
```



- int 대신 bigserial을 사용하면 자동으로 증가하기 때문에 이전 번호를 기억할 필요 X

  > bigserial = autoIncrementing eight-byte Integer
  > 
  >  (alias = serial8), 시퀀스에 의해 관리
  > 
  >  bigint = signed eight-byte Integer (alias = int8)
  > 
  >  integer = signed four-byte integer (alias=int, int4)

- bigserial로 생성하면 `\d`로 조회할 경우, 해당 키 값과 테이블 명, seq가 포함된 시퀀스이름이 생성됨 => `person_id_seq`
  

### 테이블 레코드 추가

```sql
-- Insert Into ( col_name ) VALUES (value);
INSERT INTO person(first_name, last_name, gender, date_of_birth) VALUES ('Anne','Smith','FEMALE',date '1988-01-09');

INSERT INTO person(first_name, last_name, gender, date_of_birth, email
) VALUES ('Jake','Jones','MALE',date '1990-12-16', 'jake@gmail.com');
```



### 테이블 조회

```sql
-- 전체 조회
SELECT * FROM 'table_name';

-- 특정 컬럼 조회
SELECT 'col_name' FROM 'table_name';
```



### 테이블 삭제

- 테이블 삭제

  ```sql
  DROP TABLE person;
  ```

  

### 정렬 (ORDER BY)

- 정렬방식은 ASC(기본값), DESC

  ```sql
  SELECT * FROM person ORDER BY country_of_birth "정렬방식"
  ```

  

### 중복 제거 (DISTINCT)

- 중복 제거
  
  ```sql
  SELECT DISTINCT country_of_birth FROM person ORDER BY country_of_birth;
  ```



### 조건

#### WHERE 절

- 단일 조건
  
  ```sql
  SELECT * FROM person WHERE gender = 'Male';
  ```

- 다중 조건
  
  ```sql
  SELECT * FROM person WHERE gender = 'Male' AND country_of_birth = 'Poland'; 
  SELECT * FROM person WHERE gender = 'Male' AND (country_of_birth = 'Poland' OR country_of_birth = 'China');
  ```

- 비교 연산자

  ```sql
  SELECT 1=1; // t (true)
  SELECT 1=2; // f (false)
  SELECT 1<>1 // f (!==와 같음)
  ```



#### Limit, Offset & Fetch

1. Limit
   
   - 조회결과 수를 제한함
   
   ```sql
   SELECT * FROM person LIMIT 5;
   ```

2. Offset
   
   - 주어진 수 이후의 결과 값을 나타냄
   
   - OFFSET 0은 OFFSET이 없는 것과 같음
   
   - OFFSET과 LIMIT이 같이 있을 경우, OFFSET 행은 LIMIT으로 반환되는 행을 스킵하고 그 다음 행부터 반환
   
   ```sql
   -- SELECT select_list FROM table_expression [ ORDER BY... ] [ LIMIT{number|ALL}] [OFFSET number]
   SELECT * FROM person OFFSET 5 LIMIT 5;
   ```

3. Fetch
   
   - 생성된 커서를 사용하여 행을 검색
   
   - 일반적으로 생성된 커서는 첫번 째 행 앞에 위치
   
   - 일부 행을 출력한 후, 커서는 가장 최근에 검색된 행에 위치
   
   - 커서의 위치는 쿼리 결과에 따라 다를 수 있음
   
   - fetch를 통해 LIMIT과 같은 효과를 나타낼 수 있음
   
   ```sql
   -- FETCH [ direction ] [ FROM | IN ] cursor_name where direction;
   
   SELECT * FROM OFFSET 5 FETCH FIRST 5 ROW ONLY;
   ```
   
   - direction에는 다음과 같은 항목들이 가능
     
     - NEXT
     
     - PRIOR
     
     - FIRST
     
     - LAST
     
     - 앞 형식들은 커서를 이동한 후, 단일 행을 가져옴
     
     - ---
     
     - ABSOLUTE <b>count</b>
     
     - RELATIVE <b>count</b>
     
     - 위 형식은 행이 없으면 빈 결과를 반환하고 첫 번째 행 또는 마지막 행 뒤에 커서가 배치됨.
     
     - ---
     
     - FORWARD
     
     - FORWARD <b>count</b>
     
     - FORWARD ALL
     
     - BACKWARD
     
     - BACKWARD <b>count</b>
     
     - BACKWARD ALL
     
     - 위 형식은 커서를 이동하지 않고 최근에 가져온 행을 다시 가져오도록 함
        커서가 첫 번째 행 앞이나 마지막 행 뒤에 위치하지 않으면 성공하며, 위치할 경우 행을 반환하지 않음
     
     - ---
     
     - <b>count</b>
     
     - ALL

#### IN (Subquery Expression)

- 같은 컬럼에서 값을 찾을 경우, 중복되는 부분을 줄일 수 있음

- 반드시 하나의 컬럼을 반환해야 함

- `IN`의 결과는 어떠한 동일 서브쿼리 행이 발견될 경우, true를 나타냄

- 동일한 행이 발견되지 않을 경우, false
  
  ```sql
  // expression IN (subquery)
  
  SELECT * FROM person WHERE country_of_birth = 'China' OR country_of_birth = 'France' OR country_of_birth = 'Brazil';
  -- IN을 통해 반복되는 컬럼명을 줄일 수 있음
  SELECT * FROM person WHERE country_of_birth IN ('China','Brazil','France');
  ```

- -

#### between

- select 검색 범위 지정 가능
  
  ```sql
  SELECT * FROM person WHERE date_of_birth BETWEEN DATE '2000-01-01' AND '2023-01-01';
  ```



#### Like and iLike

1. Like
   
   - 문자열 검색
   
   - 패턴에 맞게 명세한 구문이 포함된 컬럼을 찾을 때 사용
   
   ```sql
   SELECT * FROM person WHERE email LIKE '%@google.%';
   
   SELECT * FROM person WHERE email LIKE '_________@%';
   ```



#### Group By

- 컬럼에 따른 그룹핑이 가능
  
  ```sql
  SELECT country_of_birth FROM person GROUP BY country_of_birth;
  ```

- Count와 같은 Aggregate Function을 함께 사용하여 결과 출력 가능
  
  ※ Count(*) => 입력 행들의 수, Type: bigint
  
  ```sql
  SELECT country_of_birth, COUNT(*) FROM person GROUP BY country_of_birth ORDER BY country_of_birth;
  ```



#### Group By Having

- Group By와 함께 추가적인 필터를 적용할 때 사용

- Having 키워드는 ORDER BY보다 앞에서 사용해야 함

- Having condition
  
  ```sql
  SELECT country_of_birth, COUNT(*) FROM person GROUP BY country_of_birth HAVING COUNT(*) > 5 ORDER BY country_of_birth;
  ```

- COUNT와 같은 함수(Aggregate Functions)는 [공식 문서](https://www.postgresql.org/docs/11/functions-aggregate.html) 에서 확인



### Primary Key (PK)

- uniquely identify a record in tables

- 이미 있는 값을 넣을 경우, 'already exists' 문구와 함께 에러 발생

  ```sql
  -- 기본키 확인을 위한 조회
  SELECT * FROM person LIMIT 5; -- "person_pkey" PRIMARY_KEY, btree(id);
  ```

- 기본키 삭제

  ```sql
  -- 기본키 삭제는 조회에서 나온 PRIMARY_KEY 값 사용
  ALTER TABLE person DROP CONSTRAINT person_pkey;
  ```

- sds

- 기본키 추가

  - 기본키로 설정하려는 해당 컬럼의 값들이 unique할 경우 가능 (모든 행 unique)

  ```sql
  -- PRIMARY KEY (value)
  -- 하나의 기본키로 충분하지 않은 경우, 추가 가능 PRIMARY KEY (v1, v2);
  ALTER TABLE person ADD PRIMARY KEY (id);
  ```

- -

### Unique 제약조건

- 컬럼 또는 컬럼의 그룹에 들어있는 데이터가 테이블의 모든 행 사이에서 unique

  ```sql
  -- 1. 테이블이 있는 경우 유니크 부여
  -- (1) ADD CONSTRAINT key_name UNIQUE(column name);
  ALTER TABLE person ADD CONSTRAINT unique_email_address UNIQUE(email);
  -- => "unique_email_address" UNIQUE CONSTRAINT, btree (email)
  
  -- (2) ADD CONSTRAINT UNIQUE(column name);
  ALTER TABLE person ADD CONSTRAINT UNIQUE(email);
  -- => "person_email_key" UNIQUE CONSTRAINT, btree (email);
  -- key name을 자체적으로 부
  
  -- 테이블 생성 시 부여
  CREATE TABLE test(
      idx integer UNIQUE, -- idx integer 후, 아래에서 UNIQUE(idx)도 가능
      name text,
      price numeric
  );
  ```

- 삭제

  ```sql
  ALTER TABLE person DROP CONSTRAINT unique_email_address;
  ```



### Check constraint

- 일반적인 제약조건이며, 특정 열의 값이 true(boolean)를 충족해야 지정 가능

  ```sql
  -- 제약 조건에 이름을 할당한 경우
  ALTER TABLE person ADD CONSTRAINT gender_constraint CHECK (gender = 'Female' OR gender = 'Male');
  
  -- 테이블 생성 시, 할당
  CREATE TABLE test(
      idx interger,
      name text,
      price numeric,
      CHECK (price>0) -- 또는 CONSTRAINT valid_price CHECK (price>0)
  );
  -- 추후 ALTER쪽 추가하
  ALTER TABLE users ALTER COLUMN create_at SET default 'NOW()';
  ```

- ALTER로 지정 시에 특정 열의 CHECK 값이 만족하지 않으면 실행되지 않음



### 레코드 삭제

- DELETE 명령어를 통해서 삭제

- 한 번에 `DELETE`하는 것은 위험, `WHERE` 조건을 붙여서 사용하는 것이 좋음

  ```sql
  DELETE FROM person;
  ```

- 삭제 후, 다시 데이터를 넣으면 BIGSERIAL로 부여한 PRIMARY KEY `id`는 1이 아닌 1001번대부터 시작 => 이는 <u>시퀀스를 리셋</u>하지 않았기 때문임

- 시퀀스 리셋 (TRUNCATE)

  ```sql
  -- TRUNCATE은 테이블에 설정된 모든 행을 빠르게 삭제 
  TRUNCATE someTable RESTART IDENTITY;
  
  -- 이 후에 in`
  sert하면 정상적으로 1부터 나옴
  ```

- -

### Update Record

- ```sql
  -- single column
  UPDATE person SET email = 'fran@gmail.com' WHERE id=1;
  -- multiple column
  UPDATE person SET first_name = 'Omar', last_name = 'Montana' WHERE id=2011;
  ```

- 업데이트 규칙



### **Upsert**

- <u>UPDATE 또는 INSERT를 의미</u>하는 UPSERT operation

- 행이 이미 존재하여 업데이트하는 대신 스킵하거나 행을 삽입할 수 있게 함

- INSERT 구문의 `ON CONFLICT`절을 사용

- 에러나 예외 상황 발생 시, 무시하고 넘어갈 수 있는 것이 `ON CONFLICT`절

- 기본키, unique key가 없거나 제약조건이 없으면 에러 발생

- 사용은 `ON CONFLICT [conflict_target] [conflict_action]` 

- Conflict action 구분

  1. DO NOTHING

     - 단순히 행 삽입을 피하고 아무것도 하지 않음
     - conflict_target이 필요하지 않음

  2. DO UPDATE SET {}

     - 등록 요청을 받은 후, 2초 정도 후에 이메일 수정 등을 하려고 할 때 사용
     - `DO UPDATE SET column1 = value1,.. WHERE` 로 테이블 필드 업데이트

  ```sql
  -- 같은 id로 INSERT할 경우, 에러 발생
  INSERT INTO person (
      id,
      first_name,
      last_name,
      gender,
      email,
      date_of_birth,
      country_of_birth)
  VALUES (
      1,
      'Russ,'
      'Ruddoch',
      'Male',
      'rruddoch7@hhs.gov',
      DATE '1952-09-25',
      'Norway'
  );
  -- key (id)=(1) already exists. 에러 발생
  
  -- ON CONFLICT option을 사용할 경우
  -- 1. DO NOTHING
  INSERT INTO person (
      id,
      first_name,
      last_name,
      gender,
      email,
      date_of_birth,
      country_of_birth)
  VALUES (
      1,
      'Russ,'
      'Ruddoch',
      'Male',
      'rruddoch7@hhs.gov',
      DATE '1952-09-25',
      'Norway'
  ) ON CONFLICT (id) DO NOTHING; -- unique키에 하였기에 INSERT 0 0; 반영
  -- 2. DO UPDATE
    INSERT INTO person (
        id,
        first_name,
        last_name,
        gender,
        email,
        date_of_birth,
        country_of_birth)
    VALUES (
        1,
        'Russ,'
        'Ruddoch',
        'Male',
        'rruddoch7@hhs.gov.uk',
        DATE '1952-09-25',
        'Norway'
    ) ON CONFLICT (id) DO UPDATE SET email = EXCLUDED.email; 
    -- INSERT 0 1; 반영
  
  -- 다양한 UPDATE 반영 가능
   ...ON CONFLICT (id) DO UPDATE SET email = EXCLUDED.email,
   last_name = EXCLUDED.last_name, first_name=EXCLUDED.first_name;
  ```

    

4. 외래키와 Joins

   1. 외래키

      - 다른 테이블의 primary key를 참조

      - 참조하는 컬럼의 타입과 타입이 같아야 함

      - 외래키는 유일함

      - 테이블 간 관계를 추가하기

        ```sql
        create table person (
            id BIGSERIAL NOT NULL PRIMARY KEY ,
            first_name VARCHAR(105) NOT NULL ,
            last_name VARCHAR(150) NOT NULL ,
            email VARCHAR(150),
            gender VARCHAR(20) NOT NULL ,
            date_of_birth DATE NOT NULL ,
            country_of_birth VARCHAR(50) NOT NULL,
            -- FOREIGN KEY 설정
            car_id BIGINT REFERENCES car(id),
            UNIQUE (car_id)
        );
        ```

      - 실행: `UPDATE person SET car_id = 2 WHERE id = 1;`

      - 존재하는 값이나 없는 id로 UPDATE하는 경우, 에러 발생

      - -

   2. inner join

      - 두 테이블을 합치는 형태

      - 조건에 맞게 A 테이블과 B 테이블의 레코드를 합쳐 C의 형태가 됨

        ```sql
        SELECT person.first_name, car.make, car.model, car.price FROM person JOIN car ON person.car_id = car.id;
        ```

      - -

   3. left join

      - 두 테이블을 합치지만 A 테이블 전체와 B와 매치되는 부분을 합침

      - -

        ```sql
        SELECT * FROM person LEFT JOIN car ON car.id = person.car_id;
        
        -- car_id가 NULL인 것을 찾는 방법
        -- 1. is NULL
        SELECT * FROM person WHERE car_id IS NULL;
        -- 2. LEFT JOIN 사용
        SELECT * FROM person LEFT JOIN car ON car.id = person.car_id WHERE car.* IS NULL;
        ```

      - -

   4. 외래키 삭제

      - 참조하고 있는 키의 레코드를 삭제하려고 할 경우, 에러가 발생

      - 행을 없애거나 참조하고 있는 키의 데이터를 NULL로 변경

        ```sql
        DELETE FROM person WHERE id=3;
        DELETE FROM car WHERE id=3;
        ```

      - ㄴㅇ

   

### Useful Aggregate Function

- 신규 데이터 추가
  
  추가 중, 인코딩 에러가 날 경우에 client-encoding 설정 필요한데,
  
  환경 변수에서 새로운 시스템 변수 'PGCLIENTENCODING' 추가

- 기본 함수 예
  
  ```sql
  SELECT MIN(price) FROM car;
  SELECT AVG(price) FROM car;
  SELECT ROUND(AVG(price)) FROM car;
  SELECT make, model, MIN(price) FROM car GROUP BY make, model; 
  SELECT SUM(price) FROM car;
  ```



### discount example and round function

- 반올림 함수

- round(numeric, int)이며, 수를 반올림하면서 int만큼 decimal을 남김
  
  ```sql
  SELECT make, model, price, ROUND(price * .10,2), ROUND(price - (price * .10),2) FROM car;
  ```



### Alias

- Alias로 컬럼명을 override할 수 있음
  
  ```sql
  SELECT make, model, price AS original_price, ROUND(price * .10,2) AS ten_percent_value, ROUND(price - (price * .10),2) AS discount_after_10_percent FROM car;
  ```

- -

### Coalesce

- 인수 중에서 Null이 아닌 첫 번째 값을 반환
  
  ```sql
  -- SELECT COALESCE(description, short_description, (none))
  -- description이 null이 아니고 게다가 short_description도 아니고..
  SELECT COALESCE(1); // 1
  SELECT COALESCE(null, 1); // 1
  SELECT COALESCE(null, null, 1, 10); -- 1
  ```

- 인수는 무조건 동일한 데이터 타입이어야 함

- 컬럼 중에 NULL 값이 있을 경우, NULL 대신 다른 값을 넣어 출력 가능
  
  ```sql
  -- NULL 값에 Email not provided라는 문장을 넣음
  SELECT COALESCE(email,'Email not provided') from person;
  ```

- -

### NULLIF

- 일반적으로 숫자를 0으로 나누면 `ERROR: division by zero`라는 문구가 나옴

- NULLIF를 통해 Exception 없이 해결 가능
  
  ```sql
  SELECT NULLIF(10, 0) -- 10
  SELECT 10 / NULLIF(0,0); -- null
  SELECT COALESCE(10 / NULLIF(0,0),0); -- null
  ```

- NULLIF는 value1과 value2가 <u>같으면 null</u>을 반환하고 <u>아니라면 value1</u>을 반환

- -

### Timestamps and Date

- TimeStamp
  
  ```sql
  SELECT NOW(); -- 2023-06-30 16:37:36.263998+09
  SELECT NOW()::DATE; -- 2023-06-30
  SELECT NOW()::TIME;
  ```

- -

### Interval Operator

- 해당 operator를 통해 날짜 계산이 가능
  
  ```sql
  SELECT NOW() - INTERVAL '10 YEAR' AS YEAR; -- 2013-06-30 17:47:47.523218+09
  
  SELECT NOW() - INTERVAL '10 MONTHS'; -- 2022-08-30 17:48:03.603406+09
  -- 날짜만 출력하는 경우
  SELECT NOW()::DATE - INTERVAL '5 MONTHS'; -- 2023-01-30 00:00:00
  -- 이는 시간이 00으로 출력
  
  -- DATE
  SELECT (NOW()+INTERVAL '10 MONTHS')::DATE; -- 2024-05-03
  ```

- -

### Extracting fields

- EXTRACT(filed FROM source);

- date/time 값으로부터 시간이나 년도를 반환
  
  ```sql
  SELECT EXTRACT(YEAR FROM NOW()); -- 2023
  SELECT EXTRACT(DAY FROM NOW()); -- 3
  SELECT EXTRACT(CENTURY FROM NOW()); -- 21
  ```



### AGE Function

- v1인자에서 v2인자를 뺀 날짜 결과(년도, 월, 일)를 반환

- AGE(timestamp v1, timestamp v2) => v1 - v2

- AGE(timestamp v1) => current date - v1
  
  ```sql
  -- AGE(timestamp, timestamp)
  -- now 데이터 예: 2023-07-03 09:31:43.249729+09
  -- date_of_birth 데이터 예: 2022-09-03
  SELECT first_name, last_name, gender, country_of_birth, date_of_birth, AGE(NOW(), date_of_birth) AS age FROM person LIMIT 5;
  
  -- AGE(timestamp)
  -- 결과값 
  SELECT first_name, last_name, gender, country_of_birth, date_of_birth, AGE(date_of_birth) AS age FROM person LIMIT 5;
  ```

- 

### Sequence & Serial

#### 1. Sequence

- 고유한 정수를 생성할 수 있는 데이터베이스 개체

- 정수를 기반으로 하며 숫자를 순서대로 나타내기 위해 사용
  
  ```sql
  -- 기존 person table에 BIGSERIAL 타입으로 부여한 ID 값에 있는 sequence
  SELECT * FROM person_id_seq; // result => last_value, log_cnt, is_called
  
  -- 수 증가 구문이며, autoIncrement, BIGSERIAL에는 자동
  SELECT nextval('person_id_seq');
  
  -- 시퀀스 새로 생성하여 증가
  CREATE SEQUENCE apple;
  SELECT nextval('apple'); -- 1
  SELECT nextval('apple'); -- 2
  ```

- 

#### 2. Serial

- sequence 개체는 `CREATE SEQUENCE`를 통해 만들어야 하고 불편한 점들이 많은데, 이를 해결하기 위해서 `SERIAL`이라는 자료형을 제공

- 가상의 데이터 유형이며, 독립된 SEQUENCE 개체를 이용하는 것보다 간단
  
  ```sql
  -- DataType에 SMALLSERIAL(2 bytes), SERIAL(4 bytes),
  -- BIGSERIAL(8 bytes)을 부여할 경우, autoIncrement
  -- 즉, nextval('seq_name')이 자동으로 붙음
  CREATE TABLE nice(
      idx BIGSERIAL,
      name VARCHAR(100)
  );
  
  -- 2. 직접 부여하는 경우
  --  => sequence 생성
  CREATE SEQUENCE nice_id_seq;
  -- - 테이블 생성하면서 해당 컬럼값에 DEFAULT로 설정
  CREATE TABLE good(
      idx INTEGER NOT NULL DEFAULT nextval('nice_id_seq')
  );
  ALTER SEQUENCE nice_id_seq OWNED BY good.id;
  ```

- 

### Control Structures

- postgreSQL에서 가장 유용하고 중요한 부분
- SQL 데이터를 유연하고 강력하게 조작할 수 있음

#### RETURN

- return과 함께 있는 표현식은 함수를 종료하고, 호출 대상에게 표현식의 값을 반환함

- -

#### 조건 구문 (IF, CASE)

- IF와 CASE 절은 조건에 따른 명령을 실행할 수 있도록 함

- 3가지의 IF 구문
  
  1. `IF...THEN...END IF`
  
  2. `IF...THEN...ELSE...END IF`
  
  3. `IF...THEN...ELSIF...THEN...ELSE...END IF`

- 2가지의 CASE 구문
  
  1. `CASE...WHEN...THEN...ELSE...END CASE`
     
     ```sql
     -- 예시
     CASE x -- search-expression
         WHEN 1, 2 -- expression
         THEN 
             msg := 'one or two'; -- statements
         ELSE -- optional
             msg := 'other value than one or two';
     END CASE;
     ```
     
     - `:=` 할당자로서 `=`랑 동일하며, 비교연산에서는 사용 불가
       
  
  2. `CASE WHEN...THEN...ELSE...END CASE`
     
     - 자주 쓰이는 요소
     
     ```sql
     -- 예시
     CASE
         WHEN x BETWEEN 0 AND 10 THEN
             msg := 'value is between zero and ten';
         WHEN x BETWEEN 11 AND 20 THEN
             msg := 'value is between eleven and twenty'
     END CASE;
     ```



#### LOOP & EXIT

- `LOOP`는 조건적이지 않은 반복문으로서 EXIT나 RETURN에 의해 종료되기 전까지는끝없이 반복함

- `LOOP`의 선택사항인 `label`은 EXIT와 CONTINUE로 사용 가능
  
  ```sql
  LOOP
      IF count > 0 THEN
          EXIT; -- exit loop
      END IF;
      -- 위 조건을 줄이면 다음과 같음
      EXIT WHEN count > 0;
  END LOOP;
  ```

- EXIT는 `label`이 주어지지 않으면 내부 loop를 멈추고 END LOOP를 실행

- `label`

- `WHEN`이 명시된다면, boolean-표현식이 `참`일때만 loop를 빠져나감

#### CONTINUE

- `CONTINUE [ label ] [ WHEN boolean-expression ]`

- label이 없다면, 다음 요소의 반복으로 넘어가서 시작되며, 반복문 안에 남아있던 모든 요소가 스킵된다.

- label이 존재한다면 명시해놓은대로 실행

- WHEN이 명시되어 있다면 다음 요소의 루프는오로지 `boolean-expression`이 <u>참</u>일 경우에만 시작
  
  ```sql
  LOOP
      EXIT WHEN count > 100; -- count가 100 초과라면 EXIT
      CONTINUE WHEN count < 50; -- count가 50 미만이면 CONTINUE
  EXIT LOOP;
  ```

- -

### Table Expression

- table expression은 WHERE, GROUP BY, HAVING 절을 옵션으로 가지는 FROM 절을 포함

- INNER와 OUTER 키워드는 모든 폼에서 옵션 요소로서 없어도 동일한 결과값

- INNER는 기본적인 요소 / LEFT, RIGHT, FULL

- 

  

## 추가 사항

### CSV로 SQL 결과 Export 하기

- 쿼리 결과를 데이터로 추출하기 위해서 `\copy` 명령어 사용

  ```sql
  \copy (SELECT * FROM person LEFT JOIN car ON car.id=person.car_id) TO 'C:\Users\USER\Desktop\results.csv' DELIMITER ',' CSV HEADER;
  ```

- 

### Postgresql Extensions

1. 다양한 익스텐션이 있으며, `plv8`처럼 javascript 기반 익스텐션도 존재

   - javascript function

2. 조회는 `SELECT * FROM pg_available_extensions;`

3. 유용한 것은 `uuid-ossp`

   - UUID를 생성해줌

     > Universally unique identifier (UUID)
     >
     > - 글로벌적으로 유일

   - install 방법

     ```sql
     CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
     -- 성공적으로 설치 시, CREATE EXTENSION 문구 출력
     ```

   - -
