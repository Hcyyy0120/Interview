![image-20230420203148488](SQL练习.assets/image-20230420203148488.png)



### 一、

第一题-查询课程编号为01的课程比02的课程成绩高的所有学生的学号（重要）

```sql
SELECT
	a.* 
FROM
	( SELECT s_id, s_score FROM score WHERE c_id = '01' ) a
	JOIN ( SELECT s_id, s_score FROM score WHERE c_id = '02' ) b ON a.s_id = b.s_id 
WHERE
	a.s_score > b.s_score
```

### 二、

查询平均成绩大于60分的学生的学号和平均成绩（重点）

```sql
#1
SELECT
	t.s_id,
	t.avgScore 
FROM
	( SELECT s_id, avg( s_score ) AS avgScore FROM score GROUP BY s_id ) AS t 
WHERE
	t.avgScore > 60
#2
SELECT
	s_id,
	avg( s_score ) 
FROM
	score 
GROUP BY
	s_id 
HAVING
	avg( s_score ) > 60
```

### 三、

查询所有学生的学号、姓名、选课数、总成绩（不重要）

```sql
SELECT
	student.s_id,
	student.s_name,
	IFNULL(t.courseNum,0),
	IFNULL(t.scoreSum,0)
FROM
	student
	LEFT JOIN ( SELECT score.s_id, count( c_id ) AS courseNum, sum( s_score ) AS scoreSum 
               FROM score 
               GROUP BY score.s_id ) AS t 
               ON student.s_id = t.s_id
               
SELECT
	a.s_id,
	a.s_name,
	IFNULL( count( b.c_id ), 0 ),
	IFNULL( sum( b.s_score ), 0 ) 
FROM
	student AS a
	LEFT JOIN score AS b 
	ON a.s_id = b.s_id 
GROUP BY
	a.s_id
```

### 四、

查询姓张的老师的个数（不重要）

```sql
select count(t_id) from teacher where t_name like "张%"

#查询老师中有几种姓氏
select count(DISTINCT t_name) from teacher 
```

### 五、

查询没学过张三老师课的学生的学号和姓名(重要)

```sql
SELECT
	student.s_id,
	student.s_name 
FROM
	student 
WHERE
	s_id NOT IN (
	SELECT
		score.s_id 
	FROM
		score 
WHERE
	score.c_id = ( SELECT a.c_id FROM course AS a JOIN teacher AS b ON a.t_id = b.t_id WHERE b.t_name = "张三" ))
```

