#### 基本形
```
CREATE OR REPLACE PROCEDURE プロシージャ名 (
    引数名1 データ型,
    引数名2 データ型
) IS
  -- 変数宣言部
BEGIN
  -- 実行する処理
  DBMS_OUTPUT.PUT_LINE('プロシージャが実行されました');
END;
/
```

#### 
```
CREATE OR REPLACE PROCEDURE show_product_name (
    p_product_id IN products.product_id%TYPE
) IS
    v_name products.product_name%TYPE;
BEGIN
    SELECT product_name
    INTO v_name
    FROM products
    WHERE product_id = p_product_id;

    DBMS_OUTPUT.PUT_LINE('製品名: ' || v_name);
END;
/

EXEC show_product_name(101);
```

```
CREATE OR REPLACE PROCEDURE get_order_count (
    p_status IN work_orders.status%TYPE,
    p_count  OUT NUMBER
) IS
BEGIN
    SELECT COUNT(*)
    INTO p_count
    FROM work_orders
    WHERE status = p_status;
END;
/

DECLARE
  v_cnt NUMBER;
BEGIN
  get_order_count('完了', v_cnt);
  DBMS_OUTPUT.PUT_LINE('完了件数: ' || v_cnt);
END;
/
```
