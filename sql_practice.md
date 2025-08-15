### 間違いもある。
```
-- 問題1：製品ごとに最も高額な部品の名称を取得してください
select
	product_id,
	product_name,
	product_type,
	unit_price
from
	(
		select 
			product_id,
			product_name,
			product_type,
			unit_price,
			RANK() OVER(partition by product_type order by unit_price DESC) AS price_rank
		from 
			products
	)
where
	price_rank = 1
;

-- 問題2：「部品カテゴリが"締結材"」の中で、最も高い部品を使っている製品を抽出してください
select 
	PC.product_id,
	P.product_name,
	C.component_name,
	C.unit_cost
from 
	product_components PC
inner join
	products P
on
	PC.product_id = P.product_id
inner join
	components C
on
	PC.component_id = C.component_id
where
	C.category = '締結材'
order by 
	C.unit_cost 
DESC limit 1
;

-- 問題3：1つの製品にしか使われていないレア部品を抽出してください
select 
	component_id,
	component_name,
	product_id
from
	(
		select
			P.product_id,
			P.component_id,
			C.component_name,
			count(P.product_id) over(partition by P.component_id) AS COUNT_DATA
		from 
			product_components P
		inner join
			components C
		on
			P.component_id = C.component_id
	)
where
	COUNT_DATA = 1
;

-- 問題4：製品の平均単価より高い単価を持つ製品に使われている部品名を重複なく抽出してください
select 
	DISTINCT C.component_name
from 
	components C
where
	unit_cost >= (
		select 
			AVG(C2.unit_cost)
		from
			components C2
	)
;

-- 問題5：複数の製品に共通して使われている部品を抽出してください
select 
	component_id,
	component_name,
	product_id
from
	(
		select
			P.product_id,
			P.component_id,
			C.component_name,
			count(P.product_id) over(partition by P.component_id) AS COUNT_DATA
		from 
			product_components P
		inner join
			components C
		on
			P.component_id = C.component_id
	)
where
	COUNT_DATA >= 2
;
```
-- 集計対象期間は 2025年7月1日〜2025年8月31日 の2か月間。
-- 製品タイプ (PRODUCT_TYPE) × 年月（例: 2025-07, 2025-08）ごとに以下を集計：
-- 総売上額（unit_price × quantity）
-- 総作業時間（END_TIME - START_TIME の時間換算）
-- 1時間あたり売上（総売上額 ÷ 総作業時間）
-- 各月ごとに、1時間あたり売上が高い順に順位（RANK関数）を付与してください。
-- 最終結果は、
-- year_month（YYYY-MM形式）
-- product_type
-- total_sales
-- total_hours
-- sales_per_hour
-- rank_in_month
-- のカラム構成で表示してください。
-- 売上・時間ともに小数点以下2桁まで表示。
```
WITH PD_WO_TBL AS (
  SELECT
    PD.PRODUCT_TYPE,
    WO.WORK_ORDER_ID,
    WO.ORDER_DATE,
    SUM(PD.UNIT_PRICE * WO.QUANTITY) total_sales
  FROM
    PRODUCTS PD
  INNER JOIN
    WORK_ORDERS WO
  ON
    PD.PRODUCT_ID = WO.PRODUCT_ID
  WHERE
    WO.STATUS = '完了' AND
    WO.ORDER_DATE BETWEEN TO_DATE('2025-07-01', 'YYYY-MM-DD') AND TO_DATE('2025-08-31', 'YYYY-MM-DD')
  GROUP BY 
    PD.PRODUCT_TYPE,
    WO.WORK_ORDER_ID,
    WO.ORDER_DATE
),
PDWO_WP_TBL AS (
  SELECT
    PWT.PRODUCT_TYPE,
    TO_CHAR(PWT.ORDER_DATE, 'YYYY-MM-DD') year_month,
    PWT.total_sales,
    SUM((CAST(WP.END_TIME AS DATE) - CAST(WP.START_TIME AS DATE)) * 24) total_hours,
    ROUND(total_sales / NULLIF(SUM((CAST(WP.END_TIME AS DATE) - CAST(WP.START_TIME AS DATE)) * 24), 0), 2) sales_per_hour
  FROM
    PD_WO_TBL PWT
  INNER JOIN
    WORK_PROCESSES WP
  ON
    PWT.WORK_ORDER_ID = WP.WORK_ORDER_ID
  WHERE
    WP.START_TIME IS NOT NULL AND WP.END_TIME IS NOT NULL
  GROUP BY
    PWT.PRODUCT_TYPE,
    PWT.ORDER_DATE,
    PWT.total_sales
)
SELECT 
  PRODUCT_TYPE,
  year_month,
  total_sales,
  total_hours,
  sales_per_hour,
  RANK() OVER(PARTITION BY  year_month ORDER BY sales_per_hour) RANKING
FROM 
  PDWO_WP_TBL
;
```
