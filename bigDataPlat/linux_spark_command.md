# linux、spark等命令

***

记录用到的一些命令

# 以前命令

***

```shell
/**合并文件为新文件*/
cat predictions_result.csv/part-0000* > predictions_result.csv/result.csv
/**编译java程序*/
javac -Djava.ext.dirs=/home/houlinli/prj/lib/ RecommenderSystems.java
/**运行java程序*/   
java -Djava.ext.dirs=/home/houlinli/prj/lib/ RecommenderSystems
/**挂载windows文件夹*/
mount -t cifs -o nounix,noserverino,username="lw_coming@163.com",password="ff472386403" //10.3.9.155/weatherDisplay  /mnt/lw
/**卸载挂载文件*/
# umount /mnt/lw
/**shell中if[-z "${SPARK_HOME}"]意思*/
-z代表是否有值
/**shell中$@对参数规范*/
$*同$@: 对输入参数以空格分割成不同字符串
"$*": 输入参数当作一个字符串
"$#": 参数个数
/**yarn kill*/
yarn application -kill application_1475912079524_0018
/**yarn log*/
yarn logs -applicationId application_1475912079524_0018
/**spark submit*/
spark-submit --class app.AllInfo --master yarn-cluster  --jars /home/lw/myapp/mylibs/spark_hbase_fat.jar /home/lw/myapp/weatherDisplay.jar

spark-submit --class app.UserInfoCountSql --master local[4]  --jars /home/lw/myapp/mylibs/spark_hbase_fat.jar,/home/houlinli/prj/lib/spark-csv_2.10-1.5.0.jar,/home/houlinli/prj/lib/commons-csv-1.4.jar  /home/lw/myapp/UserInfoCountSql.jar
/**sql*/
SELECT * FROM user_log_format1 as B INNER JOIN (SELECT DISTINCT rating FROM (SELECT * from user_log_format1 WHERE user_id=328862)as A) as C on C.rating=B.rating where B.user_id=328862 
```