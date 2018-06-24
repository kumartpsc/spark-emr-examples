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
