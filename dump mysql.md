дамп базы данных MySQL
# сделать дамп >
mysqldump -u sar-bc -p tsnzv_v2 > tsnzv_v2.sql
   
# загрузить дамп <
mysqldump -u sar-bc -p tsnzv_v2 < tsnzv_v2.sql
   
