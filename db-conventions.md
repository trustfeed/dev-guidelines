# DB convetions

## DDL(Data Definition Language)

### table
* 테이블명은 소문자 snake_case를 사용
* 도메인에 따라 (namespace)_ prefix 를 붙임

#### option
* MyISAM은 사용하지 않음
* Charset: `utf8` 혹은 `utf8mb4`
* Collation: `utf8_unicode_ci` 혹은 `utf8mb4_unicode_ci`

### column 
* 소문자 snake_case 를 사용
* 변경내역 추적을 위한 컬럼의 타입은 `DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP` 사용

#### column datatype 
* Primary Key는 항상 명시적으로 정의
  - 컬럼명은 id
  - `UNSIGNED INT(11)` 혹은 `UNSIGNED BIGINT(20)`을 사용
  - 반드시 not null, primary key, auto increment 속성을 지정

* Foreign Key의 사용
  - index명은 fk_로 시작 (선택사항)
  - 가급적 FK(Foreign Key)조건을 활용하여 무결성을 보장할 것
  - 단, 사용자 데이터의 경우 샤딩을 위해 FK를 걸지 않음
  - rows 수의 상한을 예측하기 어렵거나 매우 많아질 가능성이 있다면 FK를 걸지 않음

* Boolean 컬럼은 `TINYINT(1)`을 사용하며, “is_” 접두어를 붙임. ('Y/N' 은 사용하지 않음)
  - enum을 정의하지 않아도 되며, 플래그로 변경이 용이

* IPv4를 저장할 때에는 `UNSIGNED INT` 타입을 이용
  - `VARCHAR(16)`에 비해 1/4의 저장공간만 사용

* 유효기간 등을 표현할 때 이 값이 optional 한 경우 유효기간이 없음은 null로 표현
  - 유효기간이 없음을 timestamp 컬럼 값 '0000-00-00 00:00:00'으로 표현 시 ORM에서 컬럼 매칭할 때 PHP의 DateTime과 연결하는데 DateTime이 이 값을 사용할 수 없음

* 절대적인 시각 표현이나 TimeZone 지정이 필요없는 경우에는 언제나 `TIMESTAMP` 타입을 사용
  - 예) 데이터 생성시간, 수정시간, 삭제 시간 등

* 부정확한 날짜를 표기해야 할 때는, `DATE` 타입을 사용

## SQL style conventions

### Keywords
* 키워드는 반드시 uppercase를 사용한다. 

    ```SQL
    /* Good */
    SELECT COUNT(1) FROM tablename WHERE 1;
    
    /* Bad */
    select count(1) from tablename where 1;
    ```

### Names
* Named objects should not be surrounded by backticks, no matter what MySQL says when it dumps the table structure for you.
  *  If you need to use backticks because of something in your table name, rename your table.

### Indentation and newlines

* Newlines should be used for any query that is at all complex or longer than 72 characters.

* Each clause should begin a new line.
  SELECT, JOIN, LEFT JOIN, OUTER JOIN, WHERE, UNION, etc. are keywords that begin new clauses.

    ```SQL
    /* Good */
    SELECT COUNT(1)
      FROM tablename
     WHERE really_loooong_column = CONCAT(other_column, ' street');
    
    /* Bad */
    SELECT COUNT(1) FROM tablename WHERE really_loooong_column = CONCAT(other_column, ' street');
    ```    

* The keywords that begin a clause should be right-aligned.
  The idea is to make a single character column between the keywords and their objects.

    ```SQL
    /* Good */
    SELECT COUNT(1)
      FROM tablename
     WHERE 1;
    
      SELECT key_column, COUNT(1)
        FROM tablename
    GROUP BY key_column;
    
    /* Bad */
    SELECT COUNT(1)
    FROM tablename
    WHERE 1;
    ```

* Subqueries should be aligned as though the open parenthesis were the 0-column
  So, they should be indented as a unit, to identify them as subqueries.  They should continue to have the opening keywords right-aligned.

    ```SQL
    /* Good */
    SELECT *
      FROM (  SELECT candidates.name, count(1)
                FROM candidates
                JOIN votes ON candidates.id = votes.candidate_id
            GROUP BY candidates.name) name_count
      JOIN city c ON name_count.name = c.mayor;
    
    /* Bad */
    SELECT *
      FROM (SELECT candidates.name, count(1)
        FROM candidates
        JOIN votes ON candidates.id = votes.candidate_id
        GROUP BY candidates.name) name_count
      JOIN city ON name_count.name = city.mayor;
    ```   

### Structure

* Column aliases should always use the keyword AS
  This becomes significant when a query has several columns selected with columns aliased.  Without the AS keyword, a dropped comma makes two columns become a single aliased column.

    ```SQL
    /* Good */
    SELECT ebe_ebs_sox_flag_set_for_all_crs AS sox_ok
      FROM tablename;
    
    /* Bad */
    SELECT ebe_ebs_sox_flag_set_for_all_crs sox_ok
      FROM tablename;
    
    ```    
* Table aliases and column aliases should be descriptive.
  Much like variable names, "a", "b", "x", etc are not generally useful in and of themselves outside of short examples.

* Tiny names for table aliases can sometimes work as abbreviations.
  As an example, if "releases" is referenced frequently, it might make sense to abbreviate it "r".  However, "rel" is almost as short, and much more descriptive.  Have a good reason for "r" instead of "rel".

* Subquery aliases should be even more descriptive.
  Subqueries effectively create ad-hoc tables in memory.  As such, if you name it "x", then there's absolutely nothing to suggest the intention behind the table to a later maintainer.
  
  ```SQL
  /* Good */
  SELECT *
    FROM (SELECT table1.id AS child, 
                 table2.id AS parent
            FROM table1
            JOIN table2 ON (table2.parent_id = table1.id) ) parentage;

  /* Bad */
  SELECT *
    FROM (SELECT table1.id AS child, 
                 table2.id AS parent
            FROM table1
            JOIN table2 ON (table2.parent_id = table1.id) ) x;

  /* Bad */
  SELECT *
    FROM (SELECT table1.id AS child, 
                 table2.id AS parent
            FROM table1
            JOIN table2 ON (table2.parent_id = table1.id) ) link;
  ```

  *  Abbreviations don't help in subquery aliases
