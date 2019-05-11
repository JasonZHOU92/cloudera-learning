notes:

1. Impala supported query with specific columns: save as orc?

2. Save your data eaay to query by day, and month: partition keys?

3. string manipulation in hive: replace a string, clean up a field

4. sqoop import: please type rather than copy paste.

5. stored for a long time and compability required: avro?

6. convert DF to rdd: why the heal is it not working as expected?
val df = sqlContext.sql("select * from customers")
val rdd = df.rdd \\the data has a brace

7. regex: when fields are seperated with comma, and a field contains comma itself. How to separate them with tab

8. fast internet connection and fluent work under linux environment

9. fastest compression and decompression algorithm: Snappy
highest compression rate: Gzip

10. use hive when dealing with tables, because sparks are really slow
check the source schema and target requirement before continuing

11. store binary record to hadoop (save-as-sequence? or what)

12. be really familiar with data import and export

13. Have deeper understanding of data format and compression algorithm

14. No questions for spark streaming/kafka, so don't study for that part


Problem 1:
some customer data in hive table, a column is city which sometimes start with ("Mtn", "Natl", "Pk"), you need to replace them with ("Montain", "National", "Park")

use hive case

Problem 2:
the same customer data, the telepone number is (0728-5240538), the result should only contain numbers

use hive replace regex

Problem 3:
some customer data, you need to read it from hdfs, change the delimiter, and write back with certain compression, and data format

Problem 4:
some customer data in mysql, you need to import it into hadoop. to certain format and compression algorithm

Problem 5:
Some data in hdfs, you need to load it to spark and computer the top 5 whatsoever

Problem 6:
Some customer data in hadoop/hive. The date is delimitered with comma, meanwhile the address field contains comma itself. You need to delimieterd the data with tab, meanwhile keep the same schema. The data is in hive/hadoop

Problem 7:
Optimize a query for impala. Some whatsoever data with two freqence query:
select * from balbalba where month=bla and day =bla;
select * from blablabla where month=bla;

Some deadline query at the end of month. So it should be quick and optimzed for previous queries.

Problem 8:
Inject Logs under certain dir to hadoop
use flume spoolDir
