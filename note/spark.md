# Spark
http://spark.apache.org/docs/2.3.0/

- ETL, aggregation, streaming(출제 확률 낮음) 출제
- pyspark, scala-shell 제공 (1.6 version, 2.3 version)
- Pair RDD, DataFrame, spark sql, Dstream API 숙지

## RDDs
- row level control
- schema 없는 텍스트 파일의 ETL 작업 시 사용하기 좋음
- Pair RDDs: aggregation 연산할 때 Key-Value pair 형태의 RDD로 수행하는 것이 일반적
- sc(spark context) 객체 사용

### RDDs Transformation

#### Map phase
- map: collection의 각 요소 iteration
- flatMap: map 결과인 collection을 세로로 눕힌다. ex) word count
- filter: collection의 특정 조건 요소 제거
- keyBy: key를 기준으로 (key, line) tuple 생성

  ```scala
  // 아래 두 연산 결과 같음
  rdd.keyBy(line => line.split(' ')(2))
  rdd.map(line => (line.split(' ')(2), line))
  ```

#### Reduce phase
- reduceByKey
- sortByKey: key를 기준으로 정렬
- groupByKey: key를 기준으로 collection 그룹 생성
- join: 두 rdd에서 같은 key를 기준으로 join. value가 tuple로 생성

### Practice

#### Explore RDDs

1. spark shell 실행하기

  ```bash
  pyspark       # python spark 1.6
  pyspark2      # python spark 2.3
  spark-shell   # scala spark 1.6
  spark-shell2  # scala spark 2.3
  ```

2. 텍스트파일 읽기

  ```scala
  val logsRDD = sc.textFile("/FileStore/tables/accounts.csv")
  ```

3. 파일 row count

  ```scala
  logsRDD.count()
  ```

  ```text
  res33: Long = 584699
  ```

4. 파일 내용 보기

  ```scala
  logsRDD.collect
  ```

  ```text
  res34: Array[String] = Array(186.63.149.248 - 111180 [22/Jan/2014:23:59:58 +0100] "GET /KBDOC-00288.html HTTP/1.0" 200 10678 "http://www.loudacre.com"  "Loudacre Mobile Browser Sorrento F41L", 186.63.149.248 - 111180 [22/Jan/2014:23:59:58 +0100] "GET /theme.css HTTP/1.0" 200 18109 "http://www.loudacre.com"  "Loudacre Mobile Browser Sorrento F41L", 52.38.78.242 - 61365 [22/Jan/2014:23:59:50 +0100] "GET /discounts.html HTTP/1.0" 200 4089 "http://www.loudacre.com"  "Loudacre Mobile Browser Sorrento F22L", 52.38.78.242 - 61365 [22/Jan/2014:23:59:50 +0100] "GET /theme.css HTTP/1.0" 200 10489 "http://www.loudacre.com"  "Loudacre Mobile Browser Sorrento F22L", 52.38.78.242 - 61365 [22/Jan/2014:23:59:50 +0100] "GET /code.js HTTP/1.0" 200 18673
  ```

5. `jpg`를 포함한 RDD 만들기

  ```scala
  val jpglogsRDD = logsRDD.filter(line => line.contains(".jpg"))
  ```

6. 단어 배열 RDD 만들기

  ```scala
  logsRDD.map(line => line.split(' '))
  ```

7. ip만 포함한 RDD 만들기

  ```scala
  val ipsRDD = logsRDD.map(line => line.split(' ')(0))
  ```

8. ip RDD 텍스트 파일 형식으로 저장하기

  ```scala
  ipsRDD.saveAsTextFile("/loudacre/iplist")
  ```

#### Pair RDDs

1. 각 유저 당 request 개수 세기

  ```scala
  val logsRDD = sc.textFile("/user/training/weblogs")

  val userreqs = logsRDD
    .map(line => line.split(" "))
    .map(words => (words(2), 1))
    .reduceByKey(pair => pair(0) + pair(1))
  ```

2. 유저 당 request 빈도 분포 구하기

  ```scala
  val freqcount = userreqs
    .map(pair => pair.swap)
    .countByKey()
  ```

3. 유저 계정 아이디, 계정 정보로 된 Pair RDD 만들기 (userid,[values....])

  ```scala
  val accountsRDD = sc.textFile("/loudacre/accounts")
    .map(line => line.split(","))
    .map(words => (words(0), words))
  ```

4. 유저 request 정보와 계정 정보를 Join하고 hit count merge하기

  ```scala
  val accounthits = accountsRDD.join(userreqs)
  ```

5. 처음 5개 row의 userid, hit count, first name, last name 출력하기

  ```scala
  accounthits.take(5)
    .map(pair => (pair._1, pair._2._2, pair._2._1(3), pair._2._1(4)))
    .map(pair => println(pair))
  ```

## DataFrames
- spark context 사용 (버전 별로 문법 주의)

  ```scala
  sqlContext.read // 1.6 version
  spark.read // 2.3 version
  ```

- DataFrames vs DataSets
  - Datasets: case class 객체. strongly typed. 시험에서 쓸 일 없음
  - DataFrames: Row 객체
- 문제에서 스키마가 주어진다 -> DataFrames로 푼다

### Creating DataFrames
- read: `DataFrameReader` 객체 반환
- format: csv, json, parquet(default) 파일포맷 지정
- load: source 파일 경로 지정
- csv: format('csv').load('/path')
- json: format('json').load('/path')
- table: source 테이블 명 지정

#### DataFrame Data Sources
- 시험에서는 데이터 소스를 잘 확인하고 풀어야 한다.
- text: csv, tsv, json, plain -> schema 여부, delimiter 주의
- binary: parquet, orc
- table: hive metastore, jdbc

#### DataFrames and Parquet
- parquet는 DataFrames의 기본 포맷
- parquet 파일은 binary이기 때문에 내용을 확인하려면 툴을 사용해야함
- parquet-tools
  - local directory, hdfs directory 지정 가능. (default: local. hdfs는 full path 필요. ex) hdfs://nnhost/path/to)
  - sub command: head, schema
- avro-tools도 있다

#### Define schema programmatically
- csv, tsv의 경우 스키마 없거나, 문제에서 스키마 지정이 필요할 경우 스키마 직접 정의해야함
- StructType, StructField, schema 사용

  ```scala
  import org.apache.spark.sql.types._

  val schema = StructType(
    List(
      StruuctField("name", StringType),
      StruuctField("age", IntegerType),
      StruuctField("address", StringType)
    )
  )

  spark.read
    .format("csv")
    .option("header", "false")
    .schema(schema)
    .load("/path/to/file.csv")
  ```

### DataFrame Transformation
- select: 컬럼 선택

  ```scala
  myDF.select("name", "age")
  myDF.select($"name", $"age") // column expression
  ```

- where: 조건절에 해당하는 row 선택

  ```scala
  myDF.where("age > 20")
  myDF.where($"age" > lit(20)) // column expression
  ```

- createOrReplaceTempView: 임시 테이블 생성. table로 sql 쿼리 사용 가능

  ```scala
  myDF.createOrReplaceTempView("myTable")

  spark.sql("""
  SELECT *
    FROM myTable
  """)
  ```

#### DataFrames Aggregation
- join: key 기준으로 두 DataFrames 조인. inner(default), outer, leftsemi, ...

  ```scala
  peopleDF.join(pcodesDF, $"pcode") // key 컬럼명이 같은 경우
  peopleDF.join(zcodesDF, $"pcode"===$"zip") // key 컬럼명이 다른 경우
  ```

- groupBy: key 기준으로 그룹 배열 생성 [key, values]

  ```scala
  employeeDF.groupBy("city", "state")
  employeeDF.groupBy(concat($"fname", $"lname").alias("full_name"))
  ```

- groupBy calculation: count, avg, mean, min, max

  ```scala
  val employeeGroupDF = employeeDF.groupBy("city", "state")

  employeeGroupDF.count // 그훕 내의 item들의 개수
  ```

  ```scala
  val deviceGroupDF = deviceDF
    .select("model", "temperature")
    .groupBy("model")

  deviceGroupDF.avg   // 그룹 내의 item들의 평균
  deviceGroupDF.mean  // 그룹 내의 item들의 평균
  deviceGroupDF.min   // 그룹 내의 item들의 최소값
  deviceGroupDF.max   // 그룹 내의 item들의 최대값
  ```

### Saving DataFrames
- write: `DataFrameWriter` 객체 반환
- format: csv, json, parquet(default) 파일포맷 지정
- mode: error(default), overwrite, append, ignore 저장 방법 지정
- partitionBy: partition directory로 할 column 지정 (hive에서도 그대로 partition으로 사용)
- save: 저장 경로 지정
- saveAsTable: 저장 데이터 저장

## Streaming
- 출제된적 없지만, 만약 출제되면 DStream 문제
- 데이터 스트림으로 소켓 사용할 가능성 있음
- DStream 객체는 DataFrame api와 비슷하지만 다를 수 있음. 그 땐 `transform`, `foreachRDD` 메소드 사용할 것

  ```scala
  val distinctDS = myDS.transform(rdd => rdd.distinct())
  ```

  ```scala
  myDS.foreachRDD((rdd,time) => {
    ...
  })
  ```

- text file 저장 시 DStream 객체마다 다른 파일에 저장

  ```scala
  userreqs.saveAsTextFiles("/path/to/prepath")

  // /path/to/prepath/part-00000.....
  ```

### Processing multiple batch
- Slice: consist of a series of "batches" of data
- State: cumulative
- Windows: aggregate across sliding time period
- lineage? -> RDD가 실제 action되면 RDD dependency graph를 따라 시작점부터 실제 연산 시작.
- checkpoint? -> lineage 기반으로 모든 streaming을 하면 datasource가 영원히 필요. 이를 해결하기 위해 durable storage에 시작점을 저장.
- int array summation

  ```scala
  counts.foldLeft(0)(_+_) // count 배열을 더하는데, 첫 번째 원소는 0이랑 더하기
  counts.foldLeft(1)(_*_) // count 배열을 곱하는데, 첫 번째 원소는 1이랑 곱하기
  ```

- window functions

  ```scala
  // DStream 간격이 2초라고 가정할 때.

  reduceByKeyAndWindow(fn, Seconds(12))
  // window size가 12초다. => 6개 interval 지나감

  reduceByKeyAndWindow(fn, Seconds(12), Seconds(4))
  // window size 12초 짜리 데이터에 대해서 4초마다 한 번 누적 계산 한다.
  // 6개 interval짜리 데이터를 4초에 한 번씩 reduce해서 window DStream에 저장
  ```
