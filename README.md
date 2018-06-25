# spark-emr-examples

### Prerequisites
1. Create AWS Account
2. Create EMR cluster
3. Make sure EMR cluster has access to S3.

### First example
1. Connect to cluster using below command
   ssh -i ./mykey.pem hadoop@hostname.compute.amazonaws.com

2. run spark-shell command
3. load the file from s3 bucket, make sure that you uploaded file to S3 before execting this command
   val file = sc.textFile("s3://[your-bucket-name]/your-textfile.txt")
   
4. Run below command to count words in file
   val counts = file.flatMap(line => line.toLowerCase().replace(".", " ").replace(",", " ").split(" ")).map(word => (word, 1L)).reduceByKey(_ + _)
   
5. Sort and take first 10 words
   val sorted_counts = counts.collect().sortBy(wc => -wc._2)
   sorted_counts.take(10).foreach(println)
6. Finally save the file to S3
   sc.parallelize(sorted_counts).saveAsTextFile("s3://bucket/wordcount-kids-story")
   
   
## EMR cluster creation
Instructions:
1. create emr cluster
2. download .pem file and save it somewhere
3. setup foxyproxy tunnel, first install plugin in chrome
4. create policy-settings file and import that into foxyproxy
5. open different monitors: https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-web-interfaces.html
6. connect to cluster using
   ssh -i ./mykey.pem hadoop@nodename.amazonaws.com

### Zeppelin notebook for wordcount

### Zeppelin notebook for sparksql
import org.apache.commons.io.IOUtils
import java.net.URL
import java.nio.charset.Charset

// Zeppelin creates and injects sc (SparkContext) and sqlContext (HiveContext or SqlContext)
// So you don't need create them manually

// load bank data
val bankText = sc.parallelize(
    IOUtils.toString(
        new URL("https://s3.amazonaws.com/apache-zeppelin/tutorial/bank/bank.csv"),
        Charset.forName("utf8")).split("\n"))

case class Bank(age: Integer, job: String, marital: String, education: String, balance: Integer)

val bank = bankText.map(s => s.split(";")).filter(s => s(0) != "\"age\"").map(
    s => Bank(s(0).toInt, 
            s(1).replaceAll("\"", ""),
            s(2).replaceAll("\"", ""),
            s(3).replaceAll("\"", ""),
            s(5).replaceAll("\"", "").toInt
        )
).toDF()
bank.registerTempTable("bank")

## Query 
%sql 
select age, count(1) value
from bank 
where age <25
group by age 
order by age

## Query2
%sql 
select age, count(1) value 
from bank 
where age < ${maxAge=35} 
group by age 
order by age

## Query3
%sql 
select age, count(1) value 
from bank 
where marital="${marital=single,single|divorced|married}" 
group by age 
order by age


### Wordcount POC
val file = sc.textFile("s3://emr-poc-3-s3/kids-story.txt")
val counts = file.flatMap(line => line.toLowerCase().replace(".", " ").replace(",", " ").split(" ")).map(word => (word, 1L)).reduceByKey(_ + _)
val sorted_counts = counts.collect().sortBy(wc => -wc._2)
sorted_counts.take(10).foreach(println)

### AWS Ports:
Zeppelin: 8890
Spark RM: 8088

Name of interface 	

URI
YARN ResourceManager 	http://master-public-dns-name:8088/
YARN NodeManager 	http://slave-public-dns-name:8042/
Hadoop HDFS NameNode 	http://master-public-dns-name:50070/
Hadoop HDFS DataNode 	http://slave-public-dns-name:50075/
Spark HistoryServer 	http://master-public-dns-name:18080/
Zeppelin 	http://master-public-dns-name:8890/
Hue 	http://master-public-dns-name:8888/
Ganglia 	http://master-public-dns-name/ganglia/
HBase UI 	http://master-public-dns-name:16010/ 

