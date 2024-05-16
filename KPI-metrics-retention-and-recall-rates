WITH deliveryman_retained AS 
(
SELECT  t1.*
        ,ttt1.last_recall_time
        ,ttt1.last_recall_date
        ,ttt1.last_recall_order_sn
        ,ttt1.recall_is_W1_retained
        ,ttt1.recall_is_W2_retained
FROM    (
            —- Deliveryman's first run retention decision
            SELECT  deliveryman_id
                    ,deliveryman_name
                    ,traffic_means    
                    ,country
                    ,city_name   
                    ,first_delivered_order_receive_time    -- First run time
                    ,substr(first_delivered_order_receive_time,1,10) first_delivered_order_receive_date    -- First run date
                    ,if(
                        completed_order_num_2nd_day+completed_order_num_3rd_day+completed_order_num_4th_day+completed_order_num_5th_day+completed_order_num_6th_day+completed_order_num_7th_day+completed_order_num_8th_day>0
                        ,1
                        ,0
                    ) is_W1_retained    -- Whether the first week of retention after the first run
                    ,if(
                        completed_order_num_2nd_day+completed_order_num_3rd_day+completed_order_num_4th_day+completed_order_num_5th_day+completed_order_num_6th_day+completed_order_num_7th_day+completed_order_num_8th_day>0 AND completed_order_num_9th_day+completed_order_num_10th_day+completed_order_num_11th_day+completed_order_num_12th_day+completed_order_num_13th_day+completed_order_num_14th_day+completed_order_num_15th_day>0
                        ,1
                        ,0
                    ) is_W2_retained    -- Whether to stay for the second week after the first run
            FROM    deliveryman
            WHERE   account_type = 'Regular Account'
            OR      account_type IS NULL
        ) t1
LEFT join (
              -- Retention after 14 days of recall
              SELECT  tt1.deliveryman_id
                      ,tt1.last_recall_time
                      ,tt1.last_recall_date
                      ,tt1.last_recall_order_sn
                      ,if(
                          sum(
                              CASE    WHEN DATEDIFF(to_date(tt2.stat_date),to_date(tt1.last_recall_date),'dd') <= 7 THEN tt2.delivered_order_num 
                              END
                          ) >0
                          ,1
                          ,0
                      ) recall_is_W1_retained   -- Whether to stay after the first week of recall                      ,if(
                          sum(
                              CASE    WHEN DATEDIFF(to_date(tt2.stat_date),to_date(tt1.last_recall_date),'dd') <= 7 THEN tt2.delivered_order_num 
                              END
                          ) >0 AND sum(CASE WHEN DATEDIFF(to_date(tt2.stat_date),to_date(tt1.last_recall_date),'dd') > 7 AND DATEDIFF(to_date(tt2.stat_date),to_date(tt1.last_recall_date),'dd') <= 14 THEN tt2.delivered_order_num END) >0
                          ,1
                          ,0
                      ) recall_is_W2_retained   -- Whether the second week after the recall to stay
              FROM    (
                          -- The order receiving time and the previous order receiving time > 30 days, is a recall order, the rider is a recall rider
                          SELECT  t1.deliveryman_id
                                  ,max(t1.deliveryman_receive_order_time) last_recall_time
                                  ,substr(max(t1.deliveryman_receive_order_time),1,10) last_recall_date
                                  ,arg_max(t1.deliveryman_receive_order_time,t1.order_sn) last_recall_order_sn
                          FROM    (
                                      -- The time of this order and the time of the last order
                                      SELECT  order_sn
                                              ,deliveryman_id
                                              ,deliveryman_receive_order_time
                                              ,lead(deliveryman_receive_order_time,1) over (PARTITION BY deliveryman_id ORDER BY deliveryman_receive_order_time DESC) before_deliveryman_receive_order_time
                                      FROM    order
                                      WHERE   delivery_type = 'Deliveryman Dispatch'
                                      -- and   substr(delivered_time,1,10) = '2022-05-22'
                                      -- and   deliveryman_id = '575010'
                                      AND     delivered_time IS NOT NULL
                                  ) t1
                          WHERE   datediff(to_date(substr(t1.deliveryman_receive_order_time,1,10)), to_date(substr(t1.before_deliveryman_receive_order_time,1,10)) ,'dd') > 30    --This order taking time
                          GROUP BY t1.deliveryman_id
                      ) tt1
              LEFT JOIN deliveryman_daily tt2
              ON      tt1.deliveryman_id = tt2.deliveryman_id
              AND     tt2.stat_date > tt1.last_recall_date
              AND     DATEDIFF(to_date(tt2.stat_date),to_date(tt1.last_recall_date),'dd') <= 14
              GROUP BY tt1.deliveryman_id
                       ,tt1.last_recall_time
                       ,tt1.last_recall_date
                       ,tt1.last_recall_order_sn
          ) ttt1
ON      t1.deliveryman_id = ttt1.deliveryman_id

)

-- Retention of first run riders in the first 15 days
SELECT 
city_name
,COUNT(deliveryman_id) Number of first run drivers
,COUNT(deliveryman_name) Number of retained drivers on 14 days
,"First run” Category
from deliveryman_retained 
where first_delivered_order_receive_date >= '2024-08-08'  
and   first_delivered_order_receive_date <= '2024-08-14'  --Data of the first 15 days
and   country = "AU"
and    (is_W2_retained = 1)
group by 
city_name

union all

-- Retention of recalled riders in the first 15 days
SELECT 
city_name
,COUNT(deliveryman_id) Number of first run drivers
,COUNT(deliveryman_name) Number of retained drivers on 14 days
,"Recall”
from deliveryman_retained 
where last_recall_date >= '2024-08-04'  
and   last_recall_date <= '2024-08-14'   --Data of the first 15 days
and   country = "AU"
and   (recall_is_W2_retained = 1)
group by 
city_name
order by 
city_name
;
