### 特定製品の部品構成と総コストを表示する（難しくなって来た）

```
DECLARE
  CURSOR pc_product_component IS
    SELECT product_id FROM product_components GROUP BY product_id;
  
  CURSOR pc_component(c_product_id products.product_id%TYPE) IS
    SELECT
      PC.product_id,
      CP.component_name,
      CP.cost,
      PC.QUANTITY,
      CP.cost * PC.QUANTITY "KEI"
    FROM
      product_components PC
    INNER JOIN
      components CP
    ON
      PC.component_id = CP.component_id
    where
      PC.product_id = c_product_id;

    v_total_cost NUMBER := 0;
BEGIN
  FOR rec IN pc_product_component LOOP
    DBMS_OUTPUT.PUT_LINE('製品ID: ' || rec.product_id || 'の部品構成：');
    v_total_cost := 0; 
    FOR rec2 IN pc_component(rec.product_id) LOOP
      DBMS_OUTPUT.PUT_LINE('  ・' || rec2.component_name || '（単価: ' || rec2.cost || '）× ' || rec2.quantity || ' = ' || rec2.KEI);
      v_total_cost := v_total_cost + rec2.KEI;
    END LOOP;
    DBMS_OUTPUT.PUT_LINE('部品コスト合計: ' || v_total_cost);
    DBMS_OUTPUT.PUT_LINE('-');
  END LOOP; 
END;
/
```

#### %ROWTYPE の使い方 
```
DECLARE
  v_product products%ROWTYPE;
BEGIN
  SELECT * INTO v_product FROM products WHERE product_id = 'P001';
  DBMS_OUTPUT.PUT_LINE('製品名：' || v_product.product_name);
END;

```

#### コレクション型（TABLE OF）による複数データの保持とループ 
```
DECLARE
  TYPE name_list IS TABLE OF VARCHAR2(100) INDEX BY PLS_INTEGER;
  v_names name_list;
BEGIN
  v_names(1) := '佐藤';
  v_names(2) := '鈴木';
  v_names(3) := '高橋';

  FOR i IN 1..3 LOOP
    DBMS_OUTPUT.PUT_LINE(v_names(i));
  END LOOP;
END;
```
```
DECLARE
  TYPE name_list IS TABLE OF VARCHAR2(100) INDEX BY PLS_INTEGER;
  v_names name_list;
  i PLS_INTEGER := 1;
BEGIN
  
  FOR rec IN (SELECT component_name FROM components) LOOP
    v_names(i) := rec.component_name;
    i := i + 1;
  END LOOP;

  FOR j IN 1 .. v_names.COUNT LOOP
    DBMS_OUTPUT.PUT_LINE(v_names(j));
  END LOOP;
END;
/
```

#### EXCEPTION によるエラーハンドリング
```
DECLARE
  v_price NUMBER;
BEGIN
  SELECT cost INTO v_price FROM components WHERE component_id = 'C999'; -- 存在しない
  DBMS_OUTPUT.PUT_LINE('価格: ' || v_price);
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    DBMS_OUTPUT.PUT_LINE('部品が見つかりませんでした');
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('その他のエラーが発生しました');
END;
```
```
-- 部品の新しい単価を受け取り、その製品に含まれる部品単価を更新するストアドプロシージャを作成
CREATE OR REPLACE PROCEDURE update_component_cost (
    p_component_id IN VARCHAR2,
    p_new_cost     IN NUMBER
) IS
BEGIN
    UPDATE components
    SET unit_cost = p_new_cost
    WHERE component_id = p_component_id;

    DBMS_OUTPUT.PUT_LINE(p_component_id || ' : ' || p_new_cost || '更新しました。');
END;
/

-- 実行＆作成済確認コマンド
EXEC update_component_cost('C001', 1500);

SELECT object_name, status 
FROM user_objects 
WHERE object_type = 'PROCEDURE';
```
