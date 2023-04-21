![image-20230420203148488](SQL练习.assets/image-20230420203148488.png)



### 一、

#### 查询课程编号为01的课程比02的课程成绩高的所有学生的学号（重要）

```sql

SELECT
	a.* 
FROM
	( SELECT s_id, s_score FROM score WHERE c_id = '01' ) a
	JOIN ( SELECT s_id, s_score FROM score WHERE c_id = '02' ) b ON a.s_id = b.s_id 
WHERE
	a.s_score > b.s_score
	
#自连接
SELECT
	s1.学号 
FROM
	score s1
	JOIN score s2 ON s1.学号 = s2.学号 
WHERE
	s1.课程号 = '01' 
	AND s2.课程号 = '02' 
	AND s1.成绩 > s2.成绩

```

### 二、

#### 查询平均成绩大于60分的学生的学号和平均成绩（重点）

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

#### 查询所有学生的学号、姓名、选课数、总成绩（不重要）

```sql
#1
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
#2               
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

#### 查询姓张的老师的个数（不重要）

```sql
select count(t_id) from teacher where t_name like "张%"

#查询老师中有几种姓氏
select count(DISTINCT t_name) from teacher 
```

### 五、

#### 查询没学过张三老师课的学生的学号和姓名(重要)

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

### 六、

#### 查询学过"张三"老师所教的所有课的同学的学号、姓名（重要）

```sql
SELECT
	student.s_id,
	student.s_name 
FROM
	score
	JOIN student ON score.s_id = student.s_id 
WHERE
	score.c_id IN 
	(
		SELECT
			a.c_id 
		FROM
			course a
			JOIN teacher b ON a.t_id = b.t_id 
		WHERE
			b.t_name = "张三"
    )
```

### 七、

#### 查询学过编号为"01”的课程并且也学过编号为"02"的课程的学生的学号、姓名（重要）

```sql
SELECT
	st.s_id,
	st.s_name 
FROM
	student st
	JOIN score s 
	ON st.s_id = s.s_id 
WHERE
	s.c_id = "02" 
	AND s.s_id IN 
	(
		SELECT
			s_id 
		FROM
			score 
		WHERE
			c_id = "01"
    )
#自连接 
SELECT
	s1.学号,姓名 
FROM
	score s1
	JOIN score s2 ON s1.学号 = s2.学号
	JOIN student s ON s.学号 = s2.学号 
WHERE
	s1.课程号 = '01' AND s2.课程号 = '02';
```

### 八、

#### 查询课程编号为“02”的总成绩

```sql
select c_id,sum(s_score) from score GROUP BY c_id having c_id = "02"
```

### 九、

#### 查询  所有课程  的成绩都小于60分的学生的学号、姓名

```sql
#1
SELECT
	t1.s_id,
	st.s_name 
FROM
	( SELECT s_id, count( c_id ) cnt1 FROM score WHERE s_score < 60 GROUP BY s_id ) t1
	JOIN ( SELECT s_id, count( c_id ) cnt2 FROM score GROUP BY s_id ) t2 ON t1.s_id = t2.s_id
	JOIN student st ON st.s_id = t1.s_id 
WHERE
	t1.cnt1 = t2.cnt2

#2
SELECT
	st.s_id,
	st.s_name 
FROM
	student st
	JOIN score s ON st.s_id = s.s_id 
GROUP BY
	s_id 
HAVING
	max( s_score ) < 60
```

### 十、

#### 查询没有学全所有课的学生的学号、姓名(重点)

```sql
#注意左连接，因为有 没有选课的学生
SELECT
	st.s_id,
	st.s_name 
FROM
	student as st
	LEFT JOIN score as sc ON st.s_id = sc.s_id 
GROUP BY
	st.s_id 
HAVING
	count( DISTINCT sc.c_id ) < ( SELECT count( c_id ) FROM course )
```

