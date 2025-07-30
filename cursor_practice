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
