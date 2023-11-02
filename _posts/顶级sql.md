---
typora-root-url: ..
typora-copy-images-to: ../img/posts	
---

![image-20210622145510809](/img/posts/image-20210622145510809.png)

```sql
SELECT 
CABIN_LV_NAME,
GROUP_CONCAT(bunkrange order by CAST(left(bunkrange,3) as SIGNED) ) as bunkrange,
SUM(sum) as sum
	FROM(
SELECT
CABIN_LV_NAME,
CASE START_VALUE
 WHEN MAX(BUNK_NO)  THEN
  START_VALUE
 ELSE
  CONCAT(START_VALUE,"-",MAX(BUNK_NO)) 
END as bunkrange,

COUNT(1) as sum
FROM
 (SELECT
  BUNK_NO,CABIN_LV_NAME,
 CASE
   WHEN @PREV = BUNK_NO - 1 
   THEN @START_VALUE 
   ELSE @START_VALUE := BUNK_NO 
    END 
   AS START_VALUE,
  @PREV := BUNK_NO 
 FROM
 (SELECT
  CABIN_LV_NAME,CAST( BUNK_NO AS SIGNED ) AS BUNK_NO 
 FROM
  bk_ticket 
 WHERE
  FK_PLAN =  '${planId}'
  AND TICKET_TYPE_CODE = 'KP' 
  AND STATUS IN ( '1', '4' ) 
 ORDER BY
  CABIN_LV_NAME ASC,BUNK_NO ASC 
 ) AS C,(
  SELECT
   @PREV := NULL,
   @START_VALUE := 0 
  ) AS BUNK_NO 
 ) AS B 
GROUP BY
CABIN_LV_NAME,START_VALUE
order by 
bunkrange asc
 )s
 GROUP BY CABIN_LV_NAME
  ORDER BY FIELD(CABIN_LV_NAME,'座席','四等','三等','三等B','三等A','二等B','二等A','一等','特等') DESC
```

