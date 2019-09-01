# Sqoop
http://archive.cloudera.com/cdh5/cdh/5/sqoop-1.4.6-cdh5.15.0/

- mysql import/export 문제
- format, compression, delimeter, directory 옵션 숙지
- tool 옵션이 헷갈리면 `sqoop [tool] --help` 를 활용

## Tools

- list-tables: 테이블 목록 조회
- eval: sql 쿼리 수행
- import: 데이터 가져오기

  * --target-dir: hdfs 저장 경로 지정
  * --columns: 가져올 컬럼 지정 ex) `--columns "col1,col2,col3"`
  * --fields-terminated-by: delimiter 지정
  * --compression-codec: 압축 코덱 지정 ex) `--compression-coded org.apache.hadoop.io.compress.SnappyCodec`
  * --as-textfile: 텍스트 파일 포맷 지정 (default)
  * --as-parquetfile: parquet 파일 포맷 지정
  * --as-avrodatafile: avro 파일 포맷 지정
  * --split-by: split 키 지정 (primary key 없을 경우 필수)

- export: 데이터 내보내기 (Mysql 테이블이 만들어져 있어야 함)

  * --export-dir: import와 마찬가지
  * --update-mode: 이미 존재하는 row 처리 정책. (allowinsert, update)
  * --table: 테이블 지정
  * --import-fields-terminated-by: --fields-terminated-by로 해도 정상 작동하는 버그.

## Practice

### list-tables

1. 테이블 목록 조회

  ```bash
  sqoop list-tables \
  --connect jdbc:mysql://localhost/loudacre \
  --username training \
  --password training
  ```

  ```text
  accountdevice
  accounts
  basestations
  customerservicerep
  device
  knowledgebase
  mostactivestations
  webpage
  ```

### eval

1. accounts 테이블 schema 확인

  ```bash
  sqoop eval \
  --query "describe accounts" \
  --connect jdbc:mysql://localhost/loudacre \
  --username training \
  --password training
  ```

  ```text
  ---------------------------------------------------------------------------------------------------------
  | Field                | Type                 | Null | Key | Default              | Extra                | 
  ---------------------------------------------------------------------------------------------------------
  | acct_num             | int(11)              | NO  | PRI | (null)               |                      | 
  | acct_create_dt       | datetime             | NO  |     | (null)               |                      | 
  | acct_close_dt        | datetime             | YES |     | (null)               |                      | 
  | first_name           | varchar(255)         | NO  |     | (null)               |                      | 
  | last_name            | varchar(255)         | NO  |     | (null)               |                      | 
  | address              | varchar(255)         | NO  |     | (null)               |                      | 
  | city                 | varchar(255)         | NO  |     | (null)               |                      | 
  | state                | varchar(255)         | NO  |     | (null)               |                      | 
  | zipcode              | varchar(255)         | NO  |     | (null)               |                      | 
  | phone_number         | varchar(255)         | NO  |     | (null)               |                      | 
  | created              | datetime             | NO  |     | (null)               |                      | 
  | modified             | datetime             | NO  |     | (null)               |                      | 
  ---------------------------------------------------------------------------------------------------------
  ```

2. accounts 테이블에서 5개 row 조회

  ```bash
  sqoop eval \
  --query "SELECT * FROM accounts LIMIT 5" \
  --connect jdbc:mysql://localhost/loudacre \
  --username training \
  --password training
  ```

  ```text
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  | acct_num    | acct_create_dt      | acct_close_dt       | first_name           | last_name            | address              | city                 | state                | zipcode              | phone_number         | created             | modified            | 
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  | 1           | 2008-10-23 16:05:05.0 | (null)              | Donald               | Becton               | 2275 Washburn Street | Oakland              | CA                   | 94660                | 5100032418           | 2014-03-18 13:29:47.0 | 2014-03-18 13:29:47.0 | 
  | 2           | 2008-11-12 03:00:01.0 | (null)              | Donna                | Jones                | 3885 Elliott Street  | San Francisco        | CA                   | 94171                | 4150835799           | 2014-03-18 13:29:47.0 | 2014-03-18 13:29:47.0 | 
  | 3           | 2008-12-21 09:19:50.0 | (null)              | Dorthy               | Chalmers             | 4073 Whaley Lane     | San Mateo            | CA                   | 94479                | 6506877757           | 2014-03-18 13:29:47.0 | 2014-03-18 13:29:47.0 | 
  | 4           | 2008-11-28 00:08:09.0 | (null)              | Leila                | Spencer              | 1447 Ross Street     | San Mateo            | CA                   | 94444                | 6503198619           | 2014-03-18 13:29:47.0 | 2014-03-18 13:29:47.0 | 
  | 5           | 2008-11-15 23:06:06.0 | (null)              | Anita                | Laughlin             | 2767 Hill Street     | Richmond             | CA                   | 94872                | 5107754354           | 2014-03-18 13:29:47.0 | 2014-03-18 13:29:47.0 | 
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  ```

### import

1. accounts 테이블 가져오기

  ```bash
  sqoop import \
  --table accounts \
  --connect jdbc:mysql://localhost/loudacre \
  --username training \
  --password training \
  --target-dir /user/training/accounts \
  -m 1
  ```

2. accounts 테이블에서 city가 "San Francisco"인 row 가져오기 (`--query` 사용)

  ```bash
  sqoop import \
  --connect jdbc:mysql://localhost/loudacre \
  --username training \
  --password training \
  --target-dir /user/training/accounts \
  --query 'SELECT * FROM accounts WHERE city="San Francisco" AND $CONDITIONS' \
  -m 1
  ```

3. accounts 테이블에서 city가 "Oakland"인 row 가져오기 (`--where` 사용)

  ```bash
  sqoop import \
  --connect jdbc:mysql://localhost/loudacre \
  --username training \
  --password training \
  --target-dir /user/training/accounts \
  --table accounts \
  --where 'city="Oakland"' \
  -m 1
  ```

4. accounts 테이블의 first name, last name, address, zipcode 컬럼 가져오기

  ```bash
  sqoop import \
  --connect jdbc:mysql://localhost/loudacre \
  --username training \
  --password training \
  --target-dir /user/training/accounts \
  --table accounts \
  --columns 'first_name,last_name,address,zipcode' \
  -m 1
  ```

5. accounts 테이블을 탭 구분자 형식으로 가져오기

  ```bash
  sqoop import \
  --connect jdbc:mysql://localhost/loudacre \
  --username training \
  --password training \
  --target-dir /user/training/accounts \
  --table accounts \
  --fields-terminated-by '\t' \
  -m 1
  ```

6. accounts 테이블을 snappy 압축 형식으로 가져오기

  ```bash
  sqoop import \
  --connect jdbc:mysql://localhost/loudacre \
  --username training \
  --password training \
  --target-dir /user/training/accounts/snappy \
  --table accounts \
  --compression-codec org.apache.hadoop.io.compress.SnappyCodec \
  -m 1
  ```

7. accounts 테이블을 parquet 파일 형식으로 가져오기

  ```bash
  sqoop import \
  --connect jdbc:mysql://localhost/loudacre \
  --username training \
  --password training \
  --target-dir /user/training/accounts/snappy \
  --table accounts \
  --as-parquetfile \
  -m 1
  ```

8. accounts 테이블을 avro 파일, snappy 압축 형식으로 가져오기

  ```bash
  sqoop import \
  --connect jdbc:mysql://localhost/loudacre \
  --username training \
  --password training \
  --target-dir /user/training/accounts/snappy \
  --table accounts \
  --as-avrodatafile \
  --compression-codec org.apache.hadoop.io.compress.SnappyCodec\
  -m 1
  ```

### export

1. accounts 파일을 Mysql 테이블로 내보내기

  ```bash
  sqoop export \
  --connect jdbc:mysql://localhost/loudacre \
  --username training \
  --password training \
  --table accounts_exported \
  --export-dir /user/training/accounts \
  --update-mode allowinsert
  ```
