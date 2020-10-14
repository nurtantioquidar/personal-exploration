# Cloud Dataprocs



### General Information

A fast, easy-to-use, fully-managed cloud service for running the Apache Spark and Apache Hadoop ecosystem on Google Cloud Platform. Dataproc is a complete platform for data processing, analytics, and machine learning. Dataproc offers per-second billing, so you only pay for exactly the resources you consume. 

### Apache Spark

A unified analytics engine for large-scale data processing, used by well-known, modern enterprises, such as Netflix, Yahoo, and eBay. With in-memory speeds up to 100x faster than Hadoop, Apache Spark achieves high performance for static, batch, and streaming data, using a state-of-the-art DAG ([Directed Acyclic Graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph)) scheduler, a query optimizer, and a physical execution engine. Spark also available as an execution engine in Hive so the query can run on top of Spark instead of Map Reduce.

### Pyspark

The Spark Python API, PySpark, exposes the Spark programming model to Python. PySpark is built on top of Spark’s Java API. Data is processed in Python and cached and shuffled in the JVM. According to [Apache](https://cwiki.apache.org/confluence/display/SPARK/PySpark+Internals), Py4J enables Python programs running in a Python interpreter to dynamically access Java objects in a JVM.

### Setup

- Using Local
  We are about to use Python to run Spark then we use Pyspark as the Spark API and also using conda environment to make it easy at the first time.
  Install `pyspark` using conda:

  ```shell
  conda install -c conda-forge pyspark
  ```

  Here is the folder structure for the projects:

  ```shell
  .
  ├── data
  │   ├── ibrd-statement-of-loans-latest-available-snapshot.csv
  │   └── ibrd-summary-small-python
  ├── international_loans_dataproc.py
  ├── international_loans_local.py
  └── notes.md
  ```

  - `data`: where the input and output of the process placed
  - `ibrd-statement-of-loans-latest-available-snapshot.csv`: data source
  - `international_loans_dataproc.py`: pyspark Dataprocs scripts
  - `international_loans_local.py`: pyspark local scripts

  Pyspark local scripts:

  ```python
  #!/usr/bin/python
  
  from pyspark.sql import SparkSession
  
  
  def main():
      spark = SparkSession \
          .builder \
          .master("local[*]") \
          .appName('dataproc-python-demo') \
          .getOrCreate()
  
      # Defaults to INFO
      sc = spark.sparkContext
      sc.setLogLevel("INFO")
  
      # Loads CSV file from local directory
      df_loans = spark \
          .read \
          .format("csv") \
          .option("header", "true") \
          .option("inferSchema", "true") \
          .load("data/ibrd-statement-of-loans-latest-available-snapshot.csv")
  
      # Prints basic stats
      print("Rows of data:" + str(df_loans.count()))
      print("Inferred Schema:")
      df_loans.printSchema()
  
      # Creates temporary view using DataFrame
      df_loans.withColumnRenamed("Country", "country") \
          .withColumnRenamed("Country Code", "country_code") \
          .withColumnRenamed("Disbursed Amount", "disbursed") \
          .withColumnRenamed("Borrower's Obligation", "obligation") \
          .withColumnRenamed("Interest Rate", "interest_rate") \
          .createOrReplaceTempView("loans")
  
      # Performs basic analysis of dataset
      df_disbursement = spark.sql("""
      SELECT country, country_code,
              format_number(total_disbursement, 0) AS total_disbursement,
              format_number(ABS(total_obligation), 0) AS total_obligation,
              format_number(avg_interest_rate, 2) AS avg_interest_rate
              FROM (
              SELECT country, country_code,
              SUM(disbursed) AS total_disbursement,
              SUM(obligation) AS total_obligation,
              AVG(interest_rate) AS avg_interest_rate
              FROM loans
              GROUP BY country, country_code
              ORDER BY total_disbursement DESC
              LIMIT 25)
      """).cache()
  
      df_disbursement.show(25, True)
  
      # Saves results to a locally CSV file
      df_disbursement.repartition(1) \
          .write \
          .mode("overwrite") \
          .format("csv") \
          .option("header", "true") \
          .save("data/ibrd-summary-small-python")
  
      print("Results successfully written to CSV file")
  
      spark.stop()
  
  
  if __name__ == "__main__":
      main()
  
  ```

  The ideas about this script are doing simple transformation using SparkSQL and also producing output file in CSV then put it in certain directory.

  Execute the script using:

  ```shell
  (py-env) user@machine: rootdir/demo-script$ python international_loans_local.py
  ```

  That execution will produce this in console:
  ![image-20201014093329985](/home/nakama/.config/Typora/typora-user-images/image-20201014093329985.png)
  and also pay attention to `data` folder so we can see result of the execution script:

  ```shell
  .
  ├── data
  │   ├── ibrd-statement-of-loans-latest-available-snapshot.csv
  │   └── ibrd-summary-small-python
  │       ├── part-00000-4b6aa13b-ce5f-4845-8d94-068bd82ebfff-c000.csv
  │       └── _SUCCESS
  ├── international_loans_dataproc.py
  ├── international_loans_local.py
  └── notes.md
  ```

  Now we have file `part-00000-4b6aa13b-ce5f-4845-8d94-068bd82ebfff-c000.csv` (keep in mind that the file name format is like `part-{num_of_part}-{hash-id-format}` then the file name my differ) that contains same result as show from console:

  ```markdown
  country,country_code,total_disbursement,total_obligation,avg_interest_rate
  Brazil,BR,"52,148,131,649","16,266,149,193",3.85
  Mexico,MX,"48,916,191,845","14,614,571,327",4.68
  Indonesia,ID,"41,753,450,934","17,239,319,642",4.42
  China,CN,"39,912,259,322","14,077,440,498",3.06
  India,IN,"38,381,308,887","13,294,547,027",3.62
  Turkey,TR,"34,352,317,563","11,700,925,796",4.64
  Argentina,AR,"27,780,826,064","6,151,485,449",3.14
  Colombia,CO,"22,779,859,781","10,180,019,349",4.19
  Philippines,PH,"15,200,182,963","5,677,110,957",5.21
  "Korea, Republic of",KR,"14,918,461,307",0,6.78
  Poland,PL,"14,515,467,735","7,772,319,837",2.71
  Morocco,MA,"13,869,492,906","5,436,716,723",4.06
  "Egypt, Arab Republic of",EG,"13,308,806,309","9,180,370,728",4.02
  Romania,RO,"11,840,320,192","4,605,243,727",4.50
  Russian Federation,RU,"10,410,895,337","459,849,411",2.17
  Ukraine,UA,"9,015,803,013","5,244,776,219",2.02
  Tunisia,TN,"8,781,853,773","3,473,696,381",4.23
  Peru,PE,"8,195,794,379","1,101,792,154",3.56
  Thailand,TH,"8,151,817,580","1,017,562,225",5.95
  Pakistan,PK,"7,507,491,566","1,347,596,718",3.70
  Kazakhstan,KZ,"6,589,686,095","4,003,423,634",2.96
  Nigeria,NG,"5,438,632,035","124,179,696",5.88
  Serbia,YF,"4,702,132,013","2,442,336,979",4.57
  Jordan,JO,"4,439,107,606","2,181,499,670",3.30
  Algeria,DZ,"4,306,515,350",0,5.22
  ```

  

- Using Dataproc Cluster
  Setup Dataproc cluster using certain command (in general):

  ```shell
  gcloud dataproc clusters create example-cluster --scopes sqlservice,bigquery
  ```

  Specific command that define custom requirements like machine type, worker size, network, etc:

  ```shell
  gcloud dataproc clusters create mc-cluster --zone us-central1-a \
  	--master-machine-type n1-standard-1 --master-boot-disk-size 50 \
  	--num-workers 2 --worker-machine-type n1-standard-1 \
  	--worker-boot-disk-size 50 --network=default \
  ```

  So in cluster creation, we have an option that will allow us to connect cluster with some services in GCP:

  ```shell
  --scopes sqlservice,bigquery
  ```

   If the `--scopes` flag is not specified,  the default scopes are:

  ```tex
    https://www.googleapis.com/auth/bigquery
    https://www.googleapis.com/auth/bigtable.admin.table
    https://www.googleapis.com/auth/bigtable.data
    https://www.googleapis.com/auth/devstorage.full_control
  ```

  Pyspark Dataproc script:

  ```python
  #!/usr/bin/python
  
  from pyspark.sql import SparkSession
  import sys
  
  
  def main(argv):
      storage_bucket = argv[0]
      data_file = argv[1]
      results_directory = argv[2]
  
      print "Number of arguments: {0} arguments.".format(len(sys.argv))
      print "Argument List: {0}".format(str(sys.argv))
  
      spark = SparkSession \
          .builder \
          .master("yarn") \
          .appName('dataproc-python-demo') \
          .getOrCreate()
  
      # Defaults to INFO
      sc = spark.sparkContext
      sc.setLogLevel("WARN")
  
      # Loads CSV file from Google Storage Bucket
      df_loans = spark \
          .read \
          .format("csv") \
          .option("header", "true") \
          .option("inferSchema", "true") \
          .load(storage_bucket + "/" + data_file)
  
      # Creates temporary view using DataFrame
      df_loans.withColumnRenamed("Country", "country") \
          .withColumnRenamed("Country Code", "country_code") \
          .withColumnRenamed("Disbursed Amount", "disbursed") \
          .withColumnRenamed("Borrower's Obligation", "obligation") \
          .withColumnRenamed("Interest Rate", "interest_rate") \
          .createOrReplaceTempView("loans")
  
      # Performs basic analysis of dataset
      df_disbursement = spark.sql("""
      SELECT country, country_code,
              format_number(total_disbursement, 0) AS total_disbursement,
              format_number(ABS(total_obligation), 0) AS total_obligation,
              format_number(avg_interest_rate, 2) AS avg_interest_rate
              FROM (
              SELECT country, country_code,
              SUM(disbursed) AS total_disbursement,
              SUM(obligation) AS total_obligation,
              AVG(interest_rate) AS avg_interest_rate
              FROM loans
              GROUP BY country, country_code
              ORDER BY total_disbursement DESC
              LIMIT 25)
      """).cache()
  
      print "Results:"
      df_disbursement.show(25, True)
  
      # Saves results to single CSV file in Google Storage Bucket
      df_disbursement.write \
          .mode("overwrite") \
          .format("parquet") \
          .save(storage_bucket + "/" + results_directory)
  
      spark.stop()
  
  
  if __name__ == "__main__":
      main(sys.argv[1:])
  
  ```

  There are some different between Dataprocs and local script such as:

  - When defining spark object in local we are using local resources, but in Dataproc we are using `yarn` as resource manager.

    ```python
    # local
    spark = SparkSession \
            .builder \
            .master("local[*]") \
            .appName('dataproc-python-demo') \
            .getOrCreate()
    
    # dataproc
    spark = SparkSession \
            .builder \
            .master("yarn") \
            .appName('dataproc-python-demo') \
            .getOrCreate()
    ```

  - In local, we are storing it in CSV format yet in GCS bucket we are storing it in `parquet` with `snappy` compression:

    ```python
    # local
    df_disbursement.repartition(1) \
            .write \
            .mode("overwrite") \
            .format("csv") \
            .option("header", "true") \
            .save("data/ibrd-summary-small-python")
            
    # dataproc
    df_disbursement.write \
            .mode("overwrite") \
            .format("parquet") \
            .save(storage_bucket + "/" + results_directory)
    ```

  Uploading script resources on Google Cloud environment:
  ![image-20201014100157625](/home/nakama/.config/Typora/typora-user-images/image-20201014100157625.png) 

  Submit job via UI console:
  ![image-20201014100606396](/home/nakama/.config/Typora/typora-user-images/image-20201014100606396.png)

  or submit via REST:

  ```json
  POST /v1/projects/tokopedia-staging-198806/regions/asia-southeast1/jobs:submit/
  {
    "projectId": "tokopedia-staging-198806",
    "job": {
      "placement": {
        "clusterName": "testing-de"
      },
      "reference": {
        "jobId": "job-c71851b1"
      },
      "pysparkJob": {
        "mainPythonFileUri": "gs://dataengineer-staging-bucket/dataproc/international_loans_dataproc.py",
        "args": [
          "gs://dataengineer-staging-bucket/dataproc",
          "ibrd-statement-of-loans-latest-available-snapshot.csv",
          "result-summary"
        ]
      }
    }
  }
  ```

  That submit command meaning Dataprocs will 

  1. execute PySpark from `gs://dataengineer-staging-bucket/dataproc/international_loans_dataproc.py`
  2. reading csv file from `gs://dataengineer-staging-bucket/dataproc` directory with file named `ibrd-statement-of-loans-latest-available-snapshot.csv`
  3. produce the result in `parquet` format in `gs://dataengineer-staging-bucket/dataproc/result-summary`

  Then we can see the log:
  ![image-20201014100748897](/home/nakama/.config/Typora/typora-user-images/image-20201014100748897.png)

  The result:
  ![image-20201014100825768](/home/nakama/.config/Typora/typora-user-images/image-20201014100825768.png)

  Then we can view the content of the result file using Bigquery:
  ![image-20201014101152500](/home/nakama/.config/Typora/typora-user-images/image-20201014101152500.png)



### Conclusion

In this post, we have seen basic use case using Spark and also creating one using python. We did some transformation using SparkSQL then produce some output file in certain file format (CSV and Parquet) by running it either in local and using Dataproc cluster. 
