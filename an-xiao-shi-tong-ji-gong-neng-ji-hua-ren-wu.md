## 历史表统计计划任务

在前面有一张历史表，也就是ammeter\__history_表_，_它是对_ammeter_\_real按照小时和电表号分组统计得到的数据，在每个小时都应该定时去查找_ammeter_\_real表，将过去的一个小时的数据分组得到结果插入到ammeter\__history_表中，所以需要一个计划任务每隔一个小时执行一个存储过程，存储过程的功能是分组统计，并插入ammeter\__history_表中。

* ### 按小时统计存储过程

创建一个带输入参数的存储过程grouptimer，该存储过程主要是对电表的实时表ammeter\_real的数据按照某个小时进行分组得到所有电表某个小时的pat最小值和pat的最大值等信息插入到电表的历史表ammeter\_history中，

```
BEGIN
-- 需要定义接收游标数据的变量 
  DECLARE ammater_id VARCHAR(255);
    DECLARE pat_min FLOAT default 0;
    DECLARE pat_max FLOAT default 0;

 -- 遍历数据结束标志
  DECLARE done INT DEFAULT FALSE;

 -- 游标
    DECLARE cur CURSOR FOR 
        SELECT e_num,MIN(pat),MAX(pat) FROM ammeter_real 
            WHERE DATE_FORMAT(read_time,'%Y-%m-%d %H')=hourformat(datehour) GROUP BY 
            DATE_FORMAT(read_time,'%Y-%m-%d %H'),ammater_id;

 -- 将结束标志绑定到游标
   DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

    -- 打开游标
    OPEN  cur;     
    -- 遍历
    read_loop: LOOP
            -- 取值 取多个字段
            FETCH  NEXT from cur INTO ammater_id,pat_min,pat_max;
            IF done THEN
                LEAVE read_loop;
             END IF;
                SELECT ammater_id,pat_min,pat_max;
        -- 你自己想做的操作
       INSERT INTO ammeter_history (ammater_id,pat_min,pat_max,read_time) VALUES 
         (ammater_id,pat_min,pat_max,DATE_FORMAT(datehour,'%Y-%m-%d %H'));

    END LOOP;

    CLOSE cur;

END
```



