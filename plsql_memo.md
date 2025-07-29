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
