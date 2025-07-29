# 基本構造

```
DECLARE
   -- 変数宣言
BEGIN
   -- 実行処理
EXCEPTION
   -- エラーハンドリング
END;
```
--------
```
SET SERVEROUTPUT ON;

BEGIN
   DBMS_OUTPUT.PUT_LINE('Hello PL/SQL!');
END;
/
```
--------
```
DECLARE
   CURSOR カーソル名 IS
      SELECT文;
   受け取る変数;
BEGIN
   OPEN カーソル名;
   LOOP
      FETCH カーソル名 INTO 変数;
      EXIT WHEN カーソル名%NOTFOUND;
      -- ここに処理を書く
   END LOOP;
   CLOSE カーソル名;
END;
```
```
DECLARE
  CURSOR v_test IS
    SELECT
      P.product_id,
      P.product_name,
      SUM(WO.quantity * P.unit_price) AS total_sales
    FROM
      products P
    INNER JOIN
      work_orders WO
    ON
      P.product_id = WO.product_id
    GROUP BY
      P.product_id, P.product_name
    ORDER BY
      total_sales DESC;

  v_product_id   products.product_id%TYPE;
  v_product_name products.product_name%TYPE;
  v_total_sales  NUMBER;
BEGIN
  OPEN v_test;

  LOOP
    FETCH v_test INTO v_product_id, v_product_name, v_total_sales;
    EXIT WHEN v_test%NOTFOUND;

    DBMS_OUTPUT.PUT_LINE(
      '製品ID: ' || v_product_id ||
      ', 名称: ' || v_product_name ||
      ', 売上: ' || TO_CHAR(v_total_sales)
    );
  END LOOP;

  CLOSE v_test;
END;
/
```
```
DECLARE
  CURSOR v_test IS
    SELECT
      P.product_id,
      P.product_name,
      SUM(WO.quantity * P.unit_price) AS total_sales
    FROM
      products P
    INNER JOIN
      work_orders WO
    ON
      P.product_id = WO.product_id
    GROUP BY
      P.product_id, P.product_name
    ORDER BY
      total_sales DESC;
BEGIN
  FOR rec IN v_test LOOP
    DBMS_OUTPUT.PUT_LINE(
      '製品ID: ' || rec.product_id ||
      ', 名称: ' || rec.product_name ||
      ', 売上: ' || TO_CHAR(rec.total_sales)
    );
  END LOOP;
END;
/
```
```
DECLARE
  CURSOR c_test(p_status VARCHAR2) IS
  select 
    P.product_id,
    P.product_name,
    WO.work_order_id,
    WO.quantity, 
    WO.status
  from 
    products P
  inner join
    work_orders WO
  on
    P.product_id = WO.product_id
  where
    WO.status = p_status
  group by
    P.product_id,
    P.product_name,
    WO.work_order_id,
    WO.quantity,
    WO.status
  order by
    P.product_id;
BEGIN
  FOR rec IN c_test('完了') LOOP
    DBMS_OUTPUT.PUT_LINE('■製品ID: ' || rec.product_id || ', 名称:' || rec.product_name);
    DBMS_OUTPUT.PUT_LINE('作業ID: ' || rec.work_order_id || ' 数量:' || rec.quantity);
  END LOOP;
END;
/
```
```
DECLARE
  CURSOR c_products IS
    SELECT product_id, product_name FROM products;

  CURSOR c_orders(p_product_id VARCHAR2) IS
    SELECT work_order_id, quantity
    FROM work_orders
    WHERE product_id = p_product_id AND status = '完了';

BEGIN
  FOR prod_rec IN c_products LOOP
    DBMS_OUTPUT.PUT_LINE('■製品ID: ' || prod_rec.product_id || ', 名称: ' || prod_rec.product_name);

    FOR order_rec IN c_orders(prod_rec.product_id) LOOP
      DBMS_OUTPUT.PUT_LINE(' 作業ID: ' || order_rec.work_order_id || ', 数量: ' || order_rec.quantity);
    END LOOP;

  END LOOP;
END;
```
--------
```
BEGIN
   -- 0で割るとエラー
   DBMS_OUTPUT.PUT_LINE(10 / 0);
EXCEPTION
   WHEN ZERO_DIVIDE THEN
      DBMS_OUTPUT.PUT_LINE('0で割ることはできません');
   WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('その他のエラーが発生');
END;
```
--------
```
DECLARE
   i NUMBER := 1;
BEGIN
   LOOP
      EXIT WHEN i > 5;
      DBMS_OUTPUT.PUT_LINE('Count: ' || i);
      i := i + 1;
   END LOOP;
END;
```
例えるなら：
1. CURSORは本のしおりのようなもの
2. FETCHは「しおりの位置にある1ページを読む」動作
3. INFO 変数(v_name)は「読んだページの内容をメモするノート」
