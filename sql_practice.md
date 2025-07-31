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
