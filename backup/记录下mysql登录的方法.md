Docker的mysql

1. docker exec -it 9b /bin/bash
2. mysql -u root -p  密码123456
3. USE CONTRACT;  选择数据库
4. SHOW TABLES;  选择表项

对表进行去重的方法：
-- 1. 创建临时表以存储每个 address 的最小 id
CREATE TEMPORARY TABLE temp_table AS
SELECT MIN(id) AS id
FROM contract_address
GROUP BY address;

-- 2. 删除重复记录，保留在临时表中的记录
DELETE FROM contract_address
WHERE id NOT IN (SELECT id FROM temp_table);

-- 3. 删除临时表
DROP TEMPORARY TABLE temp_table;
