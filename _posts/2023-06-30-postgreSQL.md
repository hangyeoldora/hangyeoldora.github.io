# postgreSQL

### 정의

객체-관계형 데이터베이스 관리 시스템

- 현대적이고 오픈 소스

### DB 명령어

- DB
  
  - \l -> list
  
  - \c "database name" -> db 변경

- 테이블 생성 (relation)
  
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
  // TIMESTAMP에는 전체 날짜, 분 초 등 다 포함되어 있어서 date 대신 사용
  
  // constraints 추가
  ```

- `\d "table_name` 으로 테이블, 뷰, 시퀀스 또는 인덱스 설명 확인 가능

- `\dt` 는 테이블 목록만 조회

- int 대신 bigserial을 사용하면 자동으로 증가하기 때문에 이전 번호를 기억할 필요 X
  
  > bigserial = autoIncrementing eight-byte Integer
  > 
  > (alias = serial8)
  > 
  > bigint = signed eight-byte Integer (alias = int8)
  > 
  > integer = signed four-byte integer (alias=int, int4)

- bigserial로 생성하면 `\d`로 조회할 경우, 해당 키 값과 테이블 명, seq가 포함된 시퀀스이름이 생성됨 => `person_id_seq`

- 테이블 삭제

- DROP TABLE person;

- 테이블 레코드 추가

- Insert Into ( col_name ) VALUES (value);
  
  ```sql
  INSERT INTO person(first_name, last_name, gender, date_of_birth) VALUES ('Anne','Smith','FEMALE',date '1988-01-09');
  
  INSERT INTO person(first_name, last_name, gender, date_of_birth, email
  ) VALUES ('Jake','Jones','MALE',date '1990-12-16', 'jake@gmail.com');
  ```

- 값 조회는 `SELECT *  FROM 'table_name'`

- 예시 데이터 생성은 <a href="https://www.mockaroo.com/">mockaroo</a> 에서 생성
  
  blank => null 값

- 테이블 조회

- 전체
  
  `SELECT * FROM 'table_name'`

- 특정 컬럼
  
  `SELECT 'col_name' FROM 'table_name'`

- 정렬 <b>ORDER BY</b>
  
  - 정렬방식은 ASC(기본값), DESC
  
  `SELECT * FROM person ORDER BY coutry_of_birth "정렬방식"`

- Distinct
  
  - 중복 제거
    
    ```sql
    SELECT DISTINCT country_of_birth FROM person ORDER BY country_of_birth;
    ```

- 조건 WHERE
  
  - 단일 조건
    
    ```sql
    SELECT * FROM person WHERE gender = 'Male';
    ```
  
  - 다중 조건
    
    ```sql
    SELECT * FROM person WHERE gender = 'Male' AND country_of_birth = 'Poland'; 
    SELECT * FROM person WHERE gender = 'Male' AND (country_of_birth = 'Poland' OR country_of_birth = 'China');
    ```

- 비교연산자
  
  - 4
    
    ```sql
    SELECT 1=1; // t (true)
    SELECT 1=2; // f (false)
    SELECT 1<>1 // f (!==와 같음)
    ```

- Limit, Offset & Fetch
  
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
     // SELECT select_list FROM table_expression [ ORDER BY... ] [ LIMIT{number|ALL}] [OFFSET number]
     SELECT * FROM person OFFSET 5 LIMIT 5;
     ```
  
  3. Fetch
     
     - 생성된 커서를 사용하여 행을 검색
     
     - 일반적으로 생성된 커서는 첫번 째 행 앞에 위치
     
     - 일부 행을 출력한 후, 커서는 가장 최근에 검색된 행에 위치
     
     - 커서의 위치는 쿼리 결과에 따라 다를 수 있음
     
     - fetch를 통해 LIMIT과 같은 효과를 나타낼 수 있음
     
     ```sql
     // FETCH [ direction ] [ FROM | IN ] cursor_name where direction;
     
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
4. IN
   
   - 반복
     
     ```sql
     SELECT * FROM person WHERE country_of_birth = 'China' OR country_of_birth = 'France' OR country_of_birth = 'Brazil';
     // IN을 통해 반복되는 컬럼명을 줄일 수 있음
     SELECT * FROM person WHERE country_of_birth IN ('China','Brazil','France');
     ```
   
   - 반

5. between
   
   - select 검색 범위 지정 가능
     
     ```sql
     SELECT * FROM person WHERE date_of_birth BETWEEN DATE '2000-01-01' AND '2023-01-01';
     ```

6. Like and iLike
   
   1. Like
      
      - 문자열 검색
      
      - 패턴에 맞게 명세한 구문이 포함된 컬럼을 찾을 때 사용
      
      ```sql
      SELECT * FROM person WHERE email LIKE '%@google.%';
      
      SELECT * FROM person WHERE email LIKE '_________@%';
      ```

7. Group By
   
   - 컬럼에 따른 그룹핑이 가능
     
     ```sql
     SELECT country_of_birth FROM person GROUP BY country_of_birth;
     ```
   
   - Count와 같은 Aggregate Function을 함께 사용하여 결과 출력 가능
     
     ※ Count(*) => 입력 행들의 수, Type: bigint
     
     ```sql
     SELECT country_of_birth, COUNT(*) FROM person GROUP BY country_of_birth ORDER BY country_of_birth;
     ```

8. Group By Having
   
   - Group By와 함께 추가적인 필터를 적용할 때 사용
   
   - Having 키워드는 ORDER BY보다 앞에서 사용해야 함
   
   - Having condition
     
     ```sql
     SELECT country_of_birth, COUNT(*) FROM person GROUP BY country_of_birth HAVING COUNT(*) > 5 ORDER BY country_of_birth;
     ```
   
   - COUNT와 같은 함수(Aggregate Functions)는 [공식 문서](https://www.postgresql.org/docs/11/functions-aggregate.html) 에서 확인

9. Useful Aggregate Function
   
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

10. discount example and round function
    
    - Round()
      
      ```sql
      SELECT make, model, price, ROUND(price * .10,2), ROUND(price - (price * .10),2) FROM car;
      ```

11. Alias
    
    - Alias로 컬럼명을 override할 수 있음
      
      ```sql
      SELECT make, model, price AS original_price, ROUND(price * .10,2) AS ten_percent_value, ROUND(price - (price * .10),2) AS discount_after_10_percent FROM car;
      ```
    
    - sdsd

12. Coalesce
    
    - 인수 중에서 Null이 아닌 첫 번째 값을 반환
      
      ```sql
      // SELECT COALESCE(description, short_description, (none))
      // description이 null이 아니고 게다가 short_description도 아니고..
      SELECT COALESCE(1); // 1
      SELECT COALESCE(null, 1); // 1
      SELECT COALESCE(null, null, 1, 10); // 1
      ```
    
    - 인수는 무조건 동일한 데이터 타입이어야 함
    
    - 컬럼 중에 NULL 값이 있을 경우, NULL 대신 다른 값을 넣어 출력 가능
      
      ```sql
      // NULL 값에 Email not provided라는 문장을 넣음
      SELECT COALESCE(email,'Email not provided') from person;
      ```
    
    - 

13. NULLIF
    
    - 일반적으로 숫자를 0으로 나누면 `ERROR: division by zero`라는 문구가 나옴
    
    - NULLIF를 통해 Exception 없이 해결 가능
      
      ```sql
      SELECT NULLIF(10, 0) // 10
      SELECT 10 / NULLIF(0,0); // null
      SELECT COALESCE(10 / NULLIF(0,0),0); // null
      ```
    
    - NULLIF는 value1과 value2가 <u>같으면 null</u>을 반환하고 <u>아니라면 value1</u>을 반환
    
    - a

14. Timestamps and Date
    
    - TimeStamp
      
      ```sql
      SELECT NOW(); // 2023-06-30 16:37:36.263998+09
      SELECT NOW()::DATE; // 2023-06-30
      SELECT NOW()::TIME;
      ```
    
    - asdsad
    
    - Interval Operator
      
      - 해당 operator를 통해 날짜 계산이 가능
      
      ```sql
      SELECT NOW() - INTERVAL '10 YEAR' AS YEAR; // 2013-06-30 17:47:47.523218+09
      
      SELECT NOW() - INTERVAL '10 MONTHS'; // 2022-08-30 17:48:03.603406+09
      // 날짜만 출력하는 경우
      SELECT NOW()::DATE - INTERVAL '5 MONTHS'; // 2023-01-30 00:00:00
      // DATE
      ```
    
    - sds
    
    - sds
    
    - sd

           

15. asd
