


And Run the Query in Redshift:  
  
```  
SELECT account_id, COUNT(*) as count  
FROM billing_segments  
WHERE from_date = :from_date -- '2026-06-30 15:00:00'
AND type NOT LIKE 'pricing_%'  
GROUP BY account_id  
HAVING count < 23;
```  
  
Run the next query for Feature Usage on Redshift to identifiy records in the 'feature_usages' table that are missing usage data for one or more days within a specific month/date range.:  
  
Review and modify the start_date, end_date and target_from_date  
  
```  
-- start_date yyyy-mm-dd local  '2026-01-01'
-- end_date yyyy-mm-dd local  '2026-01-31'
-- start_datetime '2025-12-31 15:00:00'  
WITH RECURSIVE  
params AS (  
  SELECT :start_date::date               AS start_date,  
         :end_date::date               AS end_date,  
         :start_datetime::timestamp AS target_from_date  
),  
date_series (day_dt, day_str) AS (  
  SELECT p.start_date,  
         CAST(TO_CHAR(p.start_date, 'YYYY-MM-DD') AS VARCHAR(10))  
  FROM params p  
  UNION ALL  
  SELECT CAST(DATEADD(day, 1, ds.day_dt) AS DATE),  
         CAST(TO_CHAR(DATEADD(day, 1, ds.day_dt), 'YYYY-MM-DD') AS VARCHAR(10))  
  FROM date_series ds  
  WHERE ds.day_dt < :end_date::date  
)  
SELECT f.*  
FROM feature_usages f, params p  
WHERE f.from_date = p.target_from_date  
  AND EXISTS (  
        SELECT 1  
        FROM date_series d  
        WHERE STRPOS(NVL(f.daily_usage, ''), ' to ' || d.day_str) = 0  
      )  
ORDER BY f.account_id;
```


### Send message to
https://comvex.slack.com/archives/C04064TEBK9
```
[@Azusa Kitsu / 吉津 梓](https://comvex.slack.com/team/U0489EW4G5C) [@Takeo Nogami / 野上 健雄](https://comvex.slack.com/team/U02ACTAMBNC)  
お疲れ様です。  
7月分の請求データにつきまして、こちらで確認したところ、特に差異や問題は見当たりませんでした。  
こちらの認識では問題ございませんので、本日このままデータを進めていただいて大丈夫です。  
もし何かお気づきの点やご不明な点がございましたら、お気軽にご連絡ください。  
よろしくお願いいたします。
```


