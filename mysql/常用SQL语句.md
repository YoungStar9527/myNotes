# 1 删除重复数据

```sql
-- assess_event_cooperate_label_id为主键或者唯一值
-- dept_id,assess_event_id,type_one_new,type_two_new 这些为判断重复的条件，4个完全一样就是重复
-- 套多个子查询是MYSQL不允许先查询再删除，需要多套几个避免这种异常
delete from assess_event_cooperate_label
where assess_event_cooperate_label_id in
(
select assess_event_cooperate_label_id from 
(
select A.assess_event_cooperate_label_id
FROM
	assess_event_cooperate_label A,
	(
		SELECT
			dept_id,
			assess_event_id,
			type_one_new,
			type_two_new
		FROM
			assess_event_cooperate_label
		GROUP BY
			dept_id,
			assess_event_id,
			type_one_new,
			type_two_new
		HAVING
			COUNT(*) > 1
	) B
WHERE
	A.dept_id = B.dept_id
AND A.assess_event_id = B.assess_event_id
AND A.type_one_new = B.type_one_new
AND A.type_two_new = B.type_two_new
AND A.assess_event_cooperate_label_id NOT IN (
	SELECT
		MIN(
			assess_event_cooperate_label_id
		) AS ID
	FROM
		assess_event_cooperate_label
	GROUP BY
		dept_id,
		assess_event_id,
		type_one_new,
		type_two_new
	HAVING
		COUNT(*) > 1
)
) AAA
)
```

引用：

https://blog.csdn.net/anya/article/details/6407280

https://www.jianshu.com/p/ba80e1a1b7dc

# 2 修改字符串中对应内容

```sql
UPDATE assess_event_cooperate_label_copy_copy SET dept_id_str=REPLACE(dept_id_str, '484', '514');
-- 将dept_id_str字段中的484改为514
```

引用：https://www.cnblogs.com/duanxz/p/3936732.html