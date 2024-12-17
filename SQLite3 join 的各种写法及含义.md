##  SQLite3 join 的各种写法及含义

SQLite3 中的 JOIN 操作用于将两个或多个表中的数据结合起来。以下是 JOIN 的各种写法及其含义：

1. INNER JOIN（内部连接）
```sql
SELECT columns
FROM table1
INNER JOIN table2
ON table1.column = table2.column;
```
- **含义**：返回两个表中匹配的记录。如果 table1 中的某行在 table2 中没有匹配的记录，该行将不被返回。

2. LEFT JOIN (或 LEFT OUTER JOIN)
```sql
SELECT columns
FROM table1
LEFT JOIN table2
ON table1.column = table2.column;
```
- 含义：返回 table1 中所有记录，以及 table2 中匹配的记录。如果 table2 中没有匹配的记录，对应的字段将显示为 NULL。

3. RIGHT JOIN (或 RIGHT OUTER JOIN)
```sql
SELECT columns
FROM table1
RIGHT JOIN table2
ON table1.column = table2.column;
```
- 含义：与 LEFT JOIN 相反，返回 table2 中所有记录，以及 table1 中匹配的记录。如果 table1 中没有匹配的记录，对应的字段将显示为 NULL。

4. FULL JOIN (或 FULL OUTER JOIN)
```sql
SELECT columns
FROM table1
FULL JOIN table2
ON table1.column = table2.column;
```
- 含义：返回两个表中的所有记录。如果某个表中的行在另一个表中没有匹配的记录，该行的所有字段将显示为 NULL。注意，SQLite 不直接支持 FULL JOIN，但可以用 LEFT JOIN 和 RIGHT JOIN 的联合来模拟：
```sql
SELECT columns FROM table1 
LEFT JOIN table2 ON table1.column = table2.column
UNION
SELECT columns FROM table1 
RIGHT JOIN table2 ON table1.column = table2.column
WHERE table1.column IS NULL;
```

5. CROSS JOIN ( 交叉加入 )
```sql
SELECT columns
FROM table1
CROSS JOIN table2;
```
- 含义：返回两个表的笛卡尔积，即每行与另一表的每一行进行匹配。

6. NATURAL JOIN ( 自然联接 )
```sql
SELECT columns
FROM table1
NATURAL JOIN table2;
```
- 含义：基于两个表中相同列名的列进行 INNER JOIN。不需要指定 ON 条件，因为系统会自动匹配同名列。

注意事项
- 使用 JOIN 时，确保 ON 子句中指定的列是正确的，以避免逻辑错误。
- 选择合适的 JOIN 类型取决于所需结果的完整性和性能考虑。

这些是 SQLite3 中 JOIN 的主要类型及其用法，希望这对你理解和使用 JOIN 有帮助。
